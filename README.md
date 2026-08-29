# ☁️ AWS Hands-On Labs Roadmap

### MetaPi PSEB Training Program

A structured, practical AWS learning roadmap designed to help learners build real-world cloud skills through **hands-on labs, AWS Console practice, architecture design, troubleshooting, verification, and cost awareness**.

---

## 🎯 About This Repository

Welcome to the **AWS Hands-On Labs Roadmap**.

This repository contains a progressive series of practical AWS labs developed as part of the **MetaPi PSEB Training Program**.

The labs are designed to take learners from AWS fundamentals to practical cloud infrastructure concepts by building and testing real AWS resources.

> **Learn AWS by building it, testing it, breaking it, troubleshooting it, and understanding why it works.**

The roadmap currently contains **Labs 001–014**, with additional labs planned as the training progresses.

---

## 🧭 Learning Journey

The labs follow a progressive learning path:

```text
AWS Fundamentals
       ↓
Amazon EC2
       ↓
Web Server Deployment
       ↓
Automation
       ↓
Amazon EBS
       ↓
Backup & Recovery
       ↓
AWS IAM
       ↓
Amazon S3
       ↓
Application Deployment
       ↓
Amazon VPC
       ↓
NAT Gateway
       ↓
Network Security
       ↓
Multi-AZ Architecture
       ↓
More Advanced AWS & DevOps Concepts
```

---

# 📚 Labs 001–014

## 🟢 AWS & EC2 Fundamentals

### Lab 001 — Launch Your First EC2 Instance and Connect Using SSH

**Focus:** Amazon EC2, SSH, Security Groups and Linux

Learn how to launch an EC2 instance and securely connect to it using SSH.

**Key Concepts:**

* Amazon EC2
* AMIs
* Instance types
* Key pairs
* Security Groups
* Public IPv4
* SSH
* Linux basics

---

### Lab 002 — Deploy an HTML/CSS Website on EC2 Using Apache

**Focus:** Apache Web Server

Deploy a basic HTML/CSS website on an EC2 instance using Apache.

**Key Concepts:**

* Apache
* HTTP
* Port 80
* Linux package management
* Web server configuration
* Security Groups

---

### Lab 003 — Deploy a Website to EC2 Using SCP

**Focus:** Secure File Transfer

Learn how to transfer website files from a local machine to an EC2 instance using SCP.

**Key Concepts:**

* SCP
* SSH
* Secure file transfer
* Linux permissions
* Remote server management

---

### Lab 004 — Deploy an HTML Website Using Nginx

**Focus:** Nginx Web Server

Deploy a static HTML website using Nginx on an EC2 instance.

**Key Concepts:**

* Nginx
* Web server configuration
* HTTP
* Linux services
* EC2 networking

---

### Lab 005 — Automate EC2 Web Server Deployment Using User Data

**Focus:** EC2 Automation

Automate the installation and configuration of a web server during EC2 launch using User Data.

**Key Concepts:**

* EC2 User Data
* Bash scripting
* Bootstrapping
* Automated installation
* Service management
* Infrastructure automation fundamentals

---

# 💾 Storage & Backup

### Lab 006 — Create, Attach, Format and Mount an EBS Volume

**Focus:** Amazon EBS

Learn how to create and attach an EBS volume to an EC2 instance, format it, and mount it for persistent storage.

**Key Concepts:**

* Amazon EBS
* Block storage
* Filesystems
* Mount points
* Linux storage management
* Persistent storage

---

### Lab 007 — Create an EBS Snapshot and Restore Data

**Focus:** Backup & Recovery

Learn how to create an EBS snapshot and use it to restore storage and data.

**Key Concepts:**

* EBS snapshots
* Backup
* Recovery
* Data protection
* Restore operations
* Disaster recovery fundamentals

---

# 🔐 Identity & Access Management

### Lab 008 — Create IAM Users, Groups, Policies and Roles

**Focus:** AWS IAM

Learn how AWS Identity and Access Management controls authentication and authorization.

**Key Concepts:**

* IAM users
* IAM groups
* IAM policies
* IAM roles
* Permissions
* Least privilege
* Authentication
* Authorization

---

# 📦 Amazon S3

### Lab 009 — Create, Manage, and Host a Static Website on Amazon S3

**Focus:** Amazon S3 & Static Website Hosting

Learn how to create and manage an S3 bucket and use Amazon S3 for static website hosting.

**Key Concepts:**

* S3 buckets
* S3 objects
* Bucket policies
* Static website hosting
* Object storage
* Access control

---

# 🟠 Application Deployment

### Lab 010 — Deploy a PHP To-Do Application with MySQL on EC2

**Focus:** Application & Database Deployment

Deploy a PHP-based To-Do application with MySQL on an EC2 instance.

**Key Concepts:**

* PHP
* MySQL
* Apache
* Application deployment
* Database configuration
* EC2 application hosting

---

# 🌐 AWS Networking

### Lab 011 — Build a Custom VPC with Public and Private Subnets

**Focus:** Amazon VPC Fundamentals

Build a custom VPC containing public and private subnets and understand how AWS networking components work together.

**Key Concepts:**

* Amazon VPC
* CIDR blocks
* Public subnets
* Private subnets
* Route tables
* Internet Gateway
* Subnet associations

---

### Lab 012 — Configure NAT Gateway for a Private EC2 Instance

**Focus:** Private Subnet Connectivity

Configure a NAT Gateway so a private EC2 instance can initiate outbound Internet connections without being directly accessible from the Internet.

**Key Concepts:**

* NAT Gateway
* Elastic IP
* Private EC2
* Public subnet
* Private subnet
* Route tables
* Outbound Internet connectivity
* Bastion host concepts

---

### Lab 013 — Configure Security Groups and Network ACLs

**Focus:** AWS Network Security

Learn how Security Groups and Network ACLs control network traffic within a VPC.

**Key Concepts:**

* Security Groups
* Network ACLs
* Stateful vs stateless filtering
* Inbound rules
* Outbound rules
* SSH security
* HTTP security
* Network access control

---

### Lab 014 — Build a Multi-AZ VPC Architecture

**Focus:** Multi-AZ Networking & High Availability Foundations

Build a Multi-AZ VPC architecture using public and private subnets distributed across two Availability Zones.

**Key Concepts:**

* Availability Zones
* Multi-AZ architecture
* CIDR planning
* Public and private subnets
* Internet Gateway
* Route tables
* Subnet associations
* High availability foundations
* Fault isolation
* NAT Gateway architecture

### Architecture

```text
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
```

---

# 🛠️ AWS Services Covered

| AWS Service / Concept | Labs                  |
| --------------------- | --------------------- |
| Amazon EC2            | 001–005, 010, 012–014 |
| Amazon EBS            | 006–007               |
| AWS IAM               | 008                   |
| Amazon S3             | 009                   |
| Amazon VPC            | 011–014               |
| Internet Gateway      | 011, 014              |
| NAT Gateway           | 012, 014              |
| Route Tables          | 011–014               |
| Security Groups       | 001–005, 011–014      |
| Network ACLs          | 013                   |
| Availability Zones    | 014                   |

---

# 📊 Progress Tracker

|     Lab | Topic                               |    Status   |
| ------: | ----------------------------------- | :---------: |
|     001 | Launch EC2 & SSH                    | ✅ Completed |
|     002 | Apache Web Server                   | ✅ Completed |
|     003 | Website Deployment Using SCP        | ✅ Completed |
|     004 | Nginx Web Server                    | ✅ Completed |
|     005 | EC2 User Data                       | ✅ Completed |
|     006 | EBS Volume                          | ✅ Completed |
|     007 | EBS Snapshot & Restore              | ✅ Completed |
|     008 | IAM Users, Groups, Policies & Roles | ✅ Completed |
|     009 | S3 Static Website                   | ✅ Completed |
|     010 | PHP + MySQL Application             | ✅ Completed |
|     011 | Custom VPC                          | ✅ Completed |
|     012 | NAT Gateway & Private EC2           | ✅ Completed |
|     013 | Security Groups & Network ACLs      | ✅ Completed |
|     014 | Multi-AZ VPC Architecture           | ✅ Completed |
| 015–030 | Upcoming Labs                       |  🔜 Planned |

---

# 🏗️ Lab Philosophy

Each lab follows a practical learning approach:

```text
Understand
    ↓
Plan
    ↓
Build
    ↓
Test
    ↓
Troubleshoot
    ↓
Verify
    ↓
Clean Up
    ↓
Explain
```

The objective is not only to follow instructions, but to understand **why each AWS component is required and how the components work together**.

---

# 🚀 How to Use This Repository

### 1. Follow the Labs in Order

The labs are designed progressively, so learners should generally start from Lab 001 and move forward.

```text
Lab 001
   ↓
Lab 002
   ↓
Lab 003
   ↓
...
   ↓
Lab 014
```

Later labs may build on concepts introduced in earlier labs.

### 2. Read Before Creating Resources

Before starting a lab:

* Read the objective.
* Review the architecture.
* Understand the resources being created.
* Review CIDR ranges where applicable.
* Understand expected traffic flow.
* Check the cost considerations.

### 3. Build the Environment

Follow the lab instructions and create the required AWS resources.

### 4. Verify Everything

A lab is not complete simply because AWS resources were created.

Always verify:

```text
Resource
   ↓
Configuration
   ↓
Connectivity
   ↓
Expected Result
```

### 5. Clean Up

Delete or stop resources that are no longer required to avoid unnecessary AWS charges.

---

# 💰 AWS Cost Awareness

> ⚠️ **Important: AWS resources may incur charges.**

Depending on the lab, charges may come from resources such as:

* EC2 instances
* EBS volumes
* Elastic IP addresses
* NAT Gateways
* Data transfer
* Other AWS services

### Recommended Practice

After completing a lab:

```text
Review Resources
      ↓
Terminate unused EC2 instances
      ↓
Delete unused storage
      ↓
Delete unnecessary networking resources
      ↓
Release unused Elastic IPs
      ↓
Check AWS Billing
```

Always verify current AWS pricing and your account's Free Tier/credits before creating potentially billable resources.

---

# 🔐 Security Guidelines

When working with AWS:

* Never commit AWS access keys or secret credentials.
* Never upload `.pem` or private key files.
* Never publish passwords or tokens.
* Avoid exposing sensitive information in screenshots.
* Follow the principle of least privilege.
* Restrict SSH access whenever possible.
* Remove temporary resources after completing a lab.

### Never commit sensitive files such as:

```text
.env
*.pem
*.key
credentials
aws_access_key_id
aws_secret_access_key
```

---

# 🏷️ Resource Naming Convention

Where applicable, the labs use a consistent `MetaPi` naming convention.

Examples:

```text
MetaPi-MultiAZ-VPC
MetaPi-Public-A
MetaPi-Public-B
MetaPi-Private-A
MetaPi-Private-B
MetaPi-MultiAZ-IGW
MetaPi-MultiAZ-Public-RT
MetaPi-Private-A-RT
MetaPi-Private-B-RT
```

Consistent naming helps with:

* Resource identification
* Troubleshooting
* Lab management
* Resource cleanup
* Avoiding accidental changes to unrelated resources

---

# 📝 Lab Documentation Standards

Each lab aims to provide:

* 🎯 Clear objectives
* 📋 Prerequisites
* 🏗️ Architecture overview
* ⚙️ Step-by-step instructions
* 🔧 Configuration details
* 🧪 Verification steps
* 🛠️ Troubleshooting guidance
* ⚠️ Common mistakes
* 💰 Cost considerations
* 🧹 Cleanup instructions
* 📝 Knowledge-check questions

The documentation is intended to be useful for both **beginners** and learners revising AWS concepts through hands-on practice.

---

# 🤝 Contribution Guidelines

Contributions and documentation improvements are welcome.

For repository contributions, use a dedicated branch and submit a Pull Request for review.

### Recommended Workflow

```text
Create / Update Lab
        ↓
Create Dedicated Branch
        ↓
Make Changes
        ↓
Commit Changes
        ↓
Push Branch
        ↓
Open Pull Request
        ↓
Review
        ↓
Address Feedback
        ↓
Approval
        ↓
Merge into main
```

### Example Branch Names

```text
lab-015
update-readme
improve-lab-012
fix-lab-014-documentation
```

### Example Commit Messages

```text
Add Lab 015
Update repository README
Improve Lab 014 documentation
Fix NAT Gateway instructions
```

Pull Requests should clearly explain:

* What was changed
* Why the change was made
* Which lab or documentation is affected
* What was tested
* Any cost considerations
* Any additional review required

---

# 👨‍💻 Contributors

This repository is maintained as part of the **MetaPi PSEB Training Program**.

### Repository Owner

**Muhammad Afaq Nasir**
GitHub: [@Afaq-DevOps](https://github.com/Afaq-DevOps)

### Contributor

**Mustansar Maqsood**
GitHub: [@Mustansar60](https://github.com/Mustansar60)

Contributors help improve the labs, documentation, architecture explanations, and hands-on learning experience.

---

# 🎓 Who Is This Roadmap For?

This roadmap is suitable for:

* AWS beginners
* Cloud Computing students
* Software Engineering students
* DevOps learners
* IT professionals starting with AWS
* Learners preparing for AWS foundational certifications
* Students participating in structured cloud training
* Anyone who prefers learning AWS through practical hands-on exercises

---

# 📈 Current Progress

```text
Labs Completed: 14
Labs Planned:   30
Current Stage:  Multi-AZ Networking
```

### Current Learning Stage

```text
✅ AWS Fundamentals
✅ EC2
✅ Web Servers
✅ Automation
✅ EBS
✅ Backup & Recovery
✅ IAM
✅ S3
✅ Application Deployment
✅ VPC
✅ NAT Gateway
✅ Network Security
✅ Multi-AZ Architecture

🔜 Additional AWS & DevOps Labs
```

---

# 🌟 Learning Mindset

The purpose of this roadmap is not simply to complete 30 labs.

The real objective is to develop the ability to:

```text
Understand
    ↓
Design
    ↓
Build
    ↓
Test
    ↓
Troubleshoot
    ↓
Secure
    ↓
Optimize
    ↓
Explain
```

AWS infrastructure with confidence.

---

# 🚀 What's Next?

More hands-on AWS and DevOps labs will be added progressively as the training roadmap continues.

The repository will be updated as new labs are completed and reviewed.

---

## ☁️ Keep Building. Keep Learning.

> **The best way to learn cloud computing is to build real infrastructure, understand how it works, and learn how to troubleshoot it when it breaks.**

**Happy Learning & Happy Cloud Building! 🚀**

---

### 📌 MetaPi PSEB Training Program

**AWS Hands-On Labs Roadmap**

Built with a hands-on learning mindset for developing practical AWS, Cloud, and DevOps skills.
