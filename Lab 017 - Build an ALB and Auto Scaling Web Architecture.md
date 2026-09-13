# Lab 17 — Build an ALB and Auto Scaling Web Architecture

In this lab, you will build a highly available and self-healing web architecture using:

- Amazon VPC
    
- NAT Gateway
    
- Application Load Balancer (ALB)
    
- Target Group
    
- EC2 Launch Template
    
- Auto Scaling Group (ASG)
    
- Security Groups
    
- Multi-AZ private subnets
    

By the end of this lab, users will access the application through a public ALB, while the EC2 application servers remain inside private subnets.

---

## 🏗️ Architecture

```text
                         Internet
                            |
                            v
                    Internet-facing ALB
                       HTTP : 80
                       /       \
                      /         \
              Public-A          Public-B
                  |                 |
                  +-------+---------+
                          |
                          v
                  Target Group
                          |
                Auto Scaling Group
                  /             \
                 /               \
          Private-A             Private-B
             EC2                   EC2
                 \               /
                  \             /
                   NAT Gateway
                       |
                       v
                  Internet Gateway
                       |
                    Internet
```

### Traffic flow

User traffic:

```text
Internet
   ↓
ALB
   ↓
Target Group
   ↓
Private EC2 instances
```

Private EC2 outbound traffic:

```text
Private EC2
   ↓
Private Route Table
   ↓
NAT Gateway
   ↓
Internet Gateway
   ↓
Internet
```

> **Important:** The ALB is public, but the application EC2 instances are private and do not need public IPv4 addresses.

---

# Lab Objectives

After completing this lab, you should be able to:

- Create an internet-facing Application Load Balancer
    
- Configure ALB and application Security Groups
    
- Create a Target Group
    
- Create an EC2 Launch Template
    
- Deploy EC2 instances using an Auto Scaling Group
    
- Place application servers in private subnets
    
- Configure ELB health checks
    
- Test load balancing across multiple instances
    
- Understand Auto Scaling self-healing
    
- Configure target tracking at 50% CPU
    
- Understand Multi-AZ architecture
    
- Safely clean up lab resources
    

---

# Before You Start

Make sure the following existing VPC resources are available.

### VPC

```text
Name: MetaPi-MultiAZ-VPC
CIDR: 10.20.0.0/16
```

### Public Subnets

```text
MetaPi-Public-A
MetaPi-Public-B
```

### Private Subnets

```text
MetaPi-Private-A
MetaPi-Private-B
```

### Important

Use the existing `MetaPi-MultiAZ-VPC`.

**Do not create another VPC for this lab.**

Also, do not modify unrelated VPCs, NAT Gateways, ALBs, or Security Groups from previous labs.

---

# Step 1 — Prepare the Multi-AZ VPC

We will use the existing Multi-AZ VPC.

Open:

```text
AWS Console
→ VPC
→ Your VPCs
```

Find:

```text
MetaPi-MultiAZ-VPC
```

Confirm:

```text
CIDR: 10.20.0.0/16
State: Available
```

Then open:

```text
VPC
→ Subnets
```

Confirm these four subnets exist:

```text
MetaPi-Public-A
MetaPi-Public-B
MetaPi-Private-A
MetaPi-Private-B
```

### Expected design

```text
AZ-A
├── Public-A
└── Private-A

AZ-B
├── Public-B
└── Private-B
```

### Checkpoint

Before continuing, make sure:

-  Multi-AZ VPC exists
    
-  Public-A exists
    
-  Public-B exists
    
-  Private-A exists
    
-  Private-B exists
    

---

# Step 2 — Create a Training NAT Gateway

The private EC2 instances need outbound internet access to install Apache through User Data.

For this training lab, we will use **one NAT Gateway** to control cost.

> ⚠️ **Cost Warning:** NAT Gateway is a paid AWS resource. Create it only when starting the lab and delete it during cleanup.

---

## 2.1 Allocate an Elastic IP

Open:

```text
VPC
→ Elastic IPs
```

Click:

```text
Allocate Elastic IP address
```

Keep the default settings and allocate it.

Give it a name if desired:

```text
MetaPi-Lab17-NAT-EIP
```

---

## 2.2 Create the NAT Gateway

Go to:

```text
VPC
→ NAT gateways
→ Create NAT gateway
```

Configure:

```text
Name:
MetaPi-Lab17-NAT

Subnet:
MetaPi-Public-A

Connectivity type:
Public

Elastic IP:
Select the Elastic IP created above
```

Click:

```text
Create NAT gateway
```

### Wait!

Do not continue immediately.

Wait until the NAT Gateway becomes:

```text
Available
```

---

## 2.3 Add NAT Route to Private-A

Go to:

```text
VPC
→ Route tables
```

Open the route table associated with:

```text
MetaPi-Private-A
```

Go to:

```text
Routes
→ Edit routes
→ Add route
```

Configure:

```text
Destination:
0.0.0.0/0

Target:
NAT Gateway
→ MetaPi-Lab17-NAT
```

Save changes.

---

## 2.4 Add NAT Route to Private-B

Repeat the same process for the route table associated with:

```text
MetaPi-Private-B
```

Add:

```text
0.0.0.0/0
→ MetaPi-Lab17-NAT
```

### Checkpoint

Both private route tables should contain:

```text
10.20.0.0/16 → local
0.0.0.0/0    → MetaPi-Lab17-NAT
```

> **Do not delete the ********10.20.0.0/16 → local******** route.**

---

# Step 3 — Create the ALB Security Group

The ALB will receive HTTP traffic from the internet.

Open:

```text
EC2
→ Security Groups
→ Create security group
```

Configure:

```text
Security group name:
MetaPi-Lab17-ALB-SG

Description:
Security group for Lab 17 Application Load Balancer

VPC:
MetaPi-MultiAZ-VPC
```

### Inbound rule

Add:

```text
Type: HTTP
Port: 80
Source: Anywhere-IPv4
```

Which means:

```text
0.0.0.0/0
```

Description:

```text
HTTP from Internet to ALB
```

### Outbound

Keep the default:

```text
All traffic → 0.0.0.0/0
```

Create the Security Group.

---

# Step 4 — Create the Application Security Group

The application EC2 instances must **not** accept HTTP directly from the internet.

Create another Security Group:

```text
MetaPi-Lab17-App-SG
```

Description:

```text
Security group for Lab 17 private application EC2 instances
```

VPC:

```text
MetaPi-MultiAZ-VPC
```

### Inbound rule

Add:

```text
Type: HTTP
Port: 80
Source: Security Group
```

Select:

```text
MetaPi-Lab17-ALB-SG
```

Description:

```text
HTTP from Lab 17 ALB
```

### Do NOT add:

```text
HTTP from 0.0.0.0/0
SSH from 0.0.0.0/0
HTTPS from 0.0.0.0/0
```

The important security concept is:

```text
Internet
   ↓
ALB-SG
   ↓
App-SG
   ↓
Private EC2
```

### Checkpoint

Your application Security Group should have only:

```text
HTTP : 80
Source: MetaPi-Lab17-ALB-SG
```

---

# Step 5 — Create the Target Group

The Target Group tells the ALB where to send application traffic.

Open:

```text
EC2
→ Load Balancing
→ Target Groups
→ Create target group
```

Choose:

```text
Target type:
Instances
```

Configure:

```text
Target group name:
MetaPi-Lab17-TG

Protocol:
HTTP

Port:
80

VPC:
MetaPi-MultiAZ-VPC
```

For health checks:

```text
Health check protocol:
HTTP

Health check path:
/ 
```

Keep the remaining health check settings at their default values.

Create the Target Group.

### Important

**Do not manually register EC2 instances.**

The Auto Scaling Group will automatically register its instances with this Target Group.

---

# Step 6 — Create the Application Load Balancer

Open:

```text
EC2
→ Load Balancing
→ Load Balancers
→ Create Load Balancer
```

Select:

```text
Application Load Balancer
```

Configure:

```text
Load balancer name:
MetaPi-Lab17-ALB

Scheme:
Internet-facing

IP address type:
IPv4
```

---

## Network Mapping

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

> The ALB belongs in the **public subnets**.

---

## Security Group

Select:

```text
MetaPi-Lab17-ALB-SG
```

---

## Listener

Configure:

```text
Protocol:
HTTP

Port:
80

Default action:
Forward to MetaPi-Lab17-TG
```

Create the Load Balancer.

Wait until its state becomes:

```text
Active
```

### Checkpoint

Confirm:

```text
MetaPi-Lab17-ALB
        ↓
HTTP : 80
        ↓
MetaPi-Lab17-TG
```

---

# Step 7 — Create the Launch Template

The Launch Template defines how Auto Scaling should create new EC2 instances.

Open:

```text
EC2
→ Launch Templates
→ Create launch template
```

Configure:

```text
Launch template name:
MetaPi-Lab17-LT
```

Description:

```text
Lab 17 ALB Auto Scaling application template
```

---

## AMI

Select:

```text
Ubuntu Server LTS
```

Use the approved training AMI available in your AWS region.

---

## Instance Type

For this training lab:

```text
t3.micro
```

> Instance types can incur charges. Use the approved small instance type for your training environment.

---

## Key Pair

For this lab:

```text
Don't include in launch template
```

We do not need SSH access because the application will be accessed through the ALB.

---

## Network Settings

Do not select a specific subnet.

The Auto Scaling Group will decide which subnet/AZ to use.

---

## Security Group

Select:

```text
MetaPi-Lab17-App-SG
```

---

## Public IP

The application instances should remain private.

```text
No public IPv4 address required
```

---

## Storage

Use the default small root volume.

For this lab:

```text
8 GiB
gp3
Delete on termination: Yes
```

---

## User Data

Scroll to:

```text
Advanced details
→ User data
```

Paste:

```bash
#!/bin/bash

apt-get update -y
apt-get install -y apache2

cat > /var/www/html/index.html <<EOF
<h1>MetaPi ALB + Auto Scaling</h1>
<p>Server: $(hostname)</p>
EOF

systemctl enable --now apache2
```

This script:

1. Updates package information
    
2. Installs Apache
    
3. Creates a simple web page
    
4. Displays the server hostname
    
5. Starts Apache
    
6. Enables Apache at boot
    

The hostname is important because it will help us prove that the ALB is sending requests to different EC2 instances.

Create the Launch Template.

### Checkpoint

Confirm:

```text
MetaPi-Lab17-LT
Version: 1
```

---

# Step 8 — Create the Auto Scaling Group

Now we will use the Launch Template to create the application servers.

Open:

```text
EC2
→ Auto Scaling Groups
→ Create Auto Scaling group
```

---

## Step 8.1 — Group Name

Enter:

```text
MetaPi-Lab17-ASG
```

Select:

```text
Launch Template:
MetaPi-Lab17-LT
Version:
Default
```

Continue.

---

# Step 8.2 — Network

Select:

```text
VPC:
MetaPi-MultiAZ-VPC
```

For subnets, select **only the private subnets**:

```text
MetaPi-Private-A
MetaPi-Private-B
```

### Very Important

Do NOT select:

```text
MetaPi-Public-A
MetaPi-Public-B
```

The application EC2 instances must remain private.

---

# Step 8.3 — Load Balancing

Attach the existing Target Group:

```text
MetaPi-Lab17-TG
```

Enable:

```text
Elastic Load Balancing health checks
```

Keep the health check grace period around:

```text
300 seconds
```

This gives the instance enough time to boot and install Apache.

---

# Step 8.4 — Group Size

Configure:

```text
Desired capacity:
2

Minimum capacity:
2

Maximum capacity:
4
```

### What does this mean?

```text
Minimum = 2
```

The ASG should normally never go below two instances.

```text
Desired = 2
```

Start the lab with two instances.

```text
Maximum = 4
```

The ASG can scale up to four instances when required.

---

# Step 8.5 — Scaling Policy

For now, configure target tracking:

```text
Policy type:
Target tracking scaling policy

Metric:
Average CPU utilization

Target:
50%
```

Enable:

```text
Scale in
```

Use approximately:

```text
Instance warmup:
300 seconds
```

This tells Auto Scaling to maintain average CPU utilization around 50%.

Complete the ASG creation.

---

# Step 9 — Wait for Healthy Targets

After creating the ASG, go to:

```text
EC2
→ Auto Scaling Groups
→ MetaPi-Lab17-ASG
```

You should eventually see:

```text
Desired: 2
Min: 2
Max: 4
Instances: 2
```

Both instances should become healthy.

---

## Check Target Group

Open:

```text
EC2
→ Target Groups
→ MetaPi-Lab17-TG
→ Targets
```

Initially, targets may show:

```text
Initial
```

Wait for the health checks to complete.

Eventually you should see:

```text
Healthy: 2
Unhealthy: 0
```

### Important

The instances should have:

```text
Private IPv4 address
```

They should not require public IPv4 addresses.

### Checkpoint

Do not continue until:

```text
2 targets
Healthy
```

---

# Step 10 — Test the Application Through the ALB

Open:

```text
EC2
→ Load Balancers
→ MetaPi-Lab17-ALB
```

Copy its:

```text
DNS name
```

It will look similar to:

```text
MetaPi-Lab17-ALB-xxxxxxxx.eu-north-1.elb.amazonaws.com
```

Open:

```text
http://<ALB-DNS-NAME>
```

You should see:

```text
MetaPi ALB + Auto Scaling

Server: ip-10-20-xx-xx
```

Now refresh the page several times.

You may see:

```text
Server: ip-10-20-11-xx
```

and then:

```text
Server: ip-10-20-12-xx
```

### What did we prove?

The ALB is distributing requests between different backend EC2 instances.

```text
Request 1 → EC2-A
Request 2 → EC2-B
```

The exact order is controlled by the load balancer, so do not expect a guaranteed alternating pattern.

---

# Step 11 — Test Auto Scaling Self-Healing

Now we will intentionally terminate one instance.

This is a controlled failure test.

Open:

```text
EC2
→ Instances
```

Identify an instance belonging to:

```text
MetaPi-Lab17-ASG
```

Select one instance.

Choose:

```text
Instance state
→ Terminate instance
```

Confirm termination.

---

## Observe the Auto Scaling Group

Go to:

```text
EC2
→ Auto Scaling Groups
→ MetaPi-Lab17-ASG
→ Activity
```

You should see that Auto Scaling detects the terminated instance and launches a replacement.

The sequence should look similar to:

```text
Instance terminated
       ↓
ASG detects capacity/health problem
       ↓
Replacement instance launched
       ↓
Apache installs through NAT
       ↓
Instance becomes healthy
       ↓
Target registers with Target Group
```

Eventually you should return to:

```text
Desired: 2
Running: 2
Healthy: 2
```

### 🎯 What did we prove?

This is called **self-healing**.

If an instance fails, Auto Scaling can automatically replace it.

---

# Step 12 — Verify Target Tracking

Open:

```text
EC2
→ Auto Scaling Groups
→ MetaPi-Lab17-ASG
→ Automatic scaling
```

You should see a target tracking policy similar to:

```text
Target Tracking Policy

Metric:
Average CPU utilization

Target:
50%

Scale in:
Enabled
```

### What does 50% mean?

The ASG tries to maintain the average CPU utilization of the group around:

```text
50%
```

If workload increases, Auto Scaling can add instances.

If workload decreases, Auto Scaling can remove instances, while respecting:

```text
Minimum: 2
Maximum: 4
```

> Do not expect the ASG to immediately scale just because the policy exists. Scaling depends on the CloudWatch metric, evaluation periods, cooldown/warmup behavior, and actual workload.

---

# Step 13 — Understand the Architecture

At this point, the complete architecture is:

```text
                         Internet
                            |
                            v
                    Internet-facing ALB
                         HTTP :80
                            |
                     Target Group
                       /       \
                      /         \
                     v           v
              Private-A       Private-B
                  EC2             EC2
                    \             /
                     \           /
                      NAT Gateway
                           |
                           v
                    Internet Gateway
                           |
                           v
                        Internet
```

### Security flow

```text
Internet
   ↓
ALB Security Group
   ↓
HTTP :80
   ↓
Application Security Group
   ↓
Private EC2
```

The important rule is:

```text
Internet → ALB
ALB → Application EC2
```

Not:

```text
Internet → Application EC2
```

---

# Step 14 — Final Verification

Before cleanup, verify every item below.

## VPC

-  `MetaPi-MultiAZ-VPC` is being used
    
-  ALB uses Public-A and Public-B
    
-  ASG uses Private-A and Private-B
    

## NAT

-  `MetaPi-Lab17-NAT` is available
    
-  Private-A route table has NAT route
    
-  Private-B route table has NAT route
    

## Security

-  ALB-SG allows HTTP 80 from Internet
    
-  App-SG allows HTTP 80 only from ALB-SG
    
-  EC2 instances do not need public IPs
    

## Load Balancer

-  ALB is Internet-facing
    
-  Listener is HTTP :80
    
-  Listener forwards to `MetaPi-Lab17-TG`
    

## Target Group

-  Two targets are registered automatically
    
-  Two targets are Healthy
    
-  Health check path is `/`
    

## Auto Scaling

-  Desired = 2
    
-  Minimum = 2
    
-  Maximum = 4
    
-  ELB health checks enabled
    
-  Target tracking = 50% CPU
    

## Testing

-  ALB DNS opens the application
    
-  Different backend hostnames can be observed
    
-  Terminated instance was automatically replaced
    
-  Replacement became healthy
    

---

# Step 15 — Cleanup

⚠️ **Important: Cleanup is part of the lab.**

Several resources in this lab can incur charges, especially:

- NAT Gateway
    
- Elastic IP while allocated
    
- ALB
    
- EC2 instances
    

Only delete the resources created specifically for Lab 17.

---

## Cleanup Order

### 1. Delete the Auto Scaling Group

Go to:

```text
EC2
→ Auto Scaling Groups
→ MetaPi-Lab17-ASG
→ Delete
```

This terminates the ASG-managed instances.

---

### 2. Delete the Application Load Balancer

Go to:

```text
EC2
→ Load Balancers
→ MetaPi-Lab17-ALB
→ Delete
```

---

### 3. Delete the Target Group

Go to:

```text
EC2
→ Target Groups
→ MetaPi-Lab17-TG
→ Actions
→ Delete
```

---

### 4. Delete the Launch Template

Go to:

```text
EC2
→ Launch Templates
→ MetaPi-Lab17-LT
→ Actions
→ Delete
```

---

### 5. Delete the NAT Gateway

Go to:

```text
VPC
→ NAT Gateways
→ MetaPi-Lab17-NAT
→ Delete
```

Wait for the NAT Gateway to finish deleting.

---

### 6. Release the Elastic IP

Go to:

```text
VPC
→ Elastic IPs
```

Identify the EIP created specifically for:

```text
MetaPi-Lab17-NAT
```

Release it.

> ⚠️ Do not release Elastic IPs belonging to other labs.

---

### 7. Remove NAT Routes

Open the route table for:

```text
MetaPi-Private-A
```

Remove only:

```text
0.0.0.0/0 → MetaPi-Lab17-NAT
```

Repeat for:

```text
MetaPi-Private-B
```

Remove only the NAT route.

### Do NOT remove:

```text
10.20.0.0/16 → local
```

That route belongs to the VPC itself.

---

### 8. Delete Lab Security Groups

Delete:

```text
MetaPi-Lab17-App-SG
MetaPi-Lab17-ALB-SG
```

If AWS says a Security Group is still in use, check whether a Lab 17 resource is still running or attached before trying again.

---

# ⚠️ Resources You Should NOT Delete

Do not delete the shared VPC:

```text
MetaPi-MultiAZ-VPC
```

Do not delete its shared subnets:

```text
MetaPi-Public-A
MetaPi-Public-B
MetaPi-Private-A
MetaPi-Private-B
```

Also do not delete resources belonging to previous labs.

Always verify the **resource name, VPC, and purpose** before deleting anything.

---

# 🧠 What You Learned

In this lab, you built a production-style web tier using AWS managed services.

You learned how to:

```text
Public ALB
     ↓
Target Group
     ↓
Auto Scaling
     ↓
Private EC2
```

You also learned:

- Why load balancers are placed in public subnets
    
- Why application servers can remain private
    
- How Security Groups can reference other Security Groups
    
- How Target Groups perform health checks
    
- How Auto Scaling maintains desired capacity
    
- How Auto Scaling replaces failed instances
    
- How target tracking works
    
- Why Multi-AZ improves availability
    
- Why NAT Gateway is required for private instances that need outbound internet access
    
- Why production environments often use AZ-local NAT paths
    
- Why cleanup is important for AWS cost control
    

---

# 🎯 Final Architecture

```text
                     🌐 Internet
                          |
                          v
                 ┌─────────────────┐
                 │  Public ALB     │
                 │   HTTP : 80     │
                 └────────┬────────┘
                          |
                    Target Group
                     /         \
                    /           \
                   v             v
          ┌──────────────┐ ┌──────────────┐
          │ Private-A    │ │ Private-B    │
          │ EC2          │ │ EC2          │
          └──────┬───────┘ └──────┬───────┘
                 \                 /
                  \               /
                   v             v
                    NAT Gateway
                         |
                         v
                  Internet Gateway
                         |
                         v
                      Internet
```

# ✅ Lab Completed

You have successfully built an:

**Highly Available + Load Balanced + Auto Healing + Auto Scaling Web Architecture**

using:

```text
VPC
+
NAT Gateway
+
ALB
+
Target Group
+
Launch Template
+
Auto Scaling Group
+
Private EC2
+
Security Groups
```

---

## Next Lab

**Lab 18 — Create Amazon RDS MySQL and Connect from EC2**