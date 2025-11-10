---
title: "Deploying LLMs on Amazon EKS Using vLLM Deep Learning Containers"
date: "2025-08-14T10:00:00+07:00"
weight: 3
chapter: false
pre: " <b> 3. </b> "
---

> Authors: Vishal Naik and Sumeet Tripathi | Date: August 14, 2025 | In: [AWS Deep Learning AMIs](https://aws.amazon.com/deep-learning-amis/), [Best Practices](https://aws.amazon.com/blogs/architecture/category/best-practices/), [Expert (400)](https://aws.amazon.com/blogs/architecture/category/learning-levels/expert-400/), [Generative AI](https://aws.amazon.com/generative-ai/), [Technical How-to](https://aws.amazon.com/blogs/architecture/category/post-types/technical-how-to/) | [Permalink](https://aws.amazon.com/blogs/architecture/deploy-llms-on-amazon-eks-using-vllm-deep-learning-containers/) | [Comments](https://aws.amazon.com/blogs/architecture/deploy-llms-on-amazon-eks-using-vllm-deep-learning-containers/#comments)

Organizations face significant challenges when deploying large language models (LLMs) efficiently at scale. Major hurdles include optimizing GPU resource utilization, managing network infrastructure, and providing effective access to model weights. When running distributed inference workloads, organizations often encounter the complexity of orchestrating model activities across multiple nodes. Common challenges include effectively distributing model components across available GPUs, coordinating seamless communication between processing units, and maintaining consistent performance with low latency and high throughput.

[vLLM](https://github.com/vllm-project/vllm) is an open-source library for fast inference and serving of LLMs. The [vLLM AWS Deep Learning Containers (DLCs)](https://aws.amazon.com/releasenotes/aws-deep-learning-containers-for-vllm-inference/) are optimized for deploying vLLMs on [Amazon Elastic Compute Cloud](https://aws.amazon.com/ec2/)(Amazon EC2), [Amazon Elastic Container Service](https://aws.amazon.com/ecs/) (Amazon ECS), and [Amazon Elastic Kubernetes Service](https://aws.amazon.com/eks/) (Amazon EKS), and are provided free of charge. These containers package a pre-configured, tested environment that works seamlessly upon use, including necessary dependencies such as drivers and libraries for efficient running of vLLMs, and offer integrated support for [Elastic Fabric Adapter](https://aws.amazon.com/hpc/efa/) (EFA) for high-performance multi-node inference workloads. You no longer need to build inference environments from scratch. Instead, you can install vLLM DLC, which will automatically set up and configure the environment, allowing you to deploy inference workloads at scale.

In this post, we demonstrate deploying the [DeepSeek-R1-Distill-Qwen-32B](https://huggingface.co/deepseek-ai/DeepSeek-R1-Distill-Qwen-32B) model using AWS DLCs for vLLMs on Amazon EKS, illustrating how these purpose-built containers simplify deploying this powerful open-source inference engine. This solution can help address complex infrastructure challenges of deploying LLMs while maintaining performance and cost-efficiency.

## AWS DLCs

AWS DLCs provide AI/ML professionals with Docker environments optimized for training and deploying generative AI models within their pipelines and workflows on Amazon EC2, Amazon EKS, and Amazon ECS. AWS DLCs target self-managed machine learning customers who prefer to build and maintain their own AI/ML environments, want fine control over instance infrastructure, and manage their own training and inference workloads. DLCs are available as Docker images for training and inference, compatible with PyTorch and TensorFlow. They are updated with the latest versions of frameworks and drivers, verified for compatibility and security, and offered free of charge. They can also be easily customized following our recipes. Using AWS DLCs as building blocks for generative AI environments reduces operational overhead, lowers TCO, accelerates AI/ML product development, and allows teams to focus on creating value by extracting AI-powered insights from organizational data.

## Solution Overview

The diagram below illustrates the interaction between Amazon EKS, GPU-enabled EC2 instances with EFA networking, and [Amazon FSx for Lustre](https://aws.amazon.com/fsx/lustre/). Client requests flow through an Application Load Balancer (ALB) to vLLM server pods running on EKS nodes, where model weights are stored on FSx for Lustre. This architecture provides a scalable, high-performance solution for serving LLM inference workloads with optimal cost-efficiency.

> <img src="/images/bl3-1.png" alt="" width="55%">

The following diagram shows the DLC stack on AWS. It depicts a comprehensive architecture from EC2 instance layer, through container runtime, essential GPU drivers, and ML frameworks like PyTorch. The layered diagram illustrates how CUDA, NCCL, and other key components integrate to support high-performance deep learning workloads.

> <img src="/images/bl3-2.png" alt="" width="55%">

The vLLM DLCs are specially optimized for high-performance inference, with integrated support for tensor parallelism and pipeline parallelism across multiple GPUs and nodes. This optimization allows efficient scaling of large models like DeepSeek-R1-Distill-Qwen-32B, which would otherwise be very difficult to deploy and manage. The containers include CUDA and EFA driver configurations optimized for maximum throughput for distributed inference workloads. This solution leverages the following AWS services and components:

* **AWS DLCs for vLLMs** – Pre-configured, optimized Docker images that simplify deployment and maximize performance

* **EKS cluster** – Provides Kubernetes control plane to orchestrate containers

* **P4d.24xlarge instances** – [EC2 P4d instances](https://aws.amazon.com/ec2/instance-types/p4/) with 8 NVIDIA A100 GPUs each, configured in a managed node group

* **Elastic Fabric Adapter** – Network interface enabling high-throughput, low-latency communication for HPC applications

* **FSx for Lustre** – High-performance file system for storing model weights

* **LeaderWorkerSet pattern** – Custom Kubernetes resource to deploy vLLM in distributed configurations

* **AWS Load Balancer Controller** – Manages ALB for external access

By combining these components, we create an inference system that offers low-latency, high-throughput LLM serving with minimal operational overhead.

## Prerequisites

Before starting, ensure you have:

* An AWS account with access to EC2 P4 instances (may require [request a quota increase](https://aws.amazon.com/support/create-case))

* A terminal with the following tools installed:
    * [AWS CLI](https://aws.amazon.com/cli/) v2.11.0 or higher
    * [eksctl](https://eksctl.io/) v0.150.0 or higher
    * [kubectl](https://kubernetes.io/docs/tasks/tools/) v1.27 or higher
    * [Helm](https://helm.sh/) v3.12.0 or higher

* An AWS CLI profile (vllm-profile) configured with IAM roles or users with permissions to:
    * Create, manage, delete EKS clusters and node groups
    * Create, manage, delete EC2 resources, VPCs, subnets, security groups, internet gateways
    * Create and manage IAM roles
    * Create, update, delete CloudFormation stacks
    * Create and manage FSx for Lustre file systems
    * Create and manage Elastic Load Balancers

This solution can be deployed in regions where Amazon EKS, P4d instances, and FSx for Lustre are available. This guide uses the us-west-2 region. The complete deployment takes approximately 60–90 minutes.

Clone our GitHub repository containing configuration files:

```bash
# Clone the repository
git clone [https://github.com/aws-samples/sample-aws-deep-learning-containers.git](https://github.com/aws-samples/sample-aws-deep-learning-containers.git)
cd vllm-samples/deepseek/eks
```
## Creating an EKS Cluster
First, create an EKS cluster in the us-west-2 region using the provided configuration file. This sets up the Kubernetes control plane to orchestrate our containers. The cluster is configured with a VPC, subnets, and security groups optimized for GPU workloads.
```bash
# Update the region in eks-cluster.yaml if needed
sed -i "s|region: us-east-1|region: us-west-2|g" eks-cluster.yaml
# Create the EKS cluster
eksctl create cluster -f eks-cluster.yaml --profile vllm-profile
```

This takes about 15–20 minutes. During this time, eksctl creates a CloudFormation stack with the resources for your EKS cluster, as shown in the screenshot below.
> <img src="/images/bl3-3.png" alt="" width="55%">

Verify cluster creation:
```bash
# Verify cluster creation
eksctl get cluster --profile vllm-profile
```
Expected output:
```bash
NAME            REGION          EKSCTL CREATED
vllm-cluster    us-west-2       True
```
You can also view the cluster in the Amazon EKS console.

> <img src="/images/bl3-4.png" alt="" width="55%">

## Creating an EFA-Enabled Node Group

Next, create a managed node group with P4d.24xlarge instances with EFA enabled. These instances, equipped with 8 NVIDIA A100 GPUs each, provide considerable computational power for LLM inference. When deploying EFA-enabled instances like p4d.24xlarge for high-performance ML workloads, they should be placed in private subnets for secure network setup. By automatically discovering and using the availability zone of a private subnet in your node group configuration, you maintain proper network isolation while supporting high-throughput, low-latency communication essential for distributed training and inference with LLMs. We determine the AZ with the following code:

```bash
# Get the VPC ID from the EKS cluster
VPC_ID=$(aws --profile vllm-profile eks describe-cluster --name vllm-cluster \
  --query "cluster.resourcesVpcConfig.vpcId" --output text)

# Find the one of private subnet's availability zone
PRIVATE_AZ=$(aws --profile vllm-profile ec2 describe-subnets \
  --filters "Name=vpc-id,Values=$VPC_ID" "Name=map-public-ip-on-launch,Values=false" \
  --query "Subnets[0].AvailabilityZone" --output text)
echo "Selected private subnet AZ: $PRIVATE_AZ"

# update the nodegroup_az section with the private AZ value
sed -i "s|availabilityZones: \[nodegroup_az\]|availabilityZones: \[\"$PRIVATE_AZ\"\]|g" large-model-nodegroup.yaml

# Verify the change
grep "availabilityZones" large-model-nodegroup.yaml

# Create the node group with EFA support
eksctl create nodegroup -f large-model-nodegroup.yaml --profile vllm-profile
```
This takes about 10–15 minutes. The EFA configuration is critical for multi-node deployments, enabling high-throughput, low-latency networking among nodes. Once the node group is ready, configure kubectl:

```bash
# Configure kubectl to connect to the cluster
aws eks update-kubeconfig --name vllm-cluster --region us-west-2 --profile vllm-profile
```
Verify nodes are ready:

```bash
# Check node status
kubectl get nodes
```
Expected output:
```bash
NAME                                          STATUS   ROLES    AGE     VERSION
ip-192-168-xx-xx.us-west-2.compute.internal     Ready    <none>   5m      v1.31.7-eks-xxxx
ip-192-168-yy-yy.us-west-2.compute.internal     Ready    <none>   5m      v1.31.7-eks-xxxx
```
You can also see the node groups that are created on Amazon EKS console.
> <img src="/images/bl3-7.png" alt="" width="65%">
## Check NVIDIA device pods
Since we are using the Amazon EKS optimized AMI with GPU support (ami-0ad09867389dc17a1), the NVIDIA device plugin is already included in the cluster, so there is no need to install it separately. Verify that the NVIDIA device plugin is running:
```Bash
# Check NVIDIA device plugin pods
kubectl get pods -n kube-system | grep nvidia
```
Expected output:
```Plaintext
nvidia-device-plugin-daemonset-xxxxx 1/1 Running 0 3m48s
nvidia-device-plugin-daemonset-yyyyy 1/1 Running 0 3m48s
```
Verify GPU availability:
```Bash
# Check available GPUs
kubectl get nodes -o json | jq '.items[].status.capacity."[nvidia.com/gpu](https://nvidia.com/gpu)"'
```
Expected output:
```Plaintext
"8"
"8"
```
## Creating an FSx for Lustre File System

For optimal performance, create an FSx for Lustre file system to store model weights. FSx for Lustre provides high throughput, low latency access, essential for efficiently loading large model weights. Use the following code:
```Bash
# Create a security group for FSx Lustre
FSX_SG_ID=$(aws --profile vllm-profile ec2 create-security-group --group-name fsx-lustre-sg \
  --description "Security group for FSx Lustre" \
  --vpc-id $(aws --profile vllm-profile eks describe-cluster --name vllm-cluster \
  --query "cluster.resourcesVpcConfig.vpcId" --output text) \
  --query "GroupId" --output text)

echo "Created security group: $FSX_SG_ID"

# Add inbound rules for FSx Lustre
aws --profile vllm-profile ec2 authorize-security-group-ingress --group-id $FSX_SG_ID \
  --protocol tcp --port 988-1023 \
  --source-group $(aws --profile vllm-profile eks describe-cluster --name vllm-cluster \
  --query "cluster.resourcesVpcConfig.clusterSecurityGroupId" --output text)

aws --profile vllm-profile ec2 authorize-security-group-ingress --group-id $FSX_SG_ID \
      --protocol tcp --port 988-1023 \
      --source-group $FSX_SG_ID

# Create the FSx Lustre filesystem
SUBNET_ID=$(aws --profile vllm-profile eks describe-cluster --name vllm-cluster \
  --query "cluster.resourcesVpcConfig.subnetIds[0]" --output text)

echo "Using subnet: $SUBNET_ID"

FSX_ID=$(aws --profile vllm-profile fsx create-file-system --file-system-type LUSTRE \
  --storage-capacity 1200 --subnet-ids $SUBNET_ID \
  --security-group-ids $FSX_SG_ID --lustre-configuration DeploymentType=SCRATCH_2 \
  --tags Key=Name,Value=vllm-model-storage \
  --query "FileSystem.FileSystemId" --output text)

echo "Created FSx filesystem: $FSX_ID"

# Wait for the filesystem to be available (typically takes 5-10 minutes)
echo "Waiting for filesystem to become available..."
aws --profile vllm-profile fsx describe-file-systems --file-system-id $FSX_ID \
  --query "FileSystems[0].Lifecycle" --output text
Plaintext

# You can run the above command periodically until it returns "AVAILABLE"
# Example: watch -n 30 "aws --profile vllm-profile fsx describe-file-systems --file-system-id $FSX_ID --query FileSystems[0].Lifecycle --output text"
Bash

# Get the DNS name and mount name
FSX_DNS=$(aws --profile vllm-profile fsx describe-file-systems --file-system-id $FSX_ID \
  --query "FileSystems[0].DNSName" --output text)

FSX_MOUNT=$(aws --profile vllm-profile fsx describe-file-systems --file-system-id $FSX_ID \
  --query "FileSystems[0].LustreConfiguration.MountName" --output text)

echo "FSx DNS: $FSX_DNS"
echo "FSx Mount Name: $FSX_MOUNT"
```
The file system is configured with 1.2 TB of storage, using the SCRATCH_2 deployment type for high performance, and the security groups allow access from our EKS nodes. You can also check the FSx for Lustre file system in the FSx for Lustre console.
> <img src="/images/bl3-5.png" alt="" width="65%">

## Installing AWS FSx CSI Driver
To mount the FSx for Lustre file system in our Kubernetes pods, we install the AWS FSx CSI Driver. This driver allows Kubernetes to automatically provision and mount FSx for Lustre volumes.
```Bash
# Add the AWS FSx CSI Driver Helm repository
helm repo add aws-fsx-csi-driver [https://kubernetes-sigs.github.io/aws-fsx-csi-driver/](https://kubernetes-sigs.github.io/aws-fsx-csi-driver/)
helm repo update

# Install the AWS FSx CSI Driver
helm install aws-fsx-csi-driver aws-fsx-csi-driver/aws-fsx-csi-driver --namespace kube-system
```
Verify AWS FSx CSI Driver is running:

```Bash
# Check AWS FSx CSI Driver pods
kubectl get pods -n kube-system | grep fsx
```
Expected Output:

```Plaintext
fsx-csi-controller-xxxx     4/4     Running   0          24s
fsx-csi-controller-yyyy     4/4     Running   0          24s
fsx-csi-node-xxxx           3/3     Running   0          24s
fsx-csi-node-yyyy           3/3     Running   0          24s
```
Create Kubernetes resources for FSx:
We create the necessary Kubernetes resources to use our FSx for Lustre file system:
```Bash
# Update the storage class with your subnet and security group IDs
sed -i "s|<subnet-id>|$SUBNET_ID|g" fsx-storage-class.yaml
sed -i "s|<sg-id>|$FSX_SG_ID|g" fsx-storage-class.yaml

# Update the PV with your FSx Lustre details
sed -i "s|<fs-id>|$FSX_ID|g" fsx-lustre-pv.yaml
sed -i "s|<fs-id>.fsx.us-west-2.amazonaws.com|$FSX_DNS|g" fsx-lustre-pv.yaml
sed -i "s|<mount-name>|$FSX_MOUNT|g" fsx-lustre-pv.yaml

# Apply the Kubernetes resources
kubectl apply -f fsx-storage-class.yaml
kubectl apply -f fsx-lustre-pv.yaml
kubectl apply -f fsx-lustre-pvc.yaml
```
Verify the resources have been created successfully:

```Bash
# Check storage class
kubectl get sc fsx-sc

# Check persistent volume
kubectl get pv fsx-lustre-pv

# Check persistent volume claim
kubectl get pvc fsx-lustre-pvc
```
Expected Output:

```Plaintext
NAME     PROVISIONER       RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
fsx-sc   fsx.csi.aws.com   Retain          Immediate           false                  1m

NAME            CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                    STORAGECLASS   REASON   AGE
fsx-lustre-pv   1200Gi     RWX            Retain           Bound    default/fsx-lustre-pvc   fsx-sc                  1m

NAME             STATUS   VOLUME          CAPACITY   ACCESS MODES   STORAGECLASS   AGE
fsx-lustre-pvc   Bound    fsx-lustre-pv   1200Gi     RWX            fsx-sc         1m
```
The following resources include:

* A StorageClass that defines how to provision FSx for Lustre volumes

* A PersistentVolume that represents our existing FSx for Lustre file system

* A PersistentVolumeClaim that our pods will use to mount the file system
## Install AWS Load Balancer Controller
To expose our vLLM service to the outside world, we install the AWS Load Balancer Controller. This controller manages ALBs for our Kubernetes services and ingresses.
Refer to Install AWS Load Balancer Controller with Helm for more details.
```Bash
# Download the IAM policy document
curl -o iam-policy.json [https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/main/docs/install/iam_policy.json](https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/main/docs/install/iam_policy.json)

# Create the IAM policy
aws --profile vllm-profile iam create-policy --policy-name AWSLoadBalancerControllerIAMPolicy --policy-document file://iam-policy.json

# Create an IAM OIDC provider for the cluster
eksctl utils associate-iam-oidc-provider --profile vllm-profile --region=us-west-2 --cluster=vllm-cluster --approve

# Create an IAM service account for the AWS Load Balancer Controller
ACCOUNT_ID=$(aws --profile vllm-profile sts get-caller-identity --query "Account" --output text)
eksctl create iamserviceaccount \
  --profile vllm-profile \
  --cluster=vllm-cluster \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --attach-policy-arn=arn:aws:iam::$ACCOUNT_ID:policy/AWSLoadBalancerControllerIAMPolicy \
  --override-existing-serviceaccounts \
  --approve

# Install the AWS Load Balancer Controller using Helm
helm repo add eks [https://aws.github.io/eks-charts](https://aws.github.io/eks-charts)
helm repo update

# Install the CRDs
kubectl apply -f [https://raw.githubusercontent.com/aws/eks-charts/master/stable/aws-load-balancer-controller/crds/crds.yaml](https://raw.githubusercontent.com/aws/eks-charts/master/stable/aws-load-balancer-controller/crds/crds.yaml)

# Install the controller
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=vllm-cluster \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller
```
Verify that the AWS Load Balancer Controller is running:
```Bash
# Check AWS Load Balancer Controller pods
kubectl get pods -n kube-system | grep aws-load-balancer-controller
# Install the LeaderWorkerSet controller
   helm install lws oci://registry.k8s.io/lws/charts/lws \
     --version=0.6.1 \
     --namespace lws-system \
     --create-namespace \
     --wait --timeout 300s
```
## Configure Security Groups for ALB
We create a dedicated security group for the ALB and configure it to allow inbound traffic on port 80 from our client IP addresses.
We also configure the node security group to allow traffic from the ALB security group to the vLLM service port.
```Bash
# Create security group for the ALB
USER_IP=$(curl -s [https://checkip.amazonaws.com](https://checkip.amazonaws.com))

VPC_ID=$(aws --profile vllm-profile eks describe-cluster --name vllm-cluster \
  --query "cluster.resourcesVpcConfig.vpcId" --output text)

ALB_SG=$(aws --profile vllm-profile ec2 create-security-group \
  --group-name vllm-alb-sg \
  --description "Security group for vLLM ALB" \
  --vpc-id $VPC_ID \
  --query "GroupId" --output text)

echo "ALB security group: $ALB_SG"

# Allow inbound traffic on port 80 from your IP
aws --profile vllm-profile ec2 authorize-security-group-ingress \
  --group-id $ALB_SG \
  --protocol tcp \
  --port 80 \
  --cidr ${USER_IP}/32

# Get the node group security group ID
NODE_INSTANCE_ID=$(aws --profile vllm-profile ec2 describe-instances \
  --filters "Name=tag:eks:nodegroup-name,Values=vllm-p4d-nodes-efa" \
  --query "Reservations[0].Instances[0].InstanceId" --output text)

NODE_SG=$(aws --profile vllm-profile ec2 describe-instances \
  --instance-ids $NODE_INSTANCE_ID \
  --query "Reservations[0].Instances[0].SecurityGroups[0].GroupId" --output text)

echo "Node security group: $NODE_SG"

# Allow traffic from ALB security group to node security group on port 8000 (vLLM service port)
aws --profile vllm-profile ec2 authorize-security-group-ingress \
  --group-id $NODE_SG \
  --protocol tcp \
  --port 8000 \
  --source-group $ALB_SG

# Update the security group in the ingress file
sed -i "s|<sg-id>|$ALB_SG|g" vllm-deepseek-32b-lws-ingress.yaml
```
Verify that the security groups were created and configured correctly:
```Bash
# Verify ALB security group
aws --profile vllm-profile ec2 describe-security-groups --group-ids $ALB_SG --query "SecurityGroups[0].IpPermissions"
```
Expected output for the ALB security group:
```Bash
JSON
[
    {
        "FromPort": 80,
        "IpProtocol": "tcp",
        "IpRanges": [
            {
                "CidrIp": "USER_IP/32"
            }
        ],
        "ToPort": 80
    }
]

# Verify node security group rules
aws --profile vllm-profile ec2 describe-security-groups --group-ids $NODE_SG --query "SecurityGroups[0].IpPermissions"
```
## Deploying the vLLM Server
Finally, we deploy the vLLM server using the LeaderWorkerSet pattern. The AWS Deep Learning Containers (DLCs) provide an optimized, streamlined environment that minimizes the complexity typically associated with LLM deployments. The vLLM DLCs come preconfigured with the following features:
* Optimized CUDA libraries for maximum GPU utilization

* EFA drivers and configurations for high-speed node-to-node communication

* Preconfigured Ray framework for distributed computing

* Performance-tuned vLLM installation with tensor and pipeline parallelism support

This prepackaged solution significantly reduces deployment time and the need for complex environment setup, dependency management, and performance tuning that would otherwise require specialized expertise.

```Bash
# Deploy the vLLM server
# First, verify that the AWS Load Balancer Controller is running
kubectl get pods -n kube-system | grep aws-load-balancer-controller

# Wait until the controller is in Running state
# If it's not running, check the logs:
# kubectl logs -n kube-system deployment/aws-load-balancer-controller

# Apply the LeaderWorkerSet
kubectl apply -f vllm-deepseek-32b-lws.yaml
```
The deployment will start immediately, but the pod may remain in the ContainerCreating state for several minutes (5–15 minutes) while it pulls the large GPU-enabled container image. Once the container starts, it may take an additional 10–15 minutes to download and load the DeepSeek model. You can monitor the progress with the following commands:
```Bash
# Monitor pod status
kubectl get pods

# Check pod logs
kubectl logs -f <pod-name>
Here is the out put of one of the pods
Kubectl logs -f vllm-deepseek-32b-lws-0
```
Below is an example of the output from one of the running pods:
```Plaintext
NAME                    READY   STATUS    RESTARTS   AGE
vllm-deepseek-32b-lws-0   1/1     Running   0          10m
vllm-deepseek-32b-lws-0-1 1/1     Running   0          10m
```
We also deploy an ingress resource that configures the ALB to route traffic to our vLLM service:
```Bash
# Apply the ingress (only after the controller is running)
kubectl apply -f vllm-deepseek-32b-lws-ingress.yaml
```
You can check the ingress status using the command below:
```Bash
# Check ingress status
kubectl get ingress
```
Here is an example of the expected output:
```Plaintext
NAME                          CLASS   HOSTS   ADDRESS                                                                      PORTS   AGE
vllm-deepseek-32b-lws-ingress   alb     * k8s-default-vllmdeep-xxxxxxxx-xxxxxxxxxx.us-west-2.elb.amazonaws.com     80      5m
```
## Testing the Deployment

Once the deployment is complete, we can test our vLLM server. It provides the following API endpoints:

* /v1/completions – For text completions

* /v1/chat/completions – For chat completions

* /v1/embeddings – For generating embeddings

* /v1/models – For listing available models
```Bash
# Test the vLLM server
# Get the ALB endpoint
export VLLM_ENDPOINT=$(kubectl get ingress vllm-deepseek-32b-lws-ingress -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
echo "vLLM endpoint: $VLLM_ENDPOINT"

# Test the completions API
curl -X POST http://$VLLM_ENDPOINT/v1/completions \
  -H "Content-Type: application/json" \
  -d '{
      "model": "deepseek-ai/DeepSeek-R1-Distill-Qwen-32B",
      "prompt": "Hello, how are you?",
      "max_tokens": 100,
      "temperature": 0.7
  }'
  ```
 > <img src="/images/bl3-6.png" alt="" width="80%">
Here is an example of the expected output:
```Bash
JSON
{
  "id": "cmpl-xxxxxxxxxxxxxxxxxxxxxxxx",
  "object": "text_completion",
  "created": 1717000000,
  "model": "deepseek-ai/DeepSeek-R1-Distill-Qwen-32B",
  "choices": [
    {
      "index": 0,
      "text": " I'm doing well, thank you for asking! How about you? Is there anything I can help you with today?",
      "logprobs": null,
      "finish_reason": "length",
      "stop_reason": null,
      "prompt_logprobs": null
    }
  ],
  "usage": {
    "prompt_tokens": 5,
    "total_tokens": 105,
    "completion_tokens": 100
  }
}
```
You can also test the chat completions API:
```Bash
# Test the chat completions API
curl -X POST http://$VLLM_ENDPOINT/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
      "model": "deepseek-ai/DeepSeek-R1-Distill-Qwen-32B",
      "messages": [{"role": "user", "content": "What are the benefits of using FSx Lustre with EKS?"}],
      "max_tokens": 100,
      "temperature": 0.7
  }'
  ```
If you encounter errors, check the logs of your vLLM pods:
```Bash
# Troubleshooting
kubectl logs -f <pod-name>
```
## Performance Considerations

In this section, we discuss several performance considerations.

  ### Elastic Fabric Adapter (EFA)

  EFA provides significant performance benefits for distributed inference workloads:

* Reduced latency – Lower and more consistent latency for inter-GPU communication across nodes

* Higher throughput – Increased data transfer throughput between nodes

* Improved scaling – Better scaling efficiency across multiple nodes

* Better performance – Significantly improved performance for distributed inference workloads

### FSx for Lustre Integration

Using FSx for Lustre as model storage offers several advantages:

* Persistent storage – Model weights are stored on the FSx for Lustre file system and persist across pod restarts

* Faster loading – After the initial download, model loading is much faster

* Shared storage – Multiple pods can access the same model weights

* High performance – FSx for Lustre provides high-throughput, low-latency access to model weights

### Application Load Balancer

Using the AWS Load Balancer Controller with ALB offers multiple advantages:

* Path-based routing – ALB supports routing traffic to different services based on URL paths

* SSL/TLS termination – ALB can handle SSL/TLS termination, reducing load on your pods

* Authentication – ALB supports authentication via Amazon Cognito or OIDC

* AWS WAF – ALB can integrate with AWS WAF for additional security

* Access logs – ALB can log requests to an Amazon S3 bucket for auditing and analysis

## Cleanup

To avoid incurring additional costs, clean up the resources created in this post. Run the provided cleanup.sh script to delete Kubernetes resources (ingress, LeaderWorkerSet, PersistentVolumeClaim, PersistentVolume, AWS Load Balancer Controller, and StorageClass), IAM resources, the FSx for Lustre file system, and the EKS cluster:
```Bash
chmod +x cleanup.sh
./cleanup.sh
```
For more detailed cleanup instructions, including troubleshooting CloudFormation stack deletion errors, refer to the README.md file in the GitHub repository.
## Conclusion
In this post, we demonstrated how to deploy the DeepSeek-R1-Distill-Qwen-32B model on Amazon EKS using vLLMs, with GPU, EFA, and FSx for Lustre integration. This architecture provides a scalable, high-performance system for serving LLM inference workloads.

AWS Deep Learning Containers (DLCs) for vLLM offer an optimized, streamlined environment that simplifies LLM deployment by minimizing environment setup complexity, dependency management, and performance tuning. By leveraging these preconfigured containers, organizations can reduce deployment time and focus on deriving value from their LLM applications.

By combining AWS DLCs, Amazon EKS, P4d instances with NVIDIA A100 GPUs, EFA, and FSx for Lustre, you can achieve optimal performance for LLM inference while maintaining the flexibility and scalability of Kubernetes.

* This solution helps organizations:

* Efficiently deploy LLMs at scale

* Optimize GPU utilization through container orchestration

* Improve inter-node network performance using EFA

* Accelerate model loading with high-performance storage

* Deliver a scalable, high-performance inference API

The complete code and configuration files for this deployment are available in our GitHub repository. We encourage you to try it out and adapt it for your specific use case.