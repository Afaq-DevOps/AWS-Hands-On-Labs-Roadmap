# Lab 15 — Configure an Application Load Balancer for EC2

> **Objective:** Deploy an internet-facing Application Load Balancer across two Availability Zones and distribute HTTP traffic between two EC2 web servers.

---

## 🏗️ Lab Architecture

```text
                              🌐 Internet
                                   │
                                   │ HTTP :80
                                   ▼
                      ┌─────────────────────────┐
                      │ Application Load        │
                      │ Balancer                │
                      │ MetaPi-Lab15-ALB        │
                      └────────────┬────────────┘
                                   │
                                   ▼
                      ┌─────────────────────────┐
                      │ Target Group            │
                      │ MetaPi-Lab15-TG         │
                      └────────────┬────────────┘
                                   │
                    ┌──────────────┴──────────────┐
                    │                             │
                    ▼                             ▼
          ┌──────────────────┐         ┌──────────────────┐
          │ Web Server A     │         │ Web Server B     │
          │ eu-north-1a      │         │ eu-north-1b      │
          │ Public-A         │         │ Public-B         │
          │ 🟢 Healthy       │         │ 🟢 Healthy       │
          └──────────────────┘         └──────────────────┘
```

### AWS Region

```text
eu-north-1 — Europe (Stockholm)
```

### VPC

```text
MetaPi-MultiAZ-VPC
10.20.0.0/16
```

---

# 🎯 Lab Objectives

By completing this lab, you will:

- Create Security Groups for the ALB and EC2 web servers.
    
- Deploy two EC2 web servers in different Availability Zones.
    
- Create an EC2 Target Group.
    
- Configure an Application Load Balancer.
    
- Configure an HTTP listener on port `80`.
    
- Configure ALB health checks.
    
- Verify both EC2 instances become healthy.
    
- Test traffic through the ALB DNS name.
    
- Simulate an instance failure and observe ALB failover.
    

---

# 📋 Resources Used

|Resource|Name / Configuration|
|---|---|
|VPC|`MetaPi-MultiAZ-VPC`|
|Public Subnet A|`MetaPi-Public-A`|
|Public Subnet B|`MetaPi-Public-B`|
|Availability Zone A|`eu-north-1a`|
|Availability Zone B|`eu-north-1b`|
|ALB Security Group|`MetaPi-Lab15-ALB-SG`|
|Web Security Group|`MetaPi-Lab15-Web-SG`|
|Target Group|`MetaPi-Lab15-TG`|
|Load Balancer|`MetaPi-Lab15-ALB`|
|Protocol|HTTP|
|Port|`80`|

---

# 🔐 Step 1 — Use the Multi-AZ VPC

For this lab, use the existing Multi-AZ VPC and its public subnets.

### VPC

```text
MetaPi-MultiAZ-VPC
CIDR: 10.20.0.0/16
```

### Public Subnets

```text
MetaPi-Public-A
eu-north-1a
10.20.1.0/24
```

```text
MetaPi-Public-B
eu-north-1b
10.20.2.0/24
```

### ⚠️ Important

Make sure you are working in:

```text
eu-north-1
```

Do **not** accidentally select the default VPC.

---

# 🛡️ Step 2 — Create the ALB Security Group

Go to:

```text
EC2
→ Security Groups
→ Create security group
```

Create:

```text
MetaPi-Lab15-ALB-SG
```

### VPC

Select:

```text
MetaPi-MultiAZ-VPC
```

### Inbound Rules

|Type|Port|Source|
|---|--:|---|
|HTTP|80|`0.0.0.0/0`|

This allows users on the internet to access the ALB over HTTP.

### Outbound Rules

Keep the default:

```text
All traffic → 0.0.0.0/0
```

Click:

```text
Create security group
```

---

# 🛡️ Step 3 — Create the Web Server Security Group

Create another Security Group:

```text
MetaPi-Lab15-Web-SG
```

Use:

```text
VPC:
MetaPi-MultiAZ-VPC
```

### Inbound Rules

Add:

|Type|Port|Source|
|---|--:|---|
|HTTP|80|`MetaPi-Lab15-ALB-SG`|
|SSH|22|My IP|

### Why?

The architecture should be:

```text
Internet
   │
   ▼
ALB Security Group
   │
   │ HTTP :80
   ▼
Web Server Security Group
   │
   ├── Web Server A
   └── Web Server B
```

This allows the ALB to communicate with the web servers without exposing HTTP directly to the entire internet.

---

# 🖥️ Step 4 — Launch Web Server A

Go to:

```text
EC2
→ Instances
→ Launch instance
```

Configure:

|Setting|Value|
|---|---|
|Name|`MetaPi-Lab15-Web-A`|
|OS|Ubuntu|
|VPC|`MetaPi-MultiAZ-VPC`|
|Subnet|`MetaPi-Public-A`|
|Availability Zone|`eu-north-1a`|
|Auto-assign Public IP|Enabled|
|Security Group|`MetaPi-Lab15-Web-SG`|

### User Data

Under **Advanced details → User data**, add:

```bash
#!/bin/bash

apt-get update -y
apt-get install -y apache2

echo "<h1>MetaPi Lab 15 - Web Server A</h1><p>Availability Zone: eu-north-1a</p>" > /var/www/html/index.html

systemctl enable --now apache2
```

Launch the instance.

### ✅ Expected Result

You should have:

```text
MetaPi-Lab15-Web-A
eu-north-1a
MetaPi-Public-A
```

---

# 🖥️ Step 5 — Launch Web Server B

Launch another EC2 instance.

Configure:

|Setting|Value|
|---|---|
|Name|`MetaPi-Lab15-Web-B`|
|OS|Ubuntu|
|VPC|`MetaPi-MultiAZ-VPC`|
|Subnet|`MetaPi-Public-B`|
|Availability Zone|`eu-north-1b`|
|Auto-assign Public IP|Enabled|
|Security Group|`MetaPi-Lab15-Web-SG`|

### User Data

```bash
#!/bin/bash

apt-get update -y
apt-get install -y apache2

echo "<h1>MetaPi Lab 15 - Web Server B</h1><p>Availability Zone: eu-north-1b</p>" > /var/www/html/index.html

systemctl enable --now apache2
```

Launch the instance.

### ✅ Expected Result

You should have:

```text
MetaPi-Lab15-Web-B
eu-north-1b
MetaPi-Public-B
```

---

# 🎯 Step 6 — Create the Target Group

Go to:

```text
EC2
→ Target Groups
→ Create target group
```

### Target Type

Select:

```text
Instances
```

### Basic Configuration

```text
Target group name:
MetaPi-Lab15-TG

Protocol:
HTTP

Port:
80

VPC:
MetaPi-MultiAZ-VPC
```

---

## 🩺 Health Check

Configure:

```text
Health check protocol:
HTTP

Health check path:
/
```

Keep the remaining settings at their defaults unless your lab requires otherwise.

---

## 🖥️ Register Targets

Select:

```text
MetaPi-Lab15-Web-A
MetaPi-Lab15-Web-B
```

Make sure both use:

```text
Port: 80
```

Click:

```text
Include as pending below
```

Then:

```text
Create target group
```

---

# 🔎 Step 7 — Verify the Target Group

Open:

```text
EC2
→ Target Groups
→ MetaPi-Lab15-TG
→ Targets
```

Initially, targets may show:

```text
Initial
```

Wait for the health checks to complete.

### Expected Result

```text
MetaPi-Lab15-Web-A → 🟢 Healthy
MetaPi-Lab15-Web-B → 🟢 Healthy
```

You want:

```text
2 Healthy
0 Unhealthy
```

### ⚠️ If a Target Is Unhealthy

Check:

- EC2 instance is running.
    
- Apache is running.
    
- Apache is listening on port `80`.
    
- Web Security Group allows HTTP `80` from `MetaPi-Lab15-ALB-SG`.
    
- Health check path is `/`.
    
- Target is registered on port `80`.
    
- Network ACL allows the required traffic.
    

---

# ⚖️ Step 8 — Create the Application Load Balancer

Go to:

```text
EC2
→ Load Balancers
→ Create load balancer
```

Select:

```text
Application Load Balancer
```

Click:

```text
Create
```

---

# ⚙️ Step 9 — Configure Basic Settings

### Load Balancer Name

```text
MetaPi-Lab15-ALB
```

### Scheme

```text
Internet-facing
```

### IP Address Type

```text
IPv4
```

---

# 🌐 Step 10 — Configure Network Mapping

### VPC

Select:

```text
MetaPi-MultiAZ-VPC
10.20.0.0/16
```

### Availability Zones and Subnets

Select:

```text
eu-north-1a
→ MetaPi-Public-A
→ 10.20.1.0/24
```

and:

```text
eu-north-1b
→ MetaPi-Public-B
→ 10.20.2.0/24
```

### ⚠️ Important

You must select **both Availability Zones**.

The final mapping should be:

```text
eu-north-1a
      ↓
MetaPi-Public-A

eu-north-1b
      ↓
MetaPi-Public-B
```

This provides the Multi-AZ architecture.

---

# 🔐 Step 11 — Select the ALB Security Group

Under **Security Groups**, select:

```text
MetaPi-Lab15-ALB-SG
```

Verify that the Security Group allows:

```text
HTTP :80
Source: 0.0.0.0/0
```

---

# 🎧 Step 12 — Configure the Listener

Under **Listeners and routing**, configure:

```text
Protocol:
HTTP

Port:
80
```

### Default Action

Select:

```text
Forward to target groups
```

Target Group:

```text
MetaPi-Lab15-TG
```

Weight:

```text
100%
```

The final listener configuration should be:

```text
HTTP :80
      ↓
MetaPi-Lab15-TG
      ↓
Web-A + Web-B
```

---

# 🛡️ Step 13 — Service Integrations

For this basic ALB lab, do not configure additional services.

Leave:

```text
CloudFront + WAF → Not configured
AWS WAF → Not configured
Global Accelerator → Not configured
```

These are not required for this lab.

---

# 🔍 Step 14 — Review the Configuration

Before creating the ALB, verify:

```text
Name:
MetaPi-Lab15-ALB

Scheme:
Internet-facing

IP:
IPv4

VPC:
MetaPi-MultiAZ-VPC

Subnets:
MetaPi-Public-A
MetaPi-Public-B

Security Group:
MetaPi-Lab15-ALB-SG

Listener:
HTTP :80

Target Group:
MetaPi-Lab15-TG

Weight:
100%
```

If everything is correct, click:

```text
Create load balancer
```

---

# ⏳ Step 15 — Wait for the ALB to Become Active

Go to:

```text
EC2
→ Load Balancers
→ MetaPi-Lab15-ALB
```

Wait until:

```text
State:
Active
```

The ALB may initially show:

```text
Provisioning
```

This is normal.

---

# 🌍 Step 16 — Get the ALB DNS Name

Open:

```text
MetaPi-Lab15-ALB
```

Find:

```text
DNS name
```

It will look similar to:

```text
MetaPi-Lab15-ALB-xxxxxxxx.eu-north-1.elb.amazonaws.com
```

⚠️ Your DNS name will be different.

---

# 🧪 Step 17 — Test the ALB

Open your browser and enter:

```text
http://<ALB-DNS-NAME>
```

Example:

```text
http://MetaPi-Lab15-ALB-xxxxxxxx.eu-north-1.elb.amazonaws.com
```

Because the listener is configured for HTTP port `80`, use:

```text
http://
```

not:

```text
https://
```

---

# ✅ Step 18 — Verify Web Server A

The ALB should be able to route traffic to Web Server A.

Expected response:

```text
MetaPi Lab 15 - Web Server A

Availability Zone:
eu-north-1a
```

This proves:

```text
Internet
   ↓
ALB
   ↓
Target Group
   ↓
Web Server A
```

---

# ✅ Step 19 — Verify Web Server B

Refresh the ALB DNS URL or make another request.

You should also be able to receive:

```text
MetaPi Lab 15 - Web Server B

Availability Zone:
eu-north-1b
```

This proves:

```text
Internet
   ↓
ALB
   ↓
Target Group
   ↓
Web Server B
```

### ⚠️ Important

Do not expect every refresh to alternate exactly:

```text
A → B → A → B
```

Browser connection reuse and ALB behavior can result in the same server responding multiple times.

The important thing is that **both healthy targets can successfully serve traffic through the ALB**.

---

# 🧪 Step 20 — Simulate an Instance Failure

This step demonstrates the benefit of health checks.

### Stop Apache on Web Server A

Connect to Web Server A and run:

```bash
sudo systemctl stop apache2
```

Now return to:

```text
EC2
→ Target Groups
→ MetaPi-Lab15-TG
→ Targets
```

Wait for Web Server A to become:

```text
🔴 Unhealthy
```

while Web Server B remains:

```text
🟢 Healthy
```

---

# 🔄 Step 21 — Test Failover

Open the ALB DNS name again:

```text
http://<ALB-DNS-NAME>
```

Traffic should continue to the healthy Web Server B.

Architecture:

```text
                 Internet
                    │
                    ▼
             Application ALB
                    │
                    ▼
             MetaPi-Lab15-TG
                /        \
               /          \
              ▼            ▼
           Web-A         Web-B
           🔴             🟢
        Unhealthy        Healthy
                            ▲
                            │
                         Traffic
```

This demonstrates that the ALB uses target health to determine where traffic should be sent.

---

# 🔄 Step 22 — Restore Web Server A

Start Apache again:

```bash
sudo systemctl start apache2
```

Verify:

```bash
sudo systemctl status apache2
```

Wait for the ALB health check to run again.

Eventually:

```text
Web Server A → 🟢 Healthy
Web Server B → 🟢 Healthy
```

Expected:

```text
2 Healthy
0 Unhealthy
```

---

# 🔎 Step 23 — Final Verification

Verify all of the following:

### Network

```text
☑ MetaPi-MultiAZ-VPC
☑ MetaPi-Public-A
☑ MetaPi-Public-B
☑ eu-north-1a
☑ eu-north-1b
```

### Security

```text
☑ MetaPi-Lab15-ALB-SG
☑ MetaPi-Lab15-Web-SG
☑ ALB allows HTTP :80 from internet
☑ Web servers allow HTTP :80 from ALB SG
☑ SSH :22 allowed from My IP
```

### Load Balancer

```text
☑ MetaPi-Lab15-ALB
☑ Internet-facing
☑ IPv4
☑ HTTP :80
☑ Two Availability Zones
```

### Target Group

```text
☑ MetaPi-Lab15-TG
☑ Web-A registered
☑ Web-B registered
☑ Health check path = /
☑ Web-A = Healthy
☑ Web-B = Healthy
```

### Testing

```text
☑ ALB DNS works
☑ Web-A response verified
☑ Web-B response verified
☑ Failover tested
```

---

# 📸 Step 24 — Recommended Evidence Screenshots

For your lab documentation, capture these screenshots:

### 1. ALB Configuration

Show:

```text
MetaPi-Lab15-ALB
Internet-facing
IPv4
```

### 2. Network Mapping

Show:

```text
eu-north-1a → MetaPi-Public-A
eu-north-1b → MetaPi-Public-B
```

### 3. Target Group Health

Show:

```text
MetaPi-Lab15-TG

Web-A → 🟢 Healthy
Web-B → 🟢 Healthy
```

### 4. Web Server A

Show the browser response:

```text
MetaPi Lab 15 - Web Server A
eu-north-1a
```

### 5. Web Server B

Show the browser response:

```text
MetaPi Lab 15 - Web Server B
eu-north-1b
```

### 6. Failover Test

Show:

```text
Web-A → Unhealthy
Web-B → Healthy
```

and the application still accessible through the ALB.

---

# 🧠 What Did We Build?

Before this lab, users would need to access individual EC2 servers directly:

```text
User
 ├──→ Web Server A
 └──→ Web Server B
```

After this lab:

```text
                    User
                     │
                     ▼
              Application ALB
                     │
                     ▼
               Target Group
                 /       \
                ▼         ▼
             Web-A      Web-B
             AZ-1a      AZ-1b
```

The ALB now provides a **single entry point** and distributes traffic across healthy EC2 instances.

---

# 🎓 Key Concepts

### Application Load Balancer

An ALB distributes HTTP/HTTPS application traffic across registered targets.

### Target Group

A Target Group contains the backend targets that receive traffic from the ALB.

### Health Check

Health checks allow the ALB to determine whether a target is healthy.

### Multi-AZ

Deploying the web servers across two Availability Zones improves availability and resilience.

### Security Groups

Separate Security Groups allow us to control:

```text
Internet → ALB
ALB → EC2
```

independently.

---

# 🏆 Lab Completion Criteria

Lab 15 is successfully completed when:

```text
✅ ALB is Active

✅ ALB uses MetaPi-MultiAZ-VPC

✅ ALB spans two Availability Zones

✅ HTTP listener is configured on port 80

✅ MetaPi-Lab15-TG is attached

✅ Web-A is Healthy

✅ Web-B is Healthy

✅ ALB DNS successfully opens

✅ Web-A response verified

✅ Web-B response verified

✅ Failover behavior tested
```

---

# 🧹 Step 25 — Cleanup

If you are continuing with the next labs and these resources are required, **keep them running**.

If the lab resources are no longer required, clean them up to avoid unnecessary AWS charges.

Review and remove:

```text
MetaPi-Lab15-ALB
MetaPi-Lab15-TG
MetaPi-Lab15-Web-A
MetaPi-Lab15-Web-B
MetaPi-Lab15-ALB-SG
MetaPi-Lab15-Web-SG
```

### ⚠️ Important

Do **not** delete shared infrastructure if future labs depend on it:

```text
MetaPi-MultiAZ-VPC
MetaPi-Public-A
MetaPi-Public-B
Route Tables
Internet Gateway
NAT Gateway
```

Always check the next lab before deleting shared resources.

---

# 🎉 Lab 15 Completed

You have successfully deployed an **Internet-facing Application Load Balancer** across two Availability Zones and connected it to two EC2 web servers through a Target Group.

### Final Architecture

```text
                         🌐 Internet
                              │
                              │ HTTP :80
                              ▼
                   ┌──────────────────────┐
                   │ MetaPi-Lab15-ALB     │
                   │ Application LB       │
                   └──────────┬───────────┘
                              │
                              ▼
                   ┌──────────────────────┐
                   │ MetaPi-Lab15-TG      │
                   │ Health Checks: /     │
                   └──────────┬───────────┘
                              │
                   ┌──────────┴──────────┐
                   │                     │
                   ▼                     ▼
          ┌─────────────────┐   ┌─────────────────┐
          │ Web Server A    │   │ Web Server B    │
          │ eu-north-1a     │   │ eu-north-1b     │
          │ Public-A        │   │ Public-B        │
          │ 🟢 Healthy      │   │ 🟢 Healthy      │
          └─────────────────┘   └─────────────────┘
```

> **🏆 Result:** Multi-AZ Application Load Balancer successfully deployed, health checks verified, traffic routing tested, and failover behavior demonstrated.

---

## ➡️ Next Lab

**Lab 16 — Create an EC2 Auto Scaling Group**