# Lab 14 — Build a Multi-AZ VPC Architecture

> **MetaPi AWS Hands-On Labs**
> 
> **Difficulty:** 🟡 Intermediate  
> **Estimated Time:** 60–90 minutes  
> **AWS Region:** Any single AWS Region  
> **Cost:** Low, but EC2/NAT Gateway can incur charges  
> **Prerequisites:** Labs 11–13 recommended

---

## 🎯 Lab Objective

In this lab, you will build a **Multi-AZ VPC architecture** with:

- 1 VPC
- 2 Availability Zones
- 2 Public Subnets
- 2 Private Subnets
- 1 Internet Gateway
- 1 Public Route Table
- 2 Private Route Tables
- Optional test EC2 instances

The final architecture will look like:

```
                         🌐 Internet
                              |
                              |
                     Internet Gateway
                              |
                +-------------+-------------+
                |                           |
              AZ-A                         AZ-B
                |                           |
        +-------+-------+           +-------+-------+
        |               |           |               |
     Public-A        Private-A   Public-B        Private-B
   10.20.1.0/24    10.20.11.0/24 10.20.2.0/24   10.20.12.0/24
        |               |           |               |
      EC2            Future       EC2            Future
                   Resources                   Resources
```

### 🧠 Why Multi-AZ?

A single Availability Zone can become unavailable.

Instead of putting everything here:

```
Region
  |
  └── AZ-A
       |
       └── Everything
```

we distribute resources:

```
Region
 |
 +── AZ-A
 |    +── Public-A
 |    └── Private-A
 |
 └── AZ-B
      +── Public-B
      └── Private-B
```

This gives our architecture **fault-isolation and high-availability foundations**.

---

# 📋 Architecture Plan

Before creating anything, understand the addressing plan.

|Resource|CIDR|AZ|
|---|---|---|
|VPC|`10.20.0.0/16`|Region-wide|
|Public-A|`10.20.1.0/24`|AZ-A|
|Public-B|`10.20.2.0/24`|AZ-B|
|Private-A|`10.20.11.0/24`|AZ-A|
|Private-B|`10.20.12.0/24`|AZ-B|

### ⚠️ Important

**Do not assume `eu-north-1a` and `eu-north-1b` are always the same physical Availability Zones for every AWS account.**

That's why throughout this lab we'll use:

```
AZ-A
AZ-B
```

and simply select **two different AZs in your selected Region**.

---

# Step 1 — Select Your AWS Region

Open the AWS Console and go to:

**VPC → Your VPCs**

Choose **one Region** for the complete lab.

For example:

```
Region: Europe (Stockholm)
```

You can use another Region if required.

### ✅ Checkpoint

Make sure all resources created in this lab are inside the **same Region**.

---

# Step 2 — Create the VPC

Go to:

**VPC → Your VPCs → Create VPC**

Select:

```
Resources to create:
VPC only
```

Enter:

```
Name tag:
MetaPi-MultiAZ-VPC

IPv4 CIDR:
10.20.0.0/16
```

Leave IPv6 disabled unless your instructor specifically requires it.

Click:

**Create VPC**

### ✅ Checkpoint

You should now have:

```
MetaPi-MultiAZ-VPC
10.20.0.0/16
```

---

# Step 3 — Identify Two Availability Zones

Go to:

**EC2 → Instances → Launch Instance**

or:

**VPC → Subnets → Create subnet**

Look at the available Availability Zones.

Choose **two different AZs**.

For example, your account might show:

```
AZ-A → eu-north-1a
AZ-B → eu-north-1b
```

Another account might map differently.

### 🧠 Remember

We care about:

```
AZ-A ≠ AZ-B
```

not about the letters themselves.

---

# Step 4 — Create Public Subnet A

Go to:

**VPC → Subnets → Create subnet**

Select:

```
VPC:
MetaPi-MultiAZ-VPC
```

Enter:

```
Subnet name:
MetaPi-Public-A

Availability Zone:
AZ-A

IPv4 subnet CIDR:
10.20.1.0/24
```

Click:

**Create subnet**

---

# Step 5 — Create Public Subnet B

Create another subnet.

Use:

```
Subnet name:
MetaPi-Public-B

Availability Zone:
AZ-B

IPv4 subnet CIDR:
10.20.2.0/24
```

Click:

**Create subnet**

---

# Step 6 — Enable Public IPv4 for Public Subnets

Select:

```
MetaPi-Public-A
```

Go to:

**Actions → Edit subnet settings**

Enable:

```
Enable auto-assign public IPv4 address
```

Save.

Repeat for:

```
MetaPi-Public-B
```

### ✅ Checkpoint

Both public subnets should show:

```
Auto-assign public IPv4: Yes
```

---

# Step 7 — Create Private Subnet A

Create another subnet:

```
Subnet name:
MetaPi-Private-A

Availability Zone:
AZ-A

IPv4 CIDR:
10.20.11.0/24
```

Click:

**Create subnet**

---

# Step 8 — Create Private Subnet B

Create:

```
Subnet name:
MetaPi-Private-B

Availability Zone:
AZ-B

IPv4 CIDR:
10.20.12.0/24
```

Click:

**Create subnet**

---

# Step 9 — Verify the Four Subnets

You should now have:

```
MetaPi-MultiAZ-VPC
│
├── MetaPi-Public-A
│   └── 10.20.1.0/24
│
├── MetaPi-Public-B
│   └── 10.20.2.0/24
│
├── MetaPi-Private-A
│   └── 10.20.11.0/24
│
└── MetaPi-Private-B
    └── 10.20.12.0/24
```

And:

```
Public-A  → AZ-A
Public-B  → AZ-B

Private-A → AZ-A
Private-B → AZ-B
```

### 🛑 STOP & CHECK

Before continuing, make sure:

- [ ]  4 subnets exist
- [ ]  All 4 belong to `MetaPi-MultiAZ-VPC`
- [ ]  Public-A and Private-A use the same AZ
- [ ]  Public-B and Private-B use the same AZ
- [ ]  AZ-A and AZ-B are different
- [ ]  CIDRs don't overlap

---

# Step 10 — Create Internet Gateway

Go to:

**VPC → Internet Gateways**

Click:

**Create internet gateway**

Name:

```
MetaPi-MultiAZ-IGW
```

Click:

**Create internet gateway**

Then select it:

**Actions → Attach to a VPC**

Choose:

```
MetaPi-MultiAZ-VPC
```

Click:

**Attach internet gateway**

### ✅ Checkpoint

The IGW state should show:

```
Attached
```

---

# Step 11 — Create Public Route Table

Go to:

**VPC → Route Tables → Create route table**

Enter:

```
Name:
MetaPi-MultiAZ-Public-RT

VPC:
MetaPi-MultiAZ-VPC
```

Create it.

---

# Step 12 — Add Internet Route

Select:

```
MetaPi-MultiAZ-Public-RT
```

Go to:

**Routes → Edit routes → Add route**

Enter:

```
Destination:
0.0.0.0/0

Target:
Internet Gateway

MetaPi-MultiAZ-IGW
```

Save.

Your route table should now contain:

```
Destination       Target

10.20.0.0/16      local
0.0.0.0/0         MetaPi-MultiAZ-IGW
```

### 🧠 Understand This

The:

```
10.20.0.0/16 → local
```

route allows communication inside the VPC.

The:

```
0.0.0.0/0 → IGW
```

route provides a path toward the Internet.

---

# Step 13 — Associate Public Subnet A

Inside:

**MetaPi-MultiAZ-Public-RT**

go to:

**Subnet associations → Edit subnet associations**

Select:

```
MetaPi-Public-A
```

Save.

---

# Step 14 — Associate Public Subnet B

Repeat and associate:

```
MetaPi-Public-B
```

Your public route table should now be associated with:

```
MetaPi-Public-A
MetaPi-Public-B
```

---

# Step 15 — Create Private Route Table A

Create another route table:

```
Name:
MetaPi-Private-A-RT

VPC:
MetaPi-MultiAZ-VPC
```

Associate:

```
MetaPi-Private-A
```

### Important

For **this lab**, don't add an Internet Gateway route to the private route table.

It should have only:

```
10.20.0.0/16 → local
```

---

# Step 16 — Create Private Route Table B

Create:

```
Name:
MetaPi-Private-B-RT

VPC:
MetaPi-MultiAZ-VPC
```

Associate:

```
MetaPi-Private-B
```

Again, for this lab:

```
10.20.0.0/16 → local
```

only.

---

# 🛑 Checkpoint — Route Tables

You should now have:

### Public Route Table

```
MetaPi-MultiAZ-Public-RT

10.20.0.0/16 → local
0.0.0.0/0    → Internet Gateway
```

Associated with:

```
Public-A
Public-B
```

### Private Route Table A

```
MetaPi-Private-A-RT

10.20.0.0/16 → local
```

Associated with:

```
Private-A
```

### Private Route Table B

```
MetaPi-Private-B-RT

10.20.0.0/16 → local
```

Associated with:

```
Private-B
```

---

# Step 17 — Understand the Current Traffic Flow

At this point:

### Public subnet

```
EC2
 |
 v
Public Route Table
 |
 v
Internet Gateway
 |
 v
Internet
```

### Private subnet

```
EC2
 |
 v
Private Route Table
 |
 v
No Internet Gateway route
```

Therefore, a private EC2 **should not have direct Internet access** at this stage.

This is intentional.

> 💡 **NAT Gateway will be introduced in a later lab/design stage.**

---

# Step 18 — Create Security Group for Test EC2

Go to:

**EC2 → Security Groups → Create security group**

Name:

```
MetaPi-MultiAZ-Test-SG
```

Description:

```
Security group for Multi-AZ test EC2 instances
```

VPC:

```
MetaPi-MultiAZ-VPC
```

### Inbound Rules

SSH:

```
Type: SSH
Port: 22
Source: My IP
```

HTTP:

```
Type: HTTP
Port: 80
Source: 0.0.0.0/0
```

Create the security group.

---

# Step 19 — Launch Test EC2 in Public-A

Launch a small EC2 instance.

Use:

```
Name:
MetaPi-MultiAZ-Public-EC2-A
```

Select:

```
VPC:
MetaPi-MultiAZ-VPC

Subnet:
MetaPi-Public-A

Auto-assign Public IP:
Enable
```

Attach:

```
MetaPi-MultiAZ-Test-SG
```

Use your appropriate key pair.

---

# Step 20 — Launch Test EC2 in Public-B

Launch another small EC2.

Use:

```
Name:
MetaPi-MultiAZ-Public-EC2-B
```

Select:

```
VPC:
MetaPi-MultiAZ-VPC

Subnet:
MetaPi-Public-B

Auto-assign Public IP:
Enable
```

Attach:

```
MetaPi-MultiAZ-Test-SG
```

---

# Step 21 — Install Apache

Use this User Data on both instances:

```
#!/bin/bash

apt-get update -y
apt-get install -y apache2

HOSTNAME=$(hostname)

echo "<h1>MetaPi Multi-AZ</h1>" > /var/www/html/index.html
echo "<p>Server: $HOSTNAME</p>" >> /var/www/html/index.html

systemctl enable --now apache2
```

---

# Step 22 — Test Public EC2-A

After the instance passes:

```
3/3 status checks
```

copy its public IPv4 address.

Open:

```
http://PUBLIC-IP-A
```

You should see:

```
MetaPi Multi-AZ

Server: ip-10-20-x-x
```

---

# Step 23 — Test Public EC2-B

Repeat for:

```
MetaPi-MultiAZ-Public-EC2-B
```

Open:

```
http://PUBLIC-IP-B
```

You should also receive the Apache page.

---

# 🧪 Lab Verification

Run the following checks.

## VPC

```
Name:
MetaPi-MultiAZ-VPC

CIDR:
10.20.0.0/16
```

## Subnets

```
Public-A:
10.20.1.0/24 → AZ-A

Public-B:
10.20.2.0/24 → AZ-B

Private-A:
10.20.11.0/24 → AZ-A

Private-B:
10.20.12.0/24 → AZ-B
```

## Internet Gateway

```
MetaPi-MultiAZ-IGW
        |
        v
MetaPi-MultiAZ-VPC
```

## Public Route Table

```
10.20.0.0/16 → local
0.0.0.0/0    → IGW
```

Associations:

```
Public-A
Public-B
```

## Private Route Tables

```
Private-A-RT → Private-A
Private-B-RT → Private-B
```

Both should **NOT** have:

```
0.0.0.0/0 → Internet Gateway
```

---

# 🧠 Important Concept — Public vs Private Subnet

A subnet is not "public" simply because we named it Public.

A subnet becomes effectively public when:

1. Its route table has a route to an Internet Gateway.
2. The resource has a public IPv4 address/EIP.
3. Security groups/NACLs allow the required traffic.

Therefore:

```
Public Subnet
     |
Route Table
     |
0.0.0.0/0
     |
Internet Gateway
     |
Internet
```

Whereas:

```
Private Subnet
     |
Private Route Table
     |
No direct IGW route
```

---

# 🌐 Final Architecture

```
                         INTERNET
                             |
                             v
                   +-------------------+
                   | Internet Gateway  |
                   | MetaPi-MultiAZ-IGW|
                   +---------+---------+
                             |
                    Public Route Table
                  MetaPi-MultiAZ-Public-RT
                             |
              +--------------+--------------+
              |                             |
             AZ-A                          AZ-B
              |                             |
      +-------+-------+             +-------+-------+
      |               |             |               |
      v               v             v               v
 Public-A         Private-A       Public-B       Private-B
10.20.1.0/24    10.20.11.0/24   10.20.2.0/24   10.20.12.0/24
      |                               |
      v                               v
   EC2-A                             EC2-B
```

---

# 💰 Cost Warning

For this lab, the main potential charges can come from:

- EC2 instances
- EBS volumes
- Elastic IPs, depending on usage/state
- NAT Gateway **if you create one**

### ⚠️ Important

**Do NOT create a NAT Gateway just for this lab unless instructed.**

NAT Gateway has hourly and data-processing charges.

We'll deal with NAT Gateway architecture separately.

---

# ❌ Common Mistakes

### Mistake 1 — Both public subnets in same AZ

Wrong:

```
Public-A → AZ-A
Public-B → AZ-A
```

Correct:

```
Public-A → AZ-A
Public-B → AZ-B
```

---

### Mistake 2 — Private subnet has IGW route

Wrong:

```
Private-RT

0.0.0.0/0 → Internet Gateway
```

For this lab, don't do this.

---

### Mistake 3 — Overlapping CIDRs

Don't create:

```
10.20.1.0/24
10.20.1.0/24
```

Each subnet must have a unique CIDR.

---

### Mistake 4 — Assuming AZ letters are universal

Don't write your architecture as:

```
eu-north-1a = AZ-A everywhere
```

Instead:

```
AZ-A = first selected AZ
AZ-B = second selected AZ
```

---

# 🎓 What You Learned

After completing this lab, you should understand:

- What a VPC is
- What an Availability Zone is
- Why multiple AZs are used
- Public vs private subnets
- Internet Gateway
- Route tables
- CIDR planning
- Subnet association
- Public EC2 deployment
- Basic high-availability network design

---

# 📝 Trainee Challenge

Before moving to the next lab, answer these:

### Q1

Why shouldn't both public subnets be placed in the same Availability Zone?

### Q2

What makes a subnet effectively public?

### Q3

Why do we create separate private route tables for AZ-A and AZ-B?

### Q4

Can a private subnet directly use an Internet Gateway for outbound Internet access?

### Q5

If AZ-A becomes unavailable, what advantage does having AZ-B provide?

---

# 🧹 Cleanup

### If you are continuing to the next labs

**Do not delete the VPC.**

Keep:

```
MetaPi-MultiAZ-VPC
```

because future labs can build on this architecture.

### If you are finished with the complete training

Clean up resources in the appropriate dependency order:

```
Terminate EC2
     ↓
Delete NAT Gateway (if created)
     ↓
Release Elastic IP (if no longer needed)
     ↓
Delete route tables
     ↓
Detach/Delete Internet Gateway
     ↓
Delete subnets
     ↓
Delete VPC
```

---

# ✅ Lab Completion Checklist

```
[ ] VPC created
[ ] CIDR = 10.20.0.0/16

[ ] Public-A created
[ ] Public-B created

[ ] Private-A created
[ ] Private-B created

[ ] Public-A and Private-A are in AZ-A
[ ] Public-B and Private-B are in AZ-B

[ ] Internet Gateway created
[ ] Internet Gateway attached to VPC

[ ] Public Route Table created
[ ] 0.0.0.0/0 → IGW

[ ] Public-A associated
[ ] Public-B associated

[ ] Private-A Route Table created
[ ] Private-B Route Table created

[ ] No direct IGW route in private route tables

[ ] Public EC2-A tested
[ ] Public EC2-B tested

[ ] Apache page accessible from both EC2 instances
```

---

# 🏆 Lab 14 Completed!

You have successfully built the **network foundation for a Multi-AZ AWS architecture**.

Your VPC is now ready for future highly available workloads.

```
              Multi-AZ VPC
                    |
       +------------+------------+
       |                         |
      AZ-A                      AZ-B
       |                         |
    Public                    Public
       |                         |
   Private                    Private
```