#  Lab 015 — Configure an Application Load Balancer for EC2

> **MetaPi PSEB Training Program | AWS Hands-On Labs Roadmap**

---

## 📌 Overview

In this lab, you will deploy an **Application Load Balancer (ALB)** across two Availability Zones and use it to distribute HTTP traffic between two EC2 web servers.

The lab builds on the **Multi-AZ VPC architecture created in Lab 014**.

By completing this lab, you will understand how a load balancer receives requests from users, checks the health of backend servers, and forwards traffic only to healthy targets.

---

## 🎯 Learning Objectives

By completing this hands-on lab, you will learn how to:

* Create an Application Load Balancer Security Group.
* Create a Web Server Security Group.
* Understand Security Group to Security Group communication.
* Launch EC2 web servers in two Availability Zones.
* Configure Apache automatically using EC2 User Data.
* Create an EC2 Target Group.
* Register EC2 instances as targets.
* Configure ALB listeners.
* Configure ALB health checks.
* Verify target health.
* Test traffic through the ALB DNS name.
* Simulate a backend server failure.
* Understand how an ALB routes traffic when one target becomes unhealthy.
* Understand basic high availability using multiple Availability Zones.

---

# 🏗️ Architecture

```text
                           🌐 Internet
                                |
                                | HTTP :80
                                v
                    ┌─────────────────────────┐
                    │  Application Load      │
                    │  Balancer              │
                    │  MetaPi-Lab15-ALB      │
                    └────────────┬────────────┘
                                 |
                         HTTP :80 |
                                 v
                    ┌─────────────────────────┐
                    │      Target Group       │
                    │   MetaPi-Lab15-TG       │
                    └────────────┬────────────┘
                                 |
                  ┌──────────────┴──────────────┐
                  |                             |
                  v                             v
        ┌──────────────────┐          ┌──────────────────┐
        │   Web Server A   │          │   Web Server B   │
        │   AZ: eu-north-1a│          │   AZ: eu-north-1b│
        │   Public-A       │          │   Public-B       │
        │   Apache :80     │          │   Apache :80     │
        │   🟢 Healthy     │          │   🟢 Healthy     │
        └──────────────────┘          └──────────────────┘
```

### Request Flow

```text
Client
  ↓
ALB :80
  ↓
Target Group
  ↓
Healthy EC2 Target
  ↓
Apache Web Server
  ↓
HTTP Response
```

---

# 🌍 Lab Environment

| Component           | Configuration                     |
| ------------------- | --------------------------------- |
| AWS Region          | `eu-north-1` — Europe (Stockholm) |
| VPC                 | `MetaPi-MultiAZ-VPC`              |
| VPC CIDR            | `10.20.0.0/16`                    |
| Public Subnet A     | `MetaPi-Public-A`                 |
| Public Subnet B     | `MetaPi-Public-B`                 |
| Availability Zone A | `eu-north-1a`                     |
| Availability Zone B | `eu-north-1b`                     |
| Load Balancer       | `MetaPi-Lab15-ALB`                |
| Target Group        | `MetaPi-Lab15-TG`                 |
| Web Server A        | `MetaPi-Lab15-Web-A`              |
| Web Server B        | `MetaPi-Lab15-Web-B`              |

> **Note:** Availability Zone names can map differently between AWS accounts. Always select the two Availability Zones associated with the subnets you created in your Multi-AZ VPC.

---

# 📋 Prerequisites

Before starting this lab, make sure you have completed **Lab 014 — Build a Multi-AZ VPC Architecture**.

You should already have:

* An AWS account.
* Access to the AWS Management Console.
* `MetaPi-MultiAZ-VPC`.
* `MetaPi-Public-A`.
* `MetaPi-Public-B`.
* Internet Gateway configured for the public subnets.
* Public route table configured.
* An EC2 key pair.
* Your current public IP address.

---

# 🔐 Security Design

This lab uses two Security Groups.

```text
Internet
   |
   | HTTP :80
   v
ALB Security Group
MetaPi-Lab15-ALB-SG
   |
   | HTTP :80
   v
Web Server Security Group
MetaPi-Lab15-Web-SG
   |
   +---- Web Server A
   |
   +---- Web Server B
```

The important security concept is:

> **The EC2 web servers accept HTTP traffic from the ALB Security Group instead of allowing HTTP directly from the entire internet.**

SSH access is allowed only from your own IP address.

---

# 🛠️ Step 1 — Verify the Multi-AZ VPC

Before creating the ALB, verify that your VPC from Lab 014 is available.

### Navigate to:

**AWS Console → VPC → Your VPCs**

Find:

```text
MetaPi-MultiAZ-VPC
```

Verify:

```text
CIDR: 10.20.0.0/16
State: Available
```

---

### Verify Public Subnet A

Go to:

**VPC → Subnets**

Find:

```text
MetaPi-Public-A
```

Verify that it belongs to:

```text
MetaPi-MultiAZ-VPC
```

and is associated with one Availability Zone.

---

### Verify Public Subnet B

Find:

```text
MetaPi-Public-B
```

Verify that it belongs to:

```text
MetaPi-MultiAZ-VPC
```

and is located in a **different Availability Zone** from Public-A.

### Expected result

```text
MetaPi-Public-A → AZ-A
MetaPi-Public-B → AZ-B
```

---

# 🔐 Step 2 — Create the ALB Security Group

Navigate to:

**AWS Console → EC2 → Security Groups**

Click:

**Create security group**

### Security Group Configuration

**Security group name:**

```text
MetaPi-Lab15-ALB-SG
```

**Description:**

```text
Security group for Lab 15 Application Load Balancer
```

**VPC:**

```text
MetaPi-MultiAZ-VPC
```

---

## Inbound Rules

Add:

| Type | Protocol | Port | Source      |
| ---- | -------- | ---: | ----------- |
| HTTP | TCP      |   80 | `0.0.0.0/0` |

This allows users on the internet to send HTTP requests to the ALB.

---

## Outbound Rules

Keep the default rule:

```text
All traffic
0.0.0.0/0
```

Click:

**Create security group**

---

# 🔐 Step 3 — Create the Web Server Security Group

Create another Security Group.

Navigate to:

**EC2 → Security Groups → Create security group**

### Configuration

**Security group name:**

```text
MetaPi-Lab15-Web-SG
```

**Description:**

```text
Security group for Lab 15 EC2 web servers
```

**VPC:**

```text
MetaPi-MultiAZ-VPC
```

---

## Inbound Rule 1 — HTTP

Add:

| Type | Protocol | Port | Source                |
| ---- | -------- | ---: | --------------------- |
| HTTP | TCP      |   80 | `MetaPi-Lab15-ALB-SG` |

When selecting the source, choose the **Security Group** option and select:

```text
MetaPi-Lab15-ALB-SG
```

---

## Inbound Rule 2 — SSH

Add:

| Type | Protocol | Port | Source |
| ---- | -------- | ---: | ------ |
| SSH  | TCP      |   22 | My IP  |

Do **not** use:

```text
0.0.0.0/0
```

for SSH in this lab.

Click:

**Create security group**

---

# 💡 Why Two Security Groups?

The architecture now provides a simple security boundary:

```text
Internet
   |
   | HTTP :80
   v
ALB
   |
   | HTTP :80
   v
EC2 Web Servers
```

The EC2 instances do not need to accept HTTP traffic directly from the internet.

They only accept HTTP traffic coming from:

```text
MetaPi-Lab15-ALB-SG
```

This is a more controlled security design.

---

# 🖥️ Step 4 — Launch Web Server A

Navigate to:

**AWS Console → EC2 → Instances → Launch instance**

### Name

```text
MetaPi-Lab15-Web-A
```

### AMI

Choose:

```text
Ubuntu Server
```

Use the Ubuntu version available in your selected AWS Region.

### Instance Type

For this training lab, use a small instance type suitable for your AWS account/free-tier eligibility.

For example:

```text
t3.micro
```

> Always verify the current AWS pricing/free-tier eligibility for your account before launching resources.

---

## Key Pair

Select your existing EC2 key pair.

Example:

```text
Your-Key-Pair
```

---

## Network Settings

Click:

**Edit**

Configure:

**VPC:**

```text
MetaPi-MultiAZ-VPC
```

**Subnet:**

```text
MetaPi-Public-A
```

**Auto-assign Public IP:**

```text
Enable
```

**Security Group:**

```text
MetaPi-Lab15-Web-SG
```

---

# ⚙️ Step 5 — Configure User Data for Web Server A

Scroll to:

**Advanced details → User data**

Paste:

```bash
#!/bin/bash

apt-get update -y
apt-get install -y apache2

echo "<h1>MetaPi Web Server A</h1><p>Availability Zone A</p>" > /var/www/html/index.html

systemctl enable --now apache2
```

Then click:

**Launch instance**

---

# ⏳ Step 6 — Wait for Web Server A

Wait until the instance shows:

```text
Instance state: Running
```

and:

```text
Status checks: 2/2 checks passed
```

The User Data script should install Apache automatically.

---

# 🖥️ Step 7 — Launch Web Server B

Repeat the EC2 launch process.

### Name

```text
MetaPi-Lab15-Web-B
```

### VPC

```text
MetaPi-MultiAZ-VPC
```

### Subnet

```text
MetaPi-Public-B
```

### Public IP

```text
Enabled
```

### Security Group

```text
MetaPi-Lab15-Web-SG
```

---

## User Data for Web Server B

Use:

```bash
#!/bin/bash

apt-get update -y
apt-get install -y apache2

echo "<h1>MetaPi Web Server B</h1><p>Availability Zone B</p>" > /var/www/html/index.html

systemctl enable --now apache2
```

Click:

**Launch instance**

---

# 🔎 Step 8 — Verify Both Web Servers

Wait for both instances to become:

```text
Running
```

and:

```text
2/2 status checks passed
```

You should have:

```text
MetaPi-Lab15-Web-A
        ↓
MetaPi-Public-A
        ↓
AZ-A
```

and:

```text
MetaPi-Lab15-Web-B
        ↓
MetaPi-Public-B
        ↓
AZ-B
```

---

# 🌐 Step 9 — Test Apache Directly

For each instance:

**EC2 → Instances → Select instance**

Copy the:

```text
Public IPv4 address
```

Open:

```text
http://<PUBLIC-IP>
```

### Web Server A should display:

```text
MetaPi Web Server A
Availability Zone A
```

### Web Server B should display:

```text
MetaPi Web Server B
Availability Zone B
```

If these pages do not load, **fix the EC2 web servers before continuing**.

---

# 🎯 Step 10 — Create the Target Group

Navigate to:

**EC2 → Target Groups**

Click:

**Create target group**

---

## Target Group Configuration

### Target type

Select:

```text
Instances
```

### Target group name

```text
MetaPi-Lab15-TG
```

### Protocol

```text
HTTP
```

### Port

```text
80
```

### VPC

```text
MetaPi-MultiAZ-VPC
```

---

# ❤️ Step 11 — Configure Health Checks

Configure:

**Health check protocol:**

```text
HTTP
```

**Health check path:**

```text
/
```

The ALB will periodically request:

```text
http://<target>/ 
```

and determine whether the target is healthy.

Keep the default healthy/unhealthy thresholds unless your training instructions require different values.

Click:

**Next**

---

# 🖥️ Step 12 — Register EC2 Targets

On the **Register targets** page, select:

```text
MetaPi-Lab15-Web-A
MetaPi-Lab15-Web-B
```

Make sure the port is:

```text
80
```

Click:

**Include as pending below**

Then create the target group.

---

# 🔎 Step 13 — Verify Target Group

Open:

**EC2 → Target Groups → MetaPi-Lab15-TG**

Go to:

**Targets**

Initially, the targets may show:

```text
Initial
```

Wait a few moments while AWS performs health checks.

Eventually, both should become:

```text
🟢 Healthy
```

Expected:

```text
MetaPi-Lab15-Web-A → Healthy
MetaPi-Lab15-Web-B → Healthy
```

---

# ⚖️ Step 14 — Create the Application Load Balancer

Navigate to:

**EC2 → Load Balancers**

Click:

**Create Load Balancer**

Select:

```text
Application Load Balancer
```

Click:

**Create**

---

# ⚙️ Step 15 — Configure the ALB

### Load Balancer Name

```text
MetaPi-Lab15-ALB
```

### Scheme

Select:

```text
Internet-facing
```

### IP Address Type

Select:

```text
IPv4
```

---

# 🌐 Step 16 — Configure Network Mapping

For VPC select:

```text
MetaPi-MultiAZ-VPC
```

Select the two Availability Zones/subnets:

```text
MetaPi-Public-A
MetaPi-Public-B
```

The ALB should therefore span two Availability Zones.

---

# 🔐 Step 17 — Attach the ALB Security Group

Under Security Groups, select:

```text
MetaPi-Lab15-ALB-SG
```

Make sure the ALB is **not** using the EC2 web server Security Group.

---

# 👂 Step 18 — Configure the Listener

Configure the listener:

```text
Protocol: HTTP
Port: 80
```

For the default action choose:

```text
Forward to
MetaPi-Lab15-TG
```

The traffic flow is now:

```text
HTTP :80
   ↓
MetaPi-Lab15-ALB
   ↓
MetaPi-Lab15-TG
   ↓
EC2 Target
```

Review the configuration and click:

**Create load balancer**

---

# ⏳ Step 19 — Wait for the ALB

Open:

**EC2 → Load Balancers**

Select:

```text
MetaPi-Lab15-ALB
```

Wait until the ALB state becomes:

```text
Active
```

---

# 🌍 Step 20 — Copy the ALB DNS Name

Inside the ALB details, find:

```text
DNS name
```

It will look similar to:

```text
MetaPi-Lab15-ALB-xxxxxxxx.eu-north-1.elb.amazonaws.com
```

Copy the DNS name.

---

# 🧪 Step 21 — Test the Application Through the ALB

Open your browser and visit:

```text
http://<ALB-DNS-NAME>
```

You should receive a response from one of the EC2 servers.

For example:

```text
MetaPi Web Server A
Availability Zone A
```

or:

```text
MetaPi Web Server B
Availability Zone B
```

---

# 🔄 Step 22 — Test Load Balancing

Refresh the ALB URL multiple times.

The ALB distributes requests across the healthy targets according to its configured routing behavior.

You may see:

```text
Request
   ↓
ALB
   ↓
Web Server A
```

and another request may be sent to:

```text
Request
   ↓
ALB
   ↓
Web Server B
```

Because both pages contain different server identifiers, the response helps you visually verify that requests can reach different backend instances.

> **Important:** Do not assume every browser refresh must alternate A → B → A → B. Load balancing does not guarantee strict alternation.

---

# 💥 Step 23 — Simulate a Backend Failure

Now we will intentionally make one backend unhealthy.

Connect to:

```text
MetaPi-Lab15-Web-A
```

using SSH.

Run:

```bash
sudo systemctl stop apache2
```

Verify:

```bash
sudo systemctl status apache2
```

Apache should now be stopped.

---

# ❤️ Step 24 — Wait for the Health Check

Go to:

**EC2 → Target Groups → MetaPi-Lab15-TG → Targets**

Initially Web Server A may still show:

```text
Healthy
```

Wait for the ALB health check to detect the failure.

Eventually:

```text
MetaPi-Lab15-Web-A → Unhealthy
MetaPi-Lab15-Web-B → Healthy
```

---

# 🧪 Step 25 — Test the ALB During Failure

Open:

```text
http://<ALB-DNS-NAME>
```

Refresh the page.

Traffic should continue to the healthy backend:

```text
Internet
   ↓
ALB
   ↓
Target Group
   ↓
Web Server B
   ↓
HTTP Response
```

The ALB avoids routing normal traffic to the unhealthy target.

---

# 🔧 Step 26 — Restore Web Server A

SSH into Web Server A again.

Run:

```bash
sudo systemctl start apache2
```

Verify:

```bash
sudo systemctl status apache2
```

Apache should show:

```text
active (running)
```

---

# ❤️ Step 27 — Verify Target Recovery

Return to:

**EC2 → Target Groups → MetaPi-Lab15-TG → Targets**

Wait for Web Server A to become:

```text
🟢 Healthy
```

Expected final state:

```text
MetaPi-Lab15-Web-A → Healthy
MetaPi-Lab15-Web-B → Healthy
```

---

# 🧠 What Happened During the Failure Test?

Before failure:

```text
                    ALB
                   /   \
                  /     \
             Healthy   Healthy
                |         |
             Server A   Server B
```

After stopping Apache on Server A:

```text
                    ALB
                   /   \
                  /     \
            Unhealthy   Healthy
                X          |
                         Server B
```

The ALB health check detects that Server A is not responding correctly.

Traffic can therefore continue through the healthy target.

---

# 🔍 Troubleshooting

## ❌ Target is Unhealthy

Check the following:

### 1. Apache is running

```bash
sudo systemctl status apache2
```

If stopped:

```bash
sudo systemctl start apache2
```

---

### 2. Apache is listening on port 80

Run:

```bash
sudo ss -tlnp | grep :80
```

You should see Apache listening on port 80.

---

### 3. Web Server Security Group

Verify:

```text
HTTP :80
Source: MetaPi-Lab15-ALB-SG
```

---

### 4. Target Group Port

Verify:

```text
HTTP :80
```

---

### 5. Health Check Path

Verify:

```text
/
```

---

### 6. Apache index page exists

Run:

```bash
ls -l /var/www/html/
```

You should see:

```text
index.html
```

---

## ❌ ALB DNS Name Does Not Open

Check:

* ALB state is `Active`.
* ALB is internet-facing.
* ALB is attached to both public subnets.
* ALB Security Group allows HTTP `80`.
* Public subnets have a route to the Internet Gateway.
* At least one target is healthy.

---

## ❌ Direct EC2 Website Does Not Open

Check:

* EC2 instance is running.
* Public IPv4 address exists.
* Apache is installed.
* Apache is running.
* Security Group allows HTTP `80`.
* The instance is in the correct public subnet.
* The subnet has internet routing.

---

# 📊 Final Verification Checklist

Before marking the lab complete, verify every item.

### VPC

* [ ] `MetaPi-MultiAZ-VPC` exists.
* [ ] Public-A exists.
* [ ] Public-B exists.
* [ ] Public-A and Public-B are in different Availability Zones.

### Security

* [ ] `MetaPi-Lab15-ALB-SG` exists.
* [ ] ALB Security Group allows HTTP `80` from `0.0.0.0/0`.
* [ ] `MetaPi-Lab15-Web-SG` exists.
* [ ] Web Security Group allows HTTP `80` from the ALB Security Group.
* [ ] SSH `22` is restricted to My IP.

### EC2

* [ ] `MetaPi-Lab15-Web-A` is running.
* [ ] `MetaPi-Lab15-Web-B` is running.
* [ ] Servers are deployed in different Availability Zones.
* [ ] Apache is running.
* [ ] Both web pages work.

### Target Group

* [ ] `MetaPi-Lab15-TG` exists.
* [ ] Protocol is HTTP.
* [ ] Port is 80.
* [ ] Health check path is `/`.
* [ ] Web Server A is registered.
* [ ] Web Server B is registered.
* [ ] Both targets become healthy.

### ALB

* [ ] `MetaPi-Lab15-ALB` exists.
* [ ] Scheme is Internet-facing.
* [ ] IPv4 is configured.
* [ ] ALB spans two Availability Zones.
* [ ] HTTP listener exists on port 80.
* [ ] Listener forwards traffic to `MetaPi-Lab15-TG`.
* [ ] ALB DNS name works.

### High Availability Test

* [ ] Web Server A was intentionally stopped.
* [ ] Web Server A became unhealthy.
* [ ] ALB continued serving traffic through Web Server B.
* [ ] Web Server A was started again.
* [ ] Web Server A returned to healthy state.

---

# 🧠 Key Concepts Learned

After completing this lab, you should understand:

### Application Load Balancer

An ALB distributes application-level HTTP/HTTPS traffic across registered targets.

### Target Group

A target group contains the backend resources that receive traffic from the ALB.

### Health Checks

Health checks allow the ALB to determine whether backend targets are available to receive traffic.

### Security Group Referencing

An EC2 Security Group can allow traffic from another Security Group.

This is useful for creating controlled communication between AWS resources.

### Multi-AZ Architecture

Deploying backend servers across multiple Availability Zones improves availability and reduces dependence on a single Availability Zone.

### Fault Tolerance

When one target becomes unhealthy, the ALB can continue routing traffic to healthy targets.

---

# 💰 Cost Awareness

This lab creates AWS resources that may incur charges depending on your account, region, free-tier eligibility, and current AWS pricing.

Potential billable resources include:

* Application Load Balancer.
* EC2 instances.
* EBS storage.
* Public IPv4 addresses.
* Other networking resources already created in previous labs.

> **Important:** Always review the current AWS pricing and your account's free-tier eligibility before starting the lab.

For a training environment, clean up resources when they are no longer required.

---

# 🧹 Cleanup

If you are continuing to the next lab and the resources are required, keep them.

Otherwise, clean up the resources created specifically for Lab 015.

Recommended cleanup order:

### 1. Delete the Application Load Balancer

Navigate to:

**EC2 → Load Balancers**

Select:

```text
MetaPi-Lab15-ALB
```

Delete it.

---

### 2. Delete the Target Group

Navigate to:

**EC2 → Target Groups**

Select:

```text
MetaPi-Lab15-TG
```

Delete it.

---

### 3. Terminate EC2 Instances

Terminate:

```text
MetaPi-Lab15-Web-A
MetaPi-Lab15-Web-B
```

---

### 4. Delete Security Groups

After the dependent resources have been removed, delete:

```text
MetaPi-Lab15-ALB-SG
MetaPi-Lab15-Web-SG
```

> **Warning:** Do not delete Security Groups or VPC resources that are being used by another lab.

---

# 🏁 Lab Completed

🎉 **Congratulations!**

You have successfully built a highly available application delivery setup using an **AWS Application Load Balancer**.

You created:

```text
                 🌐 Internet
                      |
                      v
               Application ALB
                      |
                      v
                Target Group
                 /         \
                /           \
               v             v
          EC2 Server A   EC2 Server B
             AZ-A           AZ-B
```

You also verified:

* Multi-AZ deployment
* Security Group based communication
* Target registration
* Health checks
* ALB traffic distribution
* Backend failure handling
* Automatic removal of unhealthy targets
* Recovery of a healthy target

---

## 🚀 Next Lab

**Lab 016 — Create an EC2 Auto Scaling Group**

In the next lab, you will extend this architecture by introducing **EC2 Auto Scaling**, allowing the environment to automatically launch and terminate EC2 instances based on defined scaling requirements.

---

> **AWS Hands-On Labs Roadmap**
>
> **Learn AWS by building it, testing it, breaking it, troubleshooting it, and understanding why it works.**
