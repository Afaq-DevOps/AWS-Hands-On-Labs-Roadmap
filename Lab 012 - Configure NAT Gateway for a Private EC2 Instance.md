# Lab 12 — Configure NAT Gateway for a Private EC2 Instance

## 🎯 Lab Objective

In this lab, you will learn how to provide **outbound Internet access to an EC2 instance located in a private subnet** using an AWS NAT Gateway.

You will also learn how to securely access the private EC2 instance through a **Bastion Host**.

By the end of this lab, your architecture will look like this:

```text
                         INTERNET
                            |
                            |
                    Internet Gateway
                            |
                            |
                    Public Subnet
                            |
                    +----------------+
                    |  NAT Gateway   |
                    |  Elastic IP    |
                    +----------------+
                            |
                            |
                    Private Route Table
                            |
                            |
                    Private Subnet
                            |
                    +----------------+
                    | Private EC2    |
                    | No Public IP   |
                    +----------------+


Administrator PC
       |
       | SSH
       v
+----------------+
| Bastion EC2    |
| Public Subnet  |
| Public IP      |
+----------------+
       |
       | SSH
       v
+----------------+
| Private EC2    |
| Private Subnet |
| No Public IP   |
+----------------+
```

---

# 🧠 Before Starting: What Are We Building?

Imagine your private EC2 is a person living inside a house.

The person needs to go outside to buy something, but you do **not** want strangers from outside to directly enter the house.

The NAT Gateway works like a **one-way exit door**.

The private EC2 can:

```text
Private EC2 → NAT Gateway → Internet
```

But the Internet cannot directly initiate a connection to:

```text
Internet → Private EC2
```

To securely manage the private EC2, we will use a Bastion Host.

```text
Your Computer
      |
      v
Bastion EC2
      |
      v
Private EC2
```

---

# 📋 Prerequisites

Before starting this lab, you should already have the following from Lab 11:

```text
MetaPi-VPC
MetaPi-Public-Subnet-A
MetaPi-Private-Subnet-A
MetaPi-Public-RT
MetaPi-Private-RT
MetaPi-IGW
```

You should also have:

```text
AWS Account
EC2 Key Pair
SSH Client
```

For this lab we will use:

```text
Region: Same region used in Lab 11
```

> ⚠️ Keep the same AWS Region throughout the lab.

---

# Step 1 — Verify the Lab 11 Network

Before creating anything, verify that the previous lab's network still exists.

Go to:

```text
AWS Console
→ VPC
→ Your VPCs
```

Find:

```text
MetaPi-VPC
```

Open the VPC and verify that the following resources exist:

```text
MetaPi-Public-Subnet-A
MetaPi-Private-Subnet-A
MetaPi-Public-RT
MetaPi-Private-RT
MetaPi-IGW
```

## ✅ Checkpoint 1

You should have:

```text
VPC
 ├── Public Subnet
 ├── Private Subnet
 ├── Public Route Table
 ├── Private Route Table
 └── Internet Gateway
```

If these resources do not exist, stop here and recreate the Lab 11 network before continuing.

---

# Step 2 — Understand Why We Need a NAT Gateway

Our private EC2 will not have a public IP address.

Therefore:

```text
Private EC2
      |
      X
      |
Internet
```

will not work directly.

We need:

```text
Private EC2
      |
      v
NAT Gateway
      |
      v
Internet Gateway
      |
      v
Internet
```

The NAT Gateway will be placed in the **public subnet**.

> ⭐ Important:
> 
> **NAT Gateway → Public Subnet**
> 
> **Private EC2 → Private Subnet**

Do not reverse these.

---

# Step 3 — Allocate an Elastic IP

The public NAT Gateway requires an Elastic IP address.

Go to:

```text
AWS Console
→ VPC
→ Elastic IP addresses
```

Click:

```text
Allocate Elastic IP address
```

Keep the default settings unless your AWS console asks for a specific allocation option.

After allocation, select the new Elastic IP.

Add a Name tag:

```text
MetaPi-Lab12-NAT-EIP
```

## ✅ Checkpoint 2

You should now see:

```text
MetaPi-Lab12-NAT-EIP
```

with an allocated IPv4 address.

---

# Step 4 — Create the NAT Gateway

Go to:

```text
VPC
→ NAT gateways
```

Click:

```text
Create NAT gateway
```

Configure it as follows:

### Name

```text
MetaPi-NAT-Gateway
```

### Subnet

Select:

```text
MetaPi-Public-Subnet-A
```

### Connectivity Type

Select:

```text
Public
```

### Elastic IP

Select:

```text
MetaPi-Lab12-NAT-EIP
```

Then click:

```text
Create NAT gateway
```

Wait until the NAT Gateway status changes from:

```text
Pending
```

to:

```text
Available
```

## ⚠️ Important

Do not continue until the NAT Gateway shows:

```text
Available
```

## ✅ Checkpoint 3

Your NAT Gateway should show:

```text
Name:
MetaPi-NAT-Gateway

Subnet:
MetaPi-Public-Subnet-A

Connectivity:
Public

Elastic IP:
MetaPi-Lab12-NAT-EIP

Status:
Available
```

---

# Step 5 — Configure the Private Route Table

Now we need to tell the private subnet:

> "Whenever you want to access the Internet, send the traffic to the NAT Gateway."

Go to:

```text
VPC
→ Route tables
```

Find:

```text
MetaPi-Private-RT
```

Open it.

Go to:

```text
Routes
→ Edit routes
```

Click:

```text
Add route
```

Configure:

```text
Destination:
0.0.0.0/0
```

For the target, select:

```text
NAT Gateway
```

Then select:

```text
MetaPi-NAT-Gateway
```

Save the route.

Your route table should now contain something similar to:

```text
Destination       Target

10.10.0.0/16      local
0.0.0.0/0         nat-xxxxxxxx
```

## 🧠 What does this mean?

This:

```text
0.0.0.0/0
```

means:

> Any destination that is not inside my VPC.

The route says:

```text
Unknown destination
        |
        v
NAT Gateway
```

## ✅ Checkpoint 4

Verify:

```text
MetaPi-Private-RT
        |
        └── 0.0.0.0/0 → MetaPi-NAT-Gateway
```

---

# Step 6 — Create the Bastion Security Group

We need a Bastion Host because our private EC2 does not have a public IP.

Go to:

```text
EC2
→ Security Groups
→ Create security group
```

Create:

```text
Security group name:
MetaPi-Bastion-SG
```

Select:

```text
VPC:
MetaPi-VPC
```

### Inbound Rule

Add:

```text
Type: SSH
Port: 22
Source: My IP
```

Do not use:

```text
0.0.0.0/0
```

for SSH in this training lab unless specifically required.

### Outbound

Leave the default outbound rule.

Create the security group.

## ✅ Checkpoint 5

You should now have:

```text
MetaPi-Bastion-SG

Inbound:
SSH 22 → My IP
```

---

# Step 7 — Create the Private EC2 Security Group

Now create the security group for the private EC2.

Go to:

```text
EC2
→ Security Groups
→ Create security group
```

Configure:

```text
Security group name:
MetaPi-Private-EC2-SG
```

Select:

```text
VPC:
MetaPi-VPC
```

### Inbound Rule

Add:

```text
Type: SSH
Port: 22
Source: Custom
```

For the source, select:

```text
MetaPi-Bastion-SG
```

This means:

```text
Only EC2 instances using MetaPi-Bastion-SG
can SSH to the private EC2.
```

## 🧠 Why not My IP?

Because the private EC2 is not directly accessed from your computer.

The connection path will be:

```text
Your Computer
      |
      v
Bastion
      |
      v
Private EC2
```

Therefore the private EC2 should trust the Bastion Security Group.

## ✅ Checkpoint 6

Your private EC2 security group should show:

```text
MetaPi-Private-EC2-SG

Inbound:
SSH 22 → MetaPi-Bastion-SG
```

---

# Step 8 — Launch the Bastion EC2

Go to:

```text
EC2
→ Instances
→ Launch instance
```

Configure:

### Name

```text
MetaPi-Lab12-Bastion
```

### AMI

Use:

```text
Ubuntu
```

### Instance Type

Use the lab-approved/free-tier-compatible instance type available in your account.

### Key Pair

Select the same key pair you will use for the private EC2.

For example:

```text
MetaPi-Lab12-Bastion
```

or your existing lab key.

### Network Settings

Select:

```text
VPC:
MetaPi-VPC
```

### Subnet

Select:

```text
MetaPi-Public-Subnet-A
```

### Auto-assign Public IP

Set:

```text
Enable
```

### Security Group

Select:

```text
MetaPi-Bastion-SG
```

Launch the instance.

---

# Step 9 — Verify the Bastion

Wait until:

```text
Instance state:
Running
```

and:

```text
Status check:
2/2 checks passed
```

Open the instance details.

Find:

```text
Public IPv4 address
```

Copy it.

For example:

```text
13.xx.xx.xx
```

Do not use the example IP above. Use the IP displayed in your AWS console.

---

# Step 10 — Connect to the Bastion

Open:

```text
Git Bash
```

Go to the directory containing your key.

For example:

```bash
cd ~/Downloads
```

Check that your key exists:

```bash
ls
```

You should see your `.pem` file.

Now connect:

```bash
ssh -i "./your-key.pem" ubuntu@<BASTION_PUBLIC_IP>
```

Example:

```bash
ssh -i "./MetaPi-Lab12-Bastion.pem" ubuntu@13.xx.xx.xx
```

If prompted:

```text
Are you sure you want to continue connecting?
```

type:

```text
yes
```

## ✅ Checkpoint 7

Successful login should look similar to:

```text
Welcome to Ubuntu
```

and:

```text
ubuntu@ip-10-10-x-x:~$
```

This confirms:

```text
Your Computer
      |
      | SSH
      v
Bastion EC2
```

is working.

---

# Step 11 — Launch the Private EC2

Now launch the private EC2.

Go to:

```text
EC2
→ Instances
→ Launch instance
```

Configure:

### Name

```text
MetaPi-Lab12-Private-EC2
```

### AMI

Use:

```text
Ubuntu
```

### Key Pair

Use the same key pair used for the Bastion.

### Network

Select:

```text
VPC:
MetaPi-VPC
```

### Subnet

Select:

```text
MetaPi-Private-Subnet-A
```

### Auto-assign Public IP

This is extremely important.

Set:

```text
Disable
```

### Security Group

Select:

```text
MetaPi-Private-EC2-SG
```

Launch the instance.

---

# Step 12 — Verify That the Private EC2 Is Actually Private

Open:

```text
EC2
→ Instances
→ MetaPi-Lab12-Private-EC2
```

Look at:

```text
Public IPv4 address
```

It should show:

```text
None
```

You should have a:

```text
Private IPv4 address
```

For example:

```text
10.10.11.157
```

Copy the private IPv4 address.

## 🚨 Important

Do not try this from your computer:

```bash
ssh ubuntu@10.10.11.157
```

Your computer cannot directly reach that private IP over the Internet.

We need the Bastion.

---

# Step 13 — Understand the SSH Path

Our goal is:

```text
Your Computer
      |
      | SSH
      v
Bastion
13.xx.xx.xx
      |
      | SSH forwarding
      v
Private EC2
10.10.xx.xx
```

The private EC2 remains private.

We do **not** copy the `.pem` file onto the Bastion.

Instead, your local computer uses the Bastion as a jump host.

---

# Step 14 — Connect to the Private EC2 Using ProxyJump

Exit the Bastion if you are currently inside it:

```bash
exit
```

You should return to your local Git Bash prompt.

Now use:

```bash
ssh -i "./your-key.pem" -J ubuntu@<BASTION_PUBLIC_IP> ubuntu@<PRIVATE_EC2_PRIVATE_IP>
```

Example:

```bash
ssh -i "./MetaPi-Lab12-Bastion.pem" -J ubuntu@13.xx.xx.xx ubuntu@10.10.11.157
```

## ⚠️ Important Authentication Note

Depending on your SSH client configuration, the Bastion connection may not automatically use the key specified for the final connection.

If you receive:

```text
Permission denied (publickey)
```

for the Bastion, do not panic.

Use the explicit ProxyCommand method:

```bash
ssh -i "./your-key.pem" -o "ProxyCommand=ssh -i ./your-key.pem -W %h:%p ubuntu@<BASTION_PUBLIC_IP>" ubuntu@<PRIVATE_EC2_PRIVATE_IP>
```

Example:

```bash
ssh -i "./MetaPi-Lab12-Bastion.pem" -o "ProxyCommand=ssh -i ./MetaPi-Lab12-Bastion.pem -W %h:%p ubuntu@13.xx.xx.xx" ubuntu@10.10.11.157
```

This explicitly tells SSH:

```text
Use my PEM key
       |
       v
Connect to Bastion
       |
       v
Forward traffic
       |
       v
Private EC2
```

## 🚫 Do NOT do this

Do not copy your private key to the Bastion and then run:

```bash
ssh -i key.pem ubuntu@10.10.xx.xx
```

The private key should remain on your local computer.

---

# Step 15 — Confirm You Are Inside the Private EC2

If successful, your terminal should look similar to:

```text
ubuntu@ip-10-10-11-157:~$
```

The IP will be different in your environment.

Run:

```bash
hostname -I
```

You should see the private IP of the instance.

Example:

```text
10.10.11.157
```

## ✅ Checkpoint 8

You have successfully completed:

```text
Local PC
   ↓
Bastion
   ↓
Private EC2
```

without giving the private EC2 a public IP.

---

# Step 16 — Test Internet Access Through NAT Gateway

Now we will prove that the NAT Gateway is working.

From inside the private EC2, run:

```bash
curl -I https://aws.amazon.com
```

You should receive an HTTP response.

For example:

```text
HTTP/2 200
```

The exact response can vary.

Now run:

```bash
sudo apt update
```

The system should be able to contact Ubuntu package repositories.

## 🧠 What just happened?

The private EC2 sent Internet traffic through:

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

The private EC2 itself still has:

```text
Public IPv4:
None
```

That is the main concept of this lab.

---

# Step 17 — Final Verification

Before declaring the lab complete, verify every item below.

### VPC

```text
MetaPi-VPC
```

### Public Subnet

```text
MetaPi-Public-Subnet-A
```

### Private Subnet

```text
MetaPi-Private-Subnet-A
```

### Internet Gateway

```text
MetaPi-IGW
```

### NAT Gateway

```text
MetaPi-NAT-Gateway
Status: Available
```

### NAT Elastic IP

```text
MetaPi-Lab12-NAT-EIP
```

### Private Route Table

```text
MetaPi-Private-RT

0.0.0.0/0
      ↓
MetaPi-NAT-Gateway
```

### Bastion

```text
MetaPi-Lab12-Bastion
Public IP: Yes
Subnet: Public
```

### Private EC2

```text
MetaPi-Lab12-Private-EC2
Public IP: None
Subnet: Private
```

### SSH Test

```text
Local PC
   ↓
Bastion
   ↓
Private EC2
```

### Internet Test

```bash
curl -I https://aws.amazon.com
```

### Package Test

```bash
sudo apt update
```

If all of these are successful:

# 🎉 LAB 12 IS COMPLETE

---

# 🧪 Troubleshooting Guide

## Problem 1 — Bastion SSH fails

If you see:

```text
Permission denied (publickey)
```

Check:

```text
Correct .pem file
Correct username
Correct Bastion public IP
Bastion Security Group allows SSH from My IP
```

For Ubuntu, the username is normally:

```text
ubuntu
```

---

## Problem 2 — Private EC2 SSH fails

Check:

```text
Private EC2 is running
Private EC2 Security Group allows SSH from MetaPi-Bastion-SG
Bastion uses MetaPi-Bastion-SG
Correct private IP is being used
Correct key is being used
```

---

## Problem 3 — Private EC2 has no Internet

Check these three things first:

### 1. NAT Gateway

```text
Status = Available
```

### 2. NAT Gateway subnet

It must be:

```text
MetaPi-Public-Subnet-A
```

### 3. Private route table

It must contain:

```text
0.0.0.0/0 → NAT Gateway
```

If any one of these is wrong, NAT Internet access will fail.

---

## Problem 4 — Private EC2 has a Public IPv4 address

This means the instance was not launched as a proper private instance.

Verify:

```text
Auto-assign Public IP = Disabled
```

The final private EC2 should show:

```text
Public IPv4 address:
None
```

---

# 🧠 Knowledge Check

Before moving to the next lab, answer these questions:

### Q1. Why is the NAT Gateway placed in the public subnet?

### Q2. Why does the private EC2 not need a public IP?

### Q3. What does this route mean?

```text
0.0.0.0/0 → NAT Gateway
```

### Q4. Can the Internet directly initiate an SSH connection to the private EC2?

### Q5. Why do we use a Bastion Host?

### Q6. What is the difference between the NAT Gateway and the Internet Gateway?

### Q7. Why should the private EC2 Security Group allow SSH from `MetaPi-Bastion-SG` instead of `0.0.0.0/0`?

---

# 🧹 Step 18 — Cleanup

⚠️ NAT Gateway is a billable AWS resource.

When the lab is finished, clean up the resources.

## If the private EC2 was created specifically for this lab

Terminate:

```text
MetaPi-Lab12-Private-EC2
```

Terminate:

```text
MetaPi-Lab12-Bastion
```

Then delete:

```text
MetaPi-NAT-Gateway
```

Wait until the NAT Gateway is completely deleted.

Then release:

```text
MetaPi-Lab12-NAT-EIP
```

Finally, if the VPC will be reused for future labs, remove:

```text
0.0.0.0/0 → NAT Gateway
```

from:

```text
MetaPi-Private-RT
```

## ⚠️ Important Cleanup Rule

Do not delete resources belonging to previous labs unless the lab instructions specifically tell you to.

For example:

```text
MetaPi-Lab11-Private-EC2
```

should not be terminated simply because you are cleaning up Lab 12.

---

# 🎯 What You Learned

After completing this lab, you should understand:

- What a NAT Gateway is
    
- Why a private subnet needs NAT for outbound Internet access
    
- Why NAT Gateway belongs in a public subnet
    
- How route tables control traffic
    
- How a private EC2 accesses the Internet without a public IP
    
- What a Bastion Host is
    
- How Security Group references work
    
- How to securely reach a private EC2 through a Bastion
    
- How SSH ProxyJump works
    
- Why the private key should remain on your local computer
    
- How to verify NAT functionality using `curl`
    
- How to verify package access using `apt update`
    
- How to clean up NAT resources safely
    

---

# 🏆 Lab Success Criteria

You can mark **Lab 12 as successfully completed** when all of these are true:

```text
[✓] NAT Gateway is Available
[✓] NAT Gateway is in Public Subnet
[✓] NAT Gateway has Elastic IP
[✓] Private Route Table points 0.0.0.0/0 to NAT Gateway
[✓] Bastion is in Public Subnet
[✓] Bastion has Public IP
[✓] Private EC2 is in Private Subnet
[✓] Private EC2 has no Public IP
[✓] Private EC2 is accessible through Bastion
[✓] curl to Internet works
[✓] sudo apt update works
[✓] Resources are cleaned up after the lab
```

# 🚀 Next Lab

## Lab 13 — Configure Security Groups and Network ACLs

In the next lab, you will learn how AWS controls network traffic using:

```text
Security Groups
        +
Network ACLs
```

You will compare:

```text
Stateful Security Groups
vs
Stateless Network ACLs
```

and perform practical traffic-control testing.