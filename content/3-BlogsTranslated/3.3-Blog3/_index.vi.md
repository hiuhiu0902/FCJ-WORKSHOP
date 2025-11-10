---
title: "Triển khai LLMs trên Amazon EKS bằng cách sử dụng vLLM Deep Learning Containers"
date: "2025-08-14T10:00:00+07:00"
weight: 3
chapter: false
pre: " <b> 3. </b> "
---

> Tác giả: Vishal Naik và Sumeet Tripathi | Ngày: 14 THÁNG 8, 2025 | Trong: [AWS Deep Learning AMIs](https://aws.amazon.com/deep-learning-amis/), [Best Practices](https://aws.amazon.com/blogs/architecture/category/best-practices/), [Expert (400)](https://aws.amazon.com/blogs/architecture/category/learning-levels/expert-400/), [Generative AI](https://aws.amazon.com/generative-ai/), [Technical How-to](https://aws.amazon.com/blogs/architecture/category/post-types/technical-how-to/) | [Permalink](https://aws.amazon.com/blogs/architecture/deploy-llms-on-amazon-eks-using-vllm-deep-learning-containers/) | [Comments](https://aws.amazon.com/blogs/architecture/deploy-llms-on-amazon-eks-using-vllm-deep-learning-containers/#comments)

Các tổ chức phải đối mặt với những thách thức đáng kể khi triển khai các large language models (LLMs) một cách hiệu quả ở quy mô lớn. Các thách thức chính bao gồm tối ưu hóa việc sử dụng tài nguyên GPU, quản lý cơ sở hạ tầng mạng và cung cấp quyền truy cập hiệu quả vào model weights. Khi chạy các distributed inference workloads, các tổ chức thường gặp phải sự phức tạp trong việc điều phối các hoạt động của model trên nhiều nodes. Những thách thức phổ biến bao gồm phân phối hiệu quả các thành phần model trên các GPUs có sẵn, điều phối giao tiếp liền mạch giữa các đơn vị xử lý, và duy trì hiệu suất nhất quán với low latency và high throughput.

[vLLM](https://github.com/vllm-project/vllm) là một thư viện open source dành cho LLM inference và serving nhanh chóng. Các [vLLM AWS Deep Learning Containers (DLCs)](https://aws.amazon.com/releasenotes/aws-deep-learning-containers-for-vllm-inference/) được tối ưu hóa cho khách hàng triển khai vLLMs trên [Amazon Elastic Compute Cloud](https://aws.amazon.com/ec2/) (Amazon EC2), [Amazon Elastic Container Service](https://aws.amazon.com/ecs/) (Amazon ECS), và [Amazon Elastic Kubernetes Service](https://aws.amazon.com/eks/) (Amazon EKS), và được cung cấp miễn phí. Các containers này đóng gói một môi trường được cấu hình sẵn, đã được kiểm thử, hoạt động liền mạch ngay khi sử dụng, bao gồm các dependencies cần thiết như drivers và thư viện để chạy vLLMs một cách hiệu quả, và cung cấp hỗ trợ tích hợp cho [Elastic Fabric Adapter](https://aws.amazon.com/hpc/efa/) (EFA) cho các multi-node inference workloads hiệu suất cao. Bạn không cần phải xây dựng môi trường inference từ đầu nữa. Thay vào đó, bạn có thể cài đặt vLLM DLC và nó sẽ tự động thiết lập và cấu hình môi trường, và bạn có thể bắt đầu triển khai các inference workloads ở quy mô lớn.

Trong bài đăng này, chúng tôi trình bày cách triển khai model [DeepSeek-R1-Distill-Qwen-32B](https://huggingface.co/deepseek-ai/DeepSeek-R1-Distill-Qwen-32B) bằng cách sử dụng AWS DLCs cho vLLMs trên Amazon EKS, giới thiệu cách các containers được xây dựng có mục đích này đơn giản hóa việc triển khai inference engine open source mạnh mẽ này. Giải pháp này có thể giúp bạn giải quyết các thách thức cơ sở hạ tầng phức tạp khi triển khai LLMs trong khi vẫn duy trì hiệu suất và cost-efficiency.

## AWS DLCs

AWS DLCs cung cấp cho các chuyên gia generative AI các môi trường Docker được tối ưu hóa để train và deploy generative AI models trong các pipelines và workflows của họ trên Amazon EC2, Amazon EKS, và Amazon ECS. AWS DLCs nhắm đến các khách hàng self-managed machine learning (ML), những người thích xây dựng và duy trì các môi trường AI/ML của riêng họ, muốn kiểm soát cấp instance đối với cơ sở hạ tầng của họ và quản lý các training và inference workloads của riêng họ. DLCs có sẵn dưới dạng Docker images cho training và inference, và cũng với PyTorch và TensorFlow. DLCs được cập nhật với phiên bản mới nhất của frameworks và drivers, được kiểm tra tính tương thích và bảo mật, và được cung cấp miễn phí. Chúng cũng có thể tùy chỉnh nhanh chóng bằng cách làm theo các hướng dẫn recipe của chúng tôi. Sử dụng AWS DLCs làm khối xây dựng cho môi trường generative AI giúp giảm gánh nặng cho các nhóm vận hành và cơ sở hạ tầng, giảm TCO cho cơ sở hạ tầng AI/ML, tăng tốc phát triển các sản phẩm generative AI, và giúp các nhóm generative AI tập trung vào công việc tạo ra giá trị gia tăng là rút ra các generative AI-powered insights từ dữ liệu của tổ chức.

## Tổng quan về giải pháp

Sơ đồ sau cho thấy sự tương tác giữa Amazon EKS, các EC2 instances hỗ trợ GPU với mạng EFA, và bộ lưu trữ [Amazon FSx for Lustre](https://aws.amazon.com/fsx/lustre/). Các yêu cầu của client chảy qua Application Load Balancer (ALB) đến các vLLM server pods chạy trên các EKS nodes, nơi truy cập model weights được lưu trữ trên FSx for Lustre. Kiến trúc này cung cấp một giải pháp scalable, high-performance để serving LLM inference workloads với optimal cost-efficiency.
> <img src="/images/bl3-1.png" alt="" width="55%">
Sơ đồ sau minh họa DLC stack trên AWS. Stack này thể hiện một kiến trúc toàn diện từ nền tảng EC2 instance thông qua container runtime, các GPU drivers thiết yếu, và các ML frameworks như PyTorch. Sơ đồ phân lớp cho thấy cách CUDA, NCCL, và các thành phần quan trọng khác tích hợp để hỗ trợ các high-performance deep learning workloads.
> <img src="/images/bl3-2.png" alt="" width="55%">
Các vLLM DLCs được tối ưu hóa đặc biệt cho high-performance inference, với hỗ trợ tích hợp cho tensor parallelism và pipeline parallelism trên nhiều GPUs và nodes. Việc tối ưu hóa này cho phép scaling hiệu quả các large models như DeepSeek-R1-Distill-Qwen-32B, mà nếu không thì sẽ rất khó để triển khai và quản lý. Các containers cũng bao gồm các cấu hình CUDA và EFA drivers được tối ưu hóa, tạo điều kiện cho maximum throughput cho các distributed inference workloads. Giải pháp này sử dụng các dịch vụ và thành phần AWS sau:

* **AWS DLCs for vLLMs** – Docker images được cấu hình sẵn, tối ưu hóa nhằm đơn giản hóa việc triển khai và tối đa hóa hiệu suất
* **EKS cluster** – Cung cấp Kubernetes control plane để điều phối containers
* **P4d.24xlarge instances** – [EC2 P4d instances](https://aws.amazon.com/ec2/instance-types/p4/) với 8 NVIDIA A100 GPUs mỗi instance, được cấu hình trong một managed node group
* **Elastic Fabric Adapter** – Giao diện mạng cho phép các high-performance computing applications scale hiệu quả
* **FSx for Lustre** – High-performance file system để lưu trữ model weights
* **LeaderWorkerSet pattern** – Custom Kubernetes resource để triển khai vLLM trong cấu hình distributed
* **AWS Load Balancer Controller** – Quản lý ALB để truy cập bên ngoài

Bằng cách kết hợp các thành phần này, chúng tôi tạo ra một hệ thống inference cung cấp khả năng LLM serving low-latency, high-throughput với minimal operational overhead.

## Điều kiện tiên quyết

Trước khi bắt đầu, hãy đảm bảo bạn có các điều kiện tiên quyết sau:

* Một AWS account có quyền truy cập vào EC2 P4 instances (bạn có thể cần [request a quota increase](https://aws.amazon.com/support/create-case))
* Truy cập vào một terminal đã cài đặt các công cụ sau:
    * [AWS CLI](https://aws.amazon.com/cli/) phiên bản 2.11.0 trở lên
    * [eksctl](https://eksctl.io/) phiên bản 0.150.0 trở lên
    * [kubectl](https://kubernetes.io/docs/tasks/tools/) phiên bản 1.27 trở lên
    * [Helm](https://helm.sh/) phiên bản 3.12.0 trở lên
* Một AWS CLI profile (vllm-profile) được cấu hình với [AWS Identity and Access Management](https://aws.amazon.com/iam/) (IAM) role hoặc user có các quyền sau:
    * Tạo, quản lý và xóa EKS clusters và node groups (xem [Create a Kubernetes cluster on the AWS Cloud](https://docs.aws.amazon.com/eks/latest/userguide/create-cluster.html) để biết thêm chi tiết)
    * Tạo, quản lý và xóa EC2 resources, bao gồm virtual private [clouds (VPCs)](https://aws.amazon.com/vpc/), subnets, security groups, và internet gateways (xem [Identity-based policies for Amazon EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-policies-for-amazon-ec2.html) để biết thêm chi tiết)
    * Tạo và quản lý IAM roles (xem [Identity-based policies and resource-based policies](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_identity-based-vs-resource-based.html) để biết thêm chi tiết)
    * Tạo, cập nhật và xóa [AWS CloudFormation](https://aws.amazon.com/cloudformation/) stacks
    * Tạo, xóa và mô tả FSx file systems (xem [Identity and access management for Amazon FSx for Lustre](https://docs.aws.amazon.com/fsx/latest/LustreGuide/iam-auth.html) để biết thêm chi tiết)
    * Tạo và quản lý Elastic Load Balancers

Giải pháp này có thể được triển khai trong các AWS Regions nơi Amazon EKS, P4d instances, và FSx for Lustre khả dụng. Hướng dẫn này sử dụng us-west-2 Region. Quá trình triển khai hoàn chỉnh mất khoảng 60–90 phút.

Clone GitHub repository của chúng tôi chứa các tệp cấu hình cần thiết:

```bash
# Clone the repository
git clone [https://github.com/aws-samples/sample-aws-deep-learning-containers.git](https://github.com/aws-samples/sample-aws-deep-learning-containers.git)
cd vllm-samples/deepseek/eks
```
## Tạo một EKS cluster
Đầu tiên, chúng tôi tạo một EKS cluster trong us-west-2 Region bằng cách sử dụng tệp cấu hình được cung cấp. Điều này thiết lập Kubernetes control plane sẽ điều phối containers của chúng tôi. Cluster được cấu hình với một VPC, subnets, và security groups được tối ưu hóa để chạy GPU workloads.
```bash
# Update the region in eks-cluster.yaml if needed
sed -i "s|region: us-east-1|region: us-west-2|g" eks-cluster.yaml
# Create the EKS cluster
eksctl create cluster -f eks-cluster.yaml --profile vllm-profile
```

Việc này sẽ mất khoảng 15–20 phút để hoàn thành. Trong thời gian này, eksctl tạo một CloudFormation stack cung cấp các tài nguyên cần thiết cho EKS cluster của bạn, như thể hiện trong ảnh chụp màn hình sau.
> <img src="/images/bl3-3.png" alt="" width="55%">

Bạn có thể xác thực việc tạo cluster bằng mã sau:

```bash
# Verify cluster creation
eksctl get cluster --profile vllm-profile
```
Expected output:
```bash
NAME            REGION          EKSCTL CREATED
vllm-cluster    us-west-2       True
```
Bạn cũng có thể xem cluster được tạo trên Amazon EKS console.

> <img src="/images/bl3-4.png" alt="" width="55%">

## Tạo một node group có hỗ trợ EFA

Tiếp theo, chúng tôi tạo một managed node group với các P4d.24xlarge instances đã bật EFA. Các instances này được trang bị 8 NVIDIA A100 GPUs mỗi instance, cung cấp sức mạnh tính toán đáng kể cho LLM inference. Khi triển khai các EFA-enabled instances như p4d.24xlarge cho các high-performance ML workloads, bạn phải đặt chúng trong các private subnets để tạo điều kiện mạng secure, tối ưu hóa. Bằng cách tự động xác định và sử dụng Availability Zone của một private subnet trong cấu hình node group của bạn, bạn có thể duy trì sự cô lập mạng thích hợp trong khi hỗ trợ giao tiếp high-throughput, low-latency thiết yếu cho distributed training và inference với LLMs. Chúng tôi xác định Availability Zone bằng mã sau:

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
Việc này sẽ mất khoảng 10–15 phút để hoàn thành. Cấu hình EFA đặc biệt quan trọng đối với các multi-node deployments, vì nó cho phép mạng high-throughput, low-latency giữa các nodes. Điều này rất quan trọng đối với các distributed inference workloads nơi giao tiếp giữa các GPUs trên các nodes khác nhau có thể trở thành một bottleneck. Sau khi node group được tạo, cấu hình kubectl để kết nối với cluster:

```bash
# Configure kubectl to connect to the cluster
aws eks update-kubeconfig --name vllm-cluster --region us-west-2 --profile vllm-profile
```
Xác minh rằng các nodes đã sẵn sàng:

```bash
# Check node status
kubectl get nodes
```
Sau đây là một ví dụ về output dự kiến:

```bash
NAME                                          STATUS   ROLES    AGE     VERSION
ip-192-168-xx-xx.us-west-2.compute.internal     Ready    <none>   5m      v1.31.7-eks-xxxx
ip-192-168-yy-yy.us-west-2.compute.internal     Ready    <none>   5m      v1.31.7-eks-xxxx
```
Bạn cũng có thể xem node group được tạo trên Amazon EKS console.
> <img src="/images/bl3-7.png" alt="" width="65%">

## Kiểm tra NVIDIA device pods
Vì chúng tôi đang sử dụng Amazon EKS optimized AMI có hỗ trợ GPU (ami-0ad09867389dc17a1), NVIDIA device plugin đã được bao gồm trong cluster, vì vậy không cần phải cài đặt riêng. Xác minh rằng NVIDIA device plugin đang chạy:

```Bash
# Check NVIDIA device plugin pods
kubectl get pods -n kube-system | grep nvidia
```
Sau đây là một ví dụ về output dự kiến:

```Plaintext
nvidia-device-plugin-daemonset-xxxxx 1/1 Running 0 3m48s
nvidia-device-plugin-daemonset-yyyyy 1/1 Running 0 3m48s
```
Xác minh rằng GPUs có sẵn trong cluster:
```Bash
# Check available GPUs
kubectl get nodes -o json | jq '.items[].status.capacity."[nvidia.com/gpu](https://nvidia.com/gpu)"'
```
Output dự kiến của chúng tôi:
```Plaintext
"8"
"8"
```
## Tạo một FSx for Lustre file system

Để có hiệu suất tối ưu, chúng tôi tạo một FSx for Lustre file system để lưu trữ model weights của mình. FSx for Lustre cung cấp high-throughput, low-latency access vào dữ liệu, điều này cần thiết để tải các large model weights một cách hiệu quả. Chúng tôi sử dụng mã sau:
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
File system được cấu hình với dung lượng lưu trữ 1.2 TB, loại triển khai SCRATCH_2 cho high performance, và security groups cho phép truy cập từ các EKS nodes của chúng tôi. Bạn cũng có thể kiểm tra FSx for Lustre file system trên FSx for Lustre console.
> <img src="/images/bl3-5.png" alt="" width="65%">

## Cài đặt AWS FSx CSI Driver
Để mount FSx for Lustre file system trong các Kubernetes pods của chúng tôi, chúng tôi cài đặt AWS FSx CSI Driver. Driver này cho phép Kubernetes tự động provision và mount FSx for Lustre volumes.

```Bash
# Add the AWS FSx CSI Driver Helm repository
helm repo add aws-fsx-csi-driver [https://kubernetes-sigs.github.io/aws-fsx-csi-driver/](https://kubernetes-sigs.github.io/aws-fsx-csi-driver/)
helm repo update

# Install the AWS FSx CSI Driver
helm install aws-fsx-csi-driver aws-fsx-csi-driver/aws-fsx-csi-driver --namespace kube-system
```
Xác minh rằng AWS FSx CSI Driver đang chạy:

```Bash
# Check AWS FSx CSI Driver pods
kubectl get pods -n kube-system | grep fsx
```
Sau đây là một ví dụ về output dự kiến:

```Plaintext
fsx-csi-controller-xxxx     4/4     Running   0          24s
fsx-csi-controller-yyyy     4/4     Running   0          24s
fsx-csi-node-xxxx           3/3     Running   0          24s
fsx-csi-node-yyyy           3/3     Running   0          24s
```
Tạo Kubernetes resources cho FSx for Lustre
Chúng tôi tạo các Kubernetes resources cần thiết để sử dụng FSx for Lustre file system của chúng tôi:

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
Xác minh rằng các resources đã được tạo thành công:
```Bash
# Check storage class
kubectl get sc fsx-sc

# Check persistent volume
kubectl get pv fsx-lustre-pv

# Check persistent volume claim
kubectl get pvc fsx-lustre-pvc
```
Sau đây là một ví dụ về output dự kiến:

```Plaintext
NAME     PROVISIONER       RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
fsx-sc   fsx.csi.aws.com   Retain          Immediate           false                  1m

NAME            CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                    STORAGECLASS   REASON   AGE
fsx-lustre-pv   1200Gi     RWX            Retain           Bound    default/fsx-lustre-pvc   fsx-sc                  1m

NAME             STATUS   VOLUME          CAPACITY   ACCESS MODES   STORAGECLASS   AGE
fsx-lustre-pvc   Bound    fsx-lustre-pv   1200Gi     RWX            fsx-sc         1m
```
Các resources này bao gồm:

* Một StorageClass định nghĩa cách provision các FSx for Lustre volumes

* Một PersistentVolume đại diện cho FSx for Lustre file system hiện có của chúng tôi

* Một PersistentVolumeClaim mà các pods của chúng tôi sẽ sử dụng để mount file system

## Cài đặt AWS Load Balancer Controller
Để expose vLLM service của chúng tôi ra thế giới bên ngoài, chúng tôi cài đặt AWS Load Balancer Controller. Controller này quản lý ALBs cho các Kubernetes services và ingresses của chúng tôi. Tham khảo Install AWS Load Balancer Controller with Helm để biết thêm chi tiết.

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
Xác minh rằng AWS Load Balancer Controller đang chạy:

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
## Cấu hình security groups cho ALB

Chúng tôi tạo một security group dành riêng cho ALB và cấu hình nó để cho phép inbound traffic trên port 80 từ các địa chỉ IP client của chúng tôi. Chúng tôi cũng cấu hình node security group để cho phép traffic từ ALB security group đến vLLM service port.

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
Xác minh rằng các security groups đã được tạo và cấu hình chính xác:

```Bash
# Verify ALB security group
aws --profile vllm-profile ec2 describe-security-groups --group-ids $ALB_SG --query "SecurityGroups[0].IpPermissions"
The following is the expected output for the ALB security group:

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
## Triển khai vLLM server
Cuối cùng, chúng tôi triển khai vLLM server bằng cách sử dụng LeaderWorkerSet pattern. Các AWS DLCs cung cấp một môi trường được tối ưu hóa, hợp lý hóa, giảm thiểu sự phức tạp thường liên quan đến việc triển khai LLMs. Các vLLM DLCs được cấu hình sẵn với các tính năng sau:

  * Optimized CUDA libraries cho maximum GPU utilization

* EFA drivers và cấu hình cho high-speed node-to-node communication

* Thiết lập Ray framework cho distributed computing

* Cài đặt vLLM được điều chỉnh hiệu suất với hỗ trợ tensor và pipeline parallelism

Giải pháp đóng gói sẵn này giảm đáng kể thời gian triển khai, nhu cầu thiết lập môi trường phức tạp, quản lý dependency và performance tuning mà nếu không sẽ đòi hỏi chuyên môn đặc biệt.

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
Việc triển khai sẽ bắt đầu ngay lập tức, nhưng pod có thể vẫn ở trạng thái ContainerCreating trong vài phút (5–15 phút) trong khi nó pull container image hỗ trợ GPU lớn. Sau khi container bắt đầu, sẽ mất thêm thời gian (10–15 phút) để download và load model DeepSeek. Bạn có thể theo dõi tiến trình bằng mã sau:

```Bash
# Monitor pod status
kubectl get pods

# Check pod logs
kubectl logs -f <pod-name>
Here is the out put of one of the pods
Kubectl logs -f vllm-deepseek-32b-lws-0
```
Sau đây là output của một trong các pods khi đang chạy:
```Plaintext
NAME                    READY   STATUS    RESTARTS   AGE
vllm-deepseek-32b-lws-0   1/1     Running   0          10m
vllm-deepseek-32b-lws-0-1 1/1     Running   0          10m
```
Chúng tôi cũng triển khai một ingress resource cấu hình ALB để định tuyến traffic đến vLLM service của chúng tôi:

```Bash
# Apply the ingress (only after the controller is running)
kubectl apply -f vllm-deepseek-32b-lws-ingress.yaml
```
Bạn có thể kiểm tra trạng thái của ingress bằng mã sau:

```Bash
# Check ingress status
kubectl get ingress
```
Sau đây là một ví dụ về output dự kiến:

```Plaintext
NAME                          CLASS   HOSTS   ADDRESS                                                                      PORTS   AGE
vllm-deepseek-32b-lws-ingress   alb     * k8s-default-vllmdeep-xxxxxxxx-xxxxxxxxxx.us-west-2.elb.amazonaws.com     80      5m
```
## Kiểm tra việc triển khai
Khi việc triển khai hoàn tất, chúng tôi có thể kiểm tra vLLM server của mình. Nó cung cấp các API endpoints sau:

* /v1/completions – Dành cho text completions

* /v1/chat/completions – Dành cho chat completions

* /v1/embeddings – Dành cho tạo embeddings

* /v1/models – Dành cho liệt kê các models có sẵn

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
 > <img src="/images/bl3-6.png" alt="" width="100%">
Sau đây là một ví dụ về output dự kiến:

```JSON

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
Bạn cũng có thể kiểm tra chat completions API:

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
Nếu bạn gặp lỗi, hãy kiểm tra logs của các vLLM pods:

```Bash
# Troubleshooting
kubectl logs -f <pod-name>
```
## Các cân nhắc về hiệu suất

Trong phần này, chúng tôi thảo luận về các cân nhắc hiệu suất khác nhau.

### Elastic Fabric Adapter
EFA cung cấp các lợi ích hiệu suất đáng kể cho các distributed inference workloads:

* Reduced latency – Latency thấp hơn và nhất quán hơn cho giao tiếp giữa các GPUs trên các nodes

* Higher throughput – Throughput cao hơn cho việc truyền dữ liệu giữa các nodes

* Improved scaling – Hiệu quả scaling tốt hơn trên nhiều nodes

* Better performance – Hiệu suất được cải thiện đáng kể cho các distributed inference workloads

### Tích hợp FSx for Lustre
Sử dụng FSx for Lustre cho bộ lưu trữ model cung cấp một số lợi ích:

* Persistent storage – Model weights được lưu trữ trên FSx for Lustre file system và duy trì qua các lần pod restarts

* Faster loading – Sau lần download ban đầu, việc load model nhanh hơn nhiều

* Shared storage – Nhiều pods có thể truy cập cùng một model weights

* High performance – FSx for Lustre cung cấp high-throughput, low-latency access vào model weights

### Application Load Balancer
Sử dụng AWS Load Balancer Controller với ALB mang lại một số lợi thế:

* Path-based routing – ALB hỗ trợ định tuyến traffic đến các dịch vụ khác nhau dựa trên URL path

* SSL/TLS termination – ALB có thể xử lý SSL/TLS termination, giảm tải cho các pods của bạn

* Authentication – ALB hỗ trợ authentication thông qua Amazon Cognito hoặc OIDC

* AWS WAF – ALB có thể được tích hợp với AWS WAF để bảo mật bổ sung

* Access logs – ALB có thể ghi log các yêu cầu vào một Amazon Simple Storage Service (Amazon S3) bucket để auditing và phân tích

## Dọn dẹp
Để tránh phát sinh chi phí bổ sung, hãy dọn dẹp các resources được tạo trong bài đăng này. Chạy script ./cleanup.sh được cung cấp để dọn dẹp các Kubernetes resources (ingress, LeaderworkerSet, PersistentVolumeClaim, PersistentVolume, AWS Load Balancer Controller, và storage class), IAM resources, FSX for Lustre file system, và EKS cluster:

```Bash
chmod +x cleanup.sh
./cleanup.sh
```
Để biết thêm hướng dẫn dọn dẹp chi tiết, bao gồm khắc phục sự cố lỗi xóa CloudFormation stack, hãy tham khảo tệp README.md trong GitHub repository.

Kết luận
Trong bài đăng này, chúng tôi đã trình bày cách triển khai model DeepSeek-R1-Distill-Qwen-32B trên Amazon EKS bằng cách sử dụng vLLMs, với hỗ trợ GPU, EFA, và tích hợp FSx for Lustre. Kiến trúc này cung cấp một hệ thống scalable, high-performance để serving LLM inference workloads. AWS Deep Learning Containers for vLLM cung cấp một môi trường được tối ưu hóa, hợp lý hóa, đơn giản hóa việc triển khai LLM bằng cách giảm thiểu sự phức tạp của cấu hình môi trường, quản lý dependency và performance tuning. Bằng cách sử dụng các containers được cấu hình sẵn này, các tổ chức có thể giảm thời gian triển khai và tập trung vào việc tạo ra giá trị từ các ứng dụng LLM của họ. Bằng cách kết hợp AWS DLCs với Amazon EKS, P4d instances với NVIDIA A100 GPUs, EFA, và FSx for Lustre, bạn có thể đạt được hiệu suất tối ưu cho LLM inference trong khi vẫn duy trì tính linh hoạt và khả năng mở rộng của Kubernetes.

Giải pháp này giúp các tổ chức:

* Triển khai LLMs hiệu quả ở quy mô lớn

* Tối ưu hóa việc sử dụng tài nguyên GPU bằng cách điều phối container

* Cải thiện hiệu suất mạng giữa các nodes với EFA

* Tăng tốc model loading với high-performance storage

* Cung cấp một inference API scalable, high performance

Mã hoàn chỉnh và các tệp cấu hình cho việc triển khai này có sẵn trong GitHub repository của chúng tôi. Chúng tôi khuyến khích bạn dùng thử và điều chỉnh nó cho trường hợp sử dụng cụ thể của mình.