# Lab 16 — Create an EC2 Auto Scaling Group

> **Level:** Beginner → Intermediate  
> **Service:** Amazon EC2 Auto Scaling  
> **Region:** `eu-north-1` (Stockholm)  
> **Architecture:** Multi-AZ EC2 Auto Scaling  
> **Next Lab:** Lab 17 — Build an ALB + Auto Scaling Web Architecture

---

## 📌 Lab Overview

In this lab, you will create an **EC2 Auto Scaling Group (ASG)** that can automatically:

- Launch EC2 instances
    
- Maintain a minimum number of instances
    
- Scale out when CPU utilization increases
    
- Scale in when CPU utilization decreases
    
- Replace an instance if it is terminated or becomes unhealthy
    
- Distribute instances across multiple Availability Zones
    

By the end of this lab, you will understand how **Launch Templates, Auto Scaling Groups, Target Tracking, Scaling Policies, and Self-Healing** work together.

---

## 🎯 Learning Objectives

After completing this lab, you should be able to:

- Create an EC2 Security Group
    
- Create and configure an EC2 Launch Template
    
- Create an Auto Scaling Group
    
- Configure Minimum, Desired, and Maximum capacity
    
- Deploy instances across multiple Availability Zones
    
- Configure Target Tracking Scaling
    
- Generate CPU load for testing
    
- Observe Scale-Out and Scale-In behavior
    
- Test Auto Scaling self-healing
    
- Clean up AWS resources safely
    

---

# 🏗️ Architecture

During this lab, you will build the following architecture:

```text
                    ┌──────────────────────────┐
                    │      AWS VPC              │
                    │   MetaPi-MultiAZ-VPC      │
                    │      10.20.0.0/16          │
                    └────────────┬───────────────┘
                                 │
                 ┌───────────────┴───────────────┐
                 │                               │
        ┌────────▼─────────┐           ┌────────▼─────────┐
        │ Public Subnet A  │           │ Public Subnet B  │
        │  eu-north-1a     │           │  eu-north-1b     │
        │ 10.20.1.0/24     │           │ 10.20.2.0/24     │
        └────────┬─────────┘           └────────┬─────────┘
                 │                               │
                 └───────────────┬───────────────┘
                                 │
                    ┌────────────▼────────────┐
                    │   Auto Scaling Group    │
                    │   MetaPi-Lab16-ASG      │
                    │                          │
                    │ Min: 1                  │
                    │ Desired: 1               │
                    │ Max: 4                   │
                    └────────────┬─────────────┘
                                 │
                       ┌─────────▼─────────┐
                       │  Launch Template  │
                       │ MetaPi-Lab16-LT   │
                       └─────────┬─────────┘
                                 │
                    ┌────────────▼────────────┐
                    │       EC2 Instances     │
                    │                          │
                    │ Apache Web Server       │
                    │ Ubuntu                  │
                    │ t3.micro               │
                    └─────────────────────────┘
```

> **Note:** Lab 16 exposes the EC2 instances directly for learning. In **Lab 17**, you will place the Auto Scaling Group behind an **Application Load Balancer (ALB)**.

---

# 📋 Prerequisites

Before starting this lab, make sure you already have:

- An AWS account
    
- Access to the AWS Management Console
    
- An AWS Region selected
    
- A VPC with public subnets
    
- Internet Gateway attached to the VPC
    
- Public route table configured
    
- A key pair available for SSH access
    

### Existing Lab Resources

This lab assumes the following resources are available:

|Resource|Name|Value|
|---|---|---|
|VPC|`MetaPi-MultiAZ-VPC`|`10.20.0.0/16`|
|Public Subnet A|`MetaPi-Public-A`|`10.20.1.0/24`|
|Public Subnet B|`MetaPi-Public-B`|`10.20.2.0/24`|
|Availability Zone A|—|`eu-north-1a`|
|Availability Zone B|—|`eu-north-1b`|

> If you are following this repository from the beginning, complete the previous VPC/Multi-AZ labs first.

---

# 🧩 Step 1 — Create the Auto Scaling Security Group

## What are we doing?

We need a Security Group for the EC2 instances that will be launched by the Auto Scaling Group.

Create:

```text
MetaPi-Lab16-ASG-SG
```

## AWS Console

Go to:

```text
AWS Console
→ EC2
→ Security Groups
→ Create security group
```

### Basic Configuration

|Setting|Value|
|---|---|
|Security group name|`MetaPi-Lab16-ASG-SG`|
|Description|`Security group for MetaPi Lab 16 Auto Scaling instances`|
|VPC|`MetaPi-MultiAZ-VPC`|

### Inbound Rules

#### HTTP

```text
Type: HTTP
Port: 80
Source: Anywhere-IPv4
```

#### SSH

```text
Type: SSH
Port: 22
Source: My IP
```

### Outbound Rules

Leave the default outbound rule:

```text
All traffic → 0.0.0.0/0
```

Click:

```text
Create security group
```

### ✅ Verify

Confirm that:

```text
MetaPi-Lab16-ASG-SG
```

exists in:

```text
MetaPi-MultiAZ-VPC
```

---

# 🚀 Step 2 — Create the Launch Template

## What is a Launch Template?

A **Launch Template** defines how Auto Scaling should create EC2 instances.

It can contain:

- AMI
    
- Instance type
    
- Key pair
    
- Security groups
    
- Storage
    
- User Data
    
- Metadata configuration
    

Instead of manually configuring every new EC2 instance, Auto Scaling uses this template.

---

## AWS Console

Go to:

```text
EC2
→ Launch Templates
→ Create launch template
```

### Launch Template Configuration

|Setting|Value|
|---|---|
|Launch template name|`MetaPi-Lab16-LT`|
|AMI|Ubuntu Server LTS|
|Instance type|Approved small training instance|
|Key pair|Your lab key|
|Security Group|`MetaPi-Lab16-ASG-SG`|
|Subnet|Do not specify|

> **Important:** Do not select a specific subnet inside the Launch Template. The Auto Scaling Group will decide which configured subnet/AZ to use.

---

## User Data

Scroll to **Advanced details → User data** and enter:

```bash
#!/bin/bash
apt-get update -y
apt-get install -y apache2

cat > /var/www/html/index.html <<EOF
<h1>MetaPi Auto Scaling Lab</h1>
<p>Instance: $(hostname)</p>
EOF

systemctl enable --now apache2
```

### What does User Data do?

When a new EC2 instance starts, the script automatically:

1. Updates package information
    
2. Installs Apache
    
3. Creates a custom web page
    
4. Starts Apache
    
5. Configures Apache to start automatically
    

This means every EC2 instance launched by the ASG will have the same web server configuration.

Click:

```text
Create launch template
```

### ✅ Verify

Confirm that:

```text
MetaPi-Lab16-LT
```

exists.

---

# ☁️ Step 3 — Create the Auto Scaling Group

## What is an Auto Scaling Group?

An **Auto Scaling Group (ASG)** manages a collection of EC2 instances.

It can:

- Maintain desired capacity
    
- Launch replacement instances
    
- Scale out
    
- Scale in
    
- Spread instances across Availability Zones
    

---

## AWS Console

Go to:

```text
EC2
→ Auto Scaling Groups
→ Create Auto Scaling group
```

### Step 1 — Choose Launch Template

Set:

```text
Name:
MetaPi-Lab16-ASG
```

Select:

```text
Launch Template:
MetaPi-Lab16-LT
```

Use the default version.

Continue.

---

# 🌐 Step 4 — Configure Network

Select:

```text
VPC:
MetaPi-MultiAZ-VPC
```

Select both public subnets:

```text
MetaPi-Public-A
MetaPi-Public-B
```

These should correspond to:

```text
eu-north-1a
eu-north-1b
```

### Why two Availability Zones?

Using multiple Availability Zones improves availability.

If one Availability Zone experiences a problem, instances can still operate in another Availability Zone.

---

# ⚙️ Step 5 — Configure Group Size

Set:

```text
Desired capacity: 1
Minimum capacity: 1
Maximum capacity: 4
```

### What do these values mean?

#### Minimum = 1

The ASG should maintain at least:

```text
1 instance
```

#### Desired = 1

The ASG starts by trying to maintain:

```text
1 instance
```

#### Maximum = 4

The ASG cannot automatically scale beyond:

```text
4 instances
```

Therefore:

```text
Minimum  → 1
Desired  → 1
Maximum  → 4
```

Continue and create the Auto Scaling Group.

---

# 🔍 Step 6 — Verify the First Instance

Open:

```text
EC2
→ Auto Scaling Groups
→ MetaPi-Lab16-ASG
→ Instance management
```

Wait until the instance becomes:

```text
InService
Healthy
```

Then go to:

```text
EC2
→ Instances
```

Find the instance managed by:

```text
MetaPi-Lab16-ASG
```

Copy its **Public IPv4 address**.

Open:

```text
http://<INSTANCE_PUBLIC_IP>
```

You should see:

```text
MetaPi Auto Scaling Lab
Instance: ip-10-20-x-x
```

### ✅ Expected Result

The hostname may be different on every instance.

For example:

```text
MetaPi Auto Scaling Lab
Instance: ip-10-20-1-93
```

This proves that the Launch Template successfully configured the EC2 instance.

---

# 📈 Step 7 — Add Target Tracking Scaling

## What is Target Tracking?

Target Tracking tells Auto Scaling:

> "Try to maintain this metric around my target value."

For this lab, we will maintain:

```text
Average CPU utilization = 50%
```

---

## AWS Console

Open:

```text
EC2
→ Auto Scaling Groups
→ MetaPi-Lab16-ASG
→ Automatic scaling
```

Create a dynamic scaling policy.

Configure:

```text
Policy type:
Target tracking scaling
```

Metric:

```text
Average CPU utilization
```

Target value:

```text
50
```

Enable:

```text
Scale in
```

Save the policy.

### Expected Policy

```text
Policy:
MetaPi-Lab16-CPU-Scaling

Metric:
Average CPU utilization

Target:
50%

Scale in:
Enabled
```

---

# 💻 Step 8 — Connect to the EC2 Instance

Open the EC2 instance and select:

```text
Connect
→ SSH client
```

AWS will provide an SSH command similar to:

```bash
ssh -i "your-key.pem" ubuntu@<PUBLIC_IP>
```

Example:

```bash
ssh -i "MetaPi-Lab16-LT.pem" ubuntu@<PUBLIC_IP>
```

> Replace the key filename and public IP with your own values.

---

# 🔥 Step 9 — Install stress-ng

Inside the EC2 instance, run:

```bash
sudo apt update
```

Then:

```bash
sudo apt install stress-ng -y
```

Verify installation:

```bash
stress-ng --version
```

---

# 🧪 Step 10 — Generate CPU Load

Run:

```bash
stress-ng --cpu 2 --timeout 10m
```

This creates CPU workload for approximately:

```text
10 minutes
```

### What is happening?

The instance CPU utilization increases.

CloudWatch monitors the CPU metric.

The Target Tracking policy evaluates the metric against the:

```text
50% target
```

If the ASG determines that additional capacity is required, it can launch additional instances.

> **Important:** Scaling does not necessarily happen immediately. CloudWatch metric collection, evaluation, instance warm-up, and Auto Scaling decision timing can introduce a delay.

---

# 📊 Step 11 — Observe Scale Out

Go to:

```text
EC2
→ Auto Scaling Groups
→ MetaPi-Lab16-ASG
→ Activity
```

Watch for scaling activities.

You may see events such as:

```text
Launching a new EC2 instance
```

The desired capacity may change:

```text
1 → 2
```

and potentially:

```text
2 → 3
2 → 4
```

depending on the observed CPU load and scaling decisions.

### Check Instance Management

Go to:

```text
Instance management
```

You may see multiple instances:

```text
Instance 1 → InService
Instance 2 → InService
...
```

### 🎯 Goal

Demonstrate that the ASG can automatically **scale out** when additional capacity is required.

---

# 📉 Step 12 — Observe Scale In

Once the stress command finishes, CPU utilization should eventually fall.

Wait for the scaling policy to evaluate the lower CPU utilization.

Go to:

```text
Auto Scaling Groups
→ MetaPi-Lab16-ASG
→ Activity
```

The ASG may gradually reduce capacity.

For example:

```text
4 → 3
3 → 2
2 → 1
```

The exact timing depends on AWS metric evaluation and Auto Scaling behavior.

### Important

Do **not** manually terminate instances during this step.

Manual termination is reserved for the self-healing test in the next step.

---

# ❤️ Step 13 — Test Self-Healing

Now we will intentionally terminate an ASG-managed EC2 instance.

## Select the Instance

Go to:

```text
EC2
→ Instances
```

Select an instance whose:

```text
Auto Scaling Group name
```

is:

```text
MetaPi-Lab16-ASG
```

Then:

```text
Instance state
→ Terminate instance
```

Confirm termination.

---

## Observe Auto Scaling Activity

Go to:

```text
EC2
→ Auto Scaling Groups
→ MetaPi-Lab16-ASG
→ Activity
```

You should eventually see events similar to:

```text
Terminating EC2 instance
```

followed by:

```text
Launching a new EC2 instance
```

The reason will indicate that Auto Scaling is replacing the terminated/unhealthy instance.

### 🎯 Expected Result

Before:

```text
Desired capacity = 1
Running instances = 1
```

After manually terminating the instance:

```text
Running instances = 0
```

Auto Scaling detects the capacity loss and launches:

```text
Replacement instance
```

Eventually:

```text
Desired capacity = 1
Running instances = 1
```

### 💡 Concept

This is called **self-healing**.

The ASG continuously tries to maintain the desired number of healthy instances.

---

# 🧠 Step 14 — Understand the ASG Components

The complete relationship is:

```text
                 Launch Template
                        │
                        │ defines
                        ▼
              ┌────────────────────┐
              │ Auto Scaling Group │
              └─────────┬──────────┘
                        │
             ┌──────────┼──────────┐
             │          │          │
             ▼          ▼          ▼
           Min       Desired      Max
            1           1          4
                        │
                        ▼
              ┌──────────────────┐
              │  EC2 Instances   │
              └────────┬─────────┘
                       │
              ┌────────┴────────┐
              ▼                 ▼
        eu-north-1a        eu-north-1b
       Public Subnet A   Public Subnet B
```

---

# 🔄 Scaling Flow

### Normal condition

```text
Desired = 1
Running = 1
```

### High CPU

```text
CPU increases
      ↓
CloudWatch metric
      ↓
Target Tracking evaluates
      ↓
ASG scales out
      ↓
New EC2 instance
```

### Low CPU

```text
CPU decreases
      ↓
CloudWatch metric
      ↓
Target Tracking evaluates
      ↓
ASG scales in
      ↓
EC2 capacity decreases
```

### Instance failure/termination

```text
EC2 instance terminated
          ↓
ASG detects capacity loss
          ↓
Replacement launched
          ↓
Desired capacity restored
```

---

# 📝 Step 15 — Lab Verification Checklist

Before finishing the lab, verify the following:

### Security Group

-  `MetaPi-Lab16-ASG-SG` created
    
-  HTTP 80 allowed
    
-  SSH 22 allowed from My IP
    

### Launch Template

-  `MetaPi-Lab16-LT` exists
    
-  Ubuntu AMI configured
    
-  Approved instance type configured
    
-  Lab key configured
    
-  ASG Security Group configured
    
-  User Data installs Apache
    

### Auto Scaling Group

-  `MetaPi-Lab16-ASG` exists
    
-  Correct Launch Template selected
    
-  `MetaPi-MultiAZ-VPC` selected
    
-  `MetaPi-Public-A` selected
    
-  `MetaPi-Public-B` selected
    
-  Two Availability Zones configured
    

### Capacity

-  Minimum = `1`
    
-  Desired = `1`
    
-  Maximum = `4`
    

### Scaling

-  Target Tracking policy exists
    
-  Metric = Average CPU utilization
    
-  Target = `50%`
    
-  Scale-in enabled
    
-  Scale-out observed
    

### Self-Healing

-  ASG-managed instance terminated
    
-  Replacement instance launched
    
-  Desired capacity restored
    

---

# 🧹 Step 16 — Cleanup

> **Important:** AWS resources can incur charges. Clean up resources when the lab is complete.

There are two safe approaches.

## Option A — Delete the Auto Scaling Group

Go to:

```text
EC2
→ Auto Scaling Groups
→ MetaPi-Lab16-ASG
→ Actions
→ Delete Auto Scaling group
```

Confirm deletion.

The ASG-managed instances will be terminated as part of the deletion.

Wait until:

```text
Auto Scaling groups = 0
```

---

## Option B — Reduce Capacity First

If you want to observe the termination process manually, you can first set:

```text
Desired capacity: 0
Minimum capacity: 0
Maximum capacity: 4
```

Wait for the ASG-managed instances to terminate.

Then delete the ASG.

---

## Delete the Launch Template

After the ASG has been deleted:

```text
EC2
→ Launch Templates
→ MetaPi-Lab16-LT
→ Actions
→ Delete template
```

Confirm deletion.

---

## Delete the Security Group

After confirming that no EC2 instances are using it:

```text
EC2
→ Security Groups
→ MetaPi-Lab16-ASG-SG
→ Actions
→ Delete security groups
```

Confirm deletion.

---

# ⚠️ Cleanup Safety Checklist

Before deleting anything, make sure you are deleting only Lab 16 resources:

```text
MetaPi-Lab16-ASG
MetaPi-Lab16-LT
MetaPi-Lab16-ASG-SG
```

### Do NOT delete:

```text
MetaPi-MultiAZ-VPC
MetaPi-Public-A
MetaPi-Public-B
```

Those resources may be required by later labs.

Also avoid deleting resources belonging to:

```text
Lab 14
Lab 15
```

unless those labs are specifically being cleaned up.

---

# 🎓 What You Learned

After completing this lab, you should understand:

### Launch Template

Defines **how EC2 instances should be launched**.

### Auto Scaling Group

Defines **how many instances should exist and how they should be managed**.

### Minimum Capacity

Defines the lowest number of instances Auto Scaling should maintain.

### Desired Capacity

Defines the normal/current target number of instances.

### Maximum Capacity

Defines the highest number of instances Auto Scaling can launch.

### Target Tracking

Allows Auto Scaling to react to CloudWatch metrics such as CPU utilization.

### Scale Out

Adds EC2 instances when additional capacity is required.

### Scale In

Removes EC2 capacity when demand decreases.

### Self-Healing

Automatically replaces terminated or unhealthy instances.

### Multi-AZ

Allows instances to operate across multiple Availability Zones for improved availability.

---

# 🏆 Lab Completion Criteria

You have successfully completed **Lab 16** when you can demonstrate:

```text
✅ Launch Template created
        ↓
✅ Auto Scaling Group created
        ↓
✅ EC2 instance launched automatically
        ↓
✅ Apache configured through User Data
        ↓
✅ Target Tracking configured at 50% CPU
        ↓
✅ Scale Out demonstrated
        ↓
✅ Scale In demonstrated
        ↓
✅ Self-Healing demonstrated
        ↓
✅ Resources cleaned up
```

---

# 📚 Key AWS Concepts

|Concept|Purpose|
|---|---|
|EC2|Virtual server|
|Launch Template|Defines EC2 launch configuration|
|Auto Scaling Group|Manages EC2 capacity|
|Target Tracking|Maintains a metric around a target|
|CloudWatch|Provides monitoring metrics|
|Scale Out|Adds instances|
|Scale In|Removes instances|
|Self-Healing|Replaces failed/terminated instances|
|Availability Zone|Isolated AWS infrastructure location|
|Multi-AZ|Distributes resources across AZs|

---

# 🚀 Next Lab

## Lab 17 — Build an ALB + Auto Scaling Web Architecture

In the next lab, we will improve this architecture by introducing an:

```text
                    Internet
                       │
                       ▼
              Application Load
                 Balancer
                       │
              ┌────────┴────────┐
              ▼                 ▼
          EC2 Instance      EC2 Instance
              │                 │
              └────────┬────────┘
                       │
                 Auto Scaling
                    Group
```

This will introduce:

- Application Load Balancer
    
- Target Groups
    
- Health Checks
    
- Load Balancing
    
- Auto Scaling integration
    
- High availability
    
- Production-style web architecture
    

---

## ⭐ Lab 16 Summary

> **You created an elastic, multi-AZ, self-healing EC2 infrastructure using an Auto Scaling Group and Target Tracking scaling policy.**
> 
> **The next step is to put the Auto Scaling web servers behind an Application Load Balancer.**