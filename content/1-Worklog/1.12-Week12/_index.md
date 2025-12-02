---
title: "Week 12 Worklog"
date: "2025-11-24T09:00:00+07:00"
weight: 12
chapter: false
pre: " <b> 1.12. </b> "
---

### Week 12 Objectives:
* Validate resilience and close project.
* Perform Chaos Engineering tests against the Java Application.

### Tasks:
| Day | Task | Start Date | Completion Date | Reference Material |
| --- | --- | --- | --- | --- |
| 1 | **Test HA:**<br>- Terminate EC2 running the Jar file, watch ASG launch new one. | 24/11/2025 | 24/11/2025 | |
| 2 | **Test DB Failover:**<br>- Reboot RDS with failover option. | 25/11/2025 | 25/11/2025 | |
| 3 | **Test Transaction Rollback:**<br>- Simulate payment failure during order creation. | 26/11/2025 | 26/11/2025 | |
| 4 | **Documentation:**<br>- Post-Mortem report. | 27/11/2025 | 27/11/2025 | |
| 5 | **Project Close:**<br>- **DESTROY ALL RESOURCES.** | 28/11/2025 | 28/11/2025 | |

### 🧠 Extra Knowledge: ACID Properties in Testing
Beyond infrastructure chaos (killing servers), I focused on **Data Integrity**.
I verified that if `OrderService.handlePaymentFailure()` is triggered, the system correctly rolls back the stock status from `PENDING` to `UNUSED`. This confirms the **A**tomicity and **C**onsistency in ACID properties of my MySQL database handled by Spring Transaction.

### 💻 Automation Code: Chaos Monkey Script
I used a Python script (external to the Java app) to randomly terminate the EC2 instance hosting my Spring Boot app to verify ASG recovery.

**File:** `chaos_test.py`
```python
import boto3
import random

def kill_random_instance():
    ec2 = boto3.client('ec2', region_name='ap-southeast-1')
    
    # 1. Get list of running instances for the project
    response = ec2.describe_instances(
        Filters=[
            {'Name': 'tag:Project', 'Values': ['GameCard']},
            {'Name': 'instance-state-name', 'Values': ['running']}
        ]
    )
    
    instances = []
    for reservation in response['Reservations']:
        for instance in reservation['Instances']:
            instances.append(instance['InstanceId'])

    if not instances:
        print("No instances found to kill!")
        return

    # 2. Pick a random victim
    victim_id = random.choice(instances)
    
    # 3. Terminate it
    print(f"🔥 Terminating instance: {victim_id} to test Auto Scaling...")
    ec2.terminate_instances(InstanceIds=[victim_id])
    print("✅ Termination signal sent. Check Console for new instance!")

if __name__ == '__main__':
    kill_random_instance()