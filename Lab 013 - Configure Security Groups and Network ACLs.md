# Lab 13 — Configure Security Groups and Network ACLs
## Lab Objective

In this lab, you will learn and demonstrate the difference between:
- **Security Groups (SGs)** — instance-level, stateful firewall
- **Network ACLs (NACLs)** — subnet-level, stateless firewall

**By the end of the lab, you will be able to:**
- Launch a web server inside a custom VPC and public subnet.
- Configure a dedicated Security Group.
- Allow SSH only from your public IP.
- Allow HTTP from the internet.
- Test the effect of removing and restoring an SG rule.
- Create and configure a custom Network ACL.
- Configure inbound and outbound NACL rules.
- Associate a NACL with a public subnet.
- Block a specific client IP using a NACL deny rule.
- Observe the difference between Security Groups and NACLs.
- Safely clean up all resources created for the lab.
---
# Architecture

```text

                         Internet

                            |

                            v

                    Network ACL

                  Subnet Boundary

                            |

                            v

                 Public Subnet

              10.10.1.0/24

                            |

                            v

                   Security Group

                 Instance Boundary

                            |

                            v

                MetaPi-Lab13-Web

                   EC2 Instance

                     Apache

                    HTTP :80

                    SSH  :22

```
---
# Prerequisites
Before starting, make sure the following resources already exist:
- VPC: `MetaPi-VPC`
- Public Subnet: `MetaPi-Public-Subnet-A`
- Region: `eu-north-1`
- A working AWS account
- Your current public IPv4 address

**Your public IP should be written as:**

```text

<YOUR_PUBLIC_IP>/32

```

  Example:
  
```text

103.111.39.132/32

```

> **Important:** Your public IP may change. Do not blindly copy the example IP. Always use the current IP shown by AWS when selecting **My IP**.

  
---
# Step 1 — Launch the Test Web Server
Go to:
**AWS Console → EC2 → Instances → Launch instance**
Configure the instance as follows.
## Name
```text

MetaPi-Lab13-Web

```
## AMI
Use:
```text

Ubuntu Server LTS

```
## Instance type
Use a free-tier eligible instance type available in your account, for example:
```text

t3.micro

```
## Key pair
Select an existing key pair that you can use for SSH.
If your training environment already has a designated key pair, use that one.
## Network settings
Select:
```text

VPC:

MetaPi-VPC

```
Select:
```text

Subnet:

MetaPi-Public-Subnet-A

```
Enable:
```text

Auto-assign Public IP:

Enable

```
## Security Group
Create a new Security Group.
Name:
```text

MetaPi-Lab13-Web-SG

```
Description:
```text

Security group for Lab 13 web server

```
Add the following inbound rules:
| Type | Protocol | Port | Source | Purpose |
|---|---|---:|---|---|
| SSH | TCP | 22 | My IP | SSH administration |
| HTTP | TCP | 80 | Anywhere-IPv4 | Public web access |

For SSH, select:
```text

My IP

```
This should create a source similar to:
```text

YOUR_PUBLIC_IP/32

```
For HTTP, select:
```text

Anywhere-IPv4

```
which means:
```text

0.0.0.0/0

```
Keep the default outbound rule:
```text

All traffic → 0.0.0.0/0

```

---
# Step 2 — Add User Data to Install Apache
In **Advanced details**, find **User data** and paste:
```bash

#!/bin/bash

apt-get update -y

apt-get install -y apache2

echo "<h1>MetaPi Lab 13 - Security Groups and NACLs</h1>" > /var/www/html/index.html

systemctl enable --now apache2

```
Launch the instance.
Wait until:
```text

Instance state = Running

```
and:
```text

Status checks = 2/2 checks passed

```
---
# Step 3 — Verify the Web Server
Open the EC2 instance details and copy its:
```text

Public IPv4 address

```
For example:
```text

13.62.226.149

```
Open:
```text

http://<PUBLIC_IP>

```
Example:
```text

http://13.62.226.149/

```
Expected result:
```text

MetaPi Lab 13 - Security Groups and NACLs

```
## Important
At this point, the website should load successfully.
If it does not load:
1. Confirm the instance is running.
2. Confirm the public IPv4 address exists.
3. Confirm the instance is in `MetaPi-Public-Subnet-A`.
4. Confirm the route table for the public subnet has a route to an Internet Gateway.
5. Confirm the Security Group allows TCP 80.
6. Confirm Apache was installed by User Data.
---
# Step 4 — Verify SSH Access
Use SSH from your machine:
```bash

ssh -i <YOUR_KEY.pem> ubuntu@<PUBLIC_IP>

```
Example:
```bash

ssh -i my-key.pem ubuntu@13.62.226.149

```
SSH should work from the IP allowed in the Security Group.
The important concept is:
```text

SSH 22 → My IP only

HTTP 80 → Internet

```
You do not need to expose SSH to:
```text

0.0.0.0/0

```
---
# Step 5 — Test a Security Group Change
Now we will deliberately break the website.
Go to:
**EC2 → Security Groups → `MetaPi-Lab13-Web-SG`**
Open:
**Inbound rules → Edit inbound rules**
Find:
```text

HTTP

TCP

80

0.0.0.0/0

```
Remove this rule.
Save the changes.
Now refresh:
```text

http://<PUBLIC_IP>

```
## Expected result
The website should no longer be reachable.
This demonstrates that the Security Group controls traffic at the EC2 instance level.

---
# Step 6 — Restore the Security Group Rule
Go back to:
**MetaPi-Lab13-Web-SG → Inbound rules → Edit inbound rules**
Add:
```text

Type: HTTP

Protocol: TCP

Port: 80

Source: 0.0.0.0/0

```
Save the changes.
Refresh:
```text

http://<PUBLIC_IP>

```
## Expected result
The website should load again.
You have now demonstrated:
```text

Security Group HTTP ALLOW

        ↓

Website works

  

Security Group HTTP removed

        ↓

Website blocked

  

Security Group HTTP restored

        ↓

Website works again

```

---
# Step 7 — Create the Custom Network ACL
Go to:
**VPC → Network ACLs**
Click:
**Create network ACL**
Configure:
```text

Name:

MetaPi-Lab13-NACL

  

VPC:

MetaPi-VPC

```
Click:
```text

Create network ACL

```
## Important
A newly created custom NACL starts with no useful allow rules. Its implicit/default final behavior is deny.
Therefore:
**Do not associate the new NACL with the public subnet until the required rules have been configured.**
This prevents accidentally cutting off access to the subnet.

---
# Step 8 — Configure NACL Inbound Rules
Open:
```text

MetaPi-Lab13-NACL

```
Go to:
```text

Inbound rules

```
Click:
```text

Edit inbound rules

```
Add these rules:
| Rule | Type | Protocol | Port | Source | Action |
|---:|---|---|---|---|---|
| 100 | SSH | TCP | 22 | `<YOUR_PUBLIC_IP>/32` | ALLOW |
| 110 | HTTP | TCP | 80 | `0.0.0.0/0` | ALLOW |
| 120 | Custom TCP | TCP | `1024-65535` | `0.0.0.0/0` | ALLOW |
| * | All traffic | All | All | `0.0.0.0/0` | DENY |
Example:  
```text

Rule 100

SSH

TCP

22

103.111.39.132/32

ALLOW

```

```text

Rule 110

HTTP

TCP

80

0.0.0.0/0

ALLOW

```

```text

Rule 120

Custom TCP

TCP

1024-65535

0.0.0.0/0

ALLOW

```
Keep:
```text

* → DENY

```
## Why is rule 120 required?
Network ACLs are **stateless**.
Unlike Security Groups, a NACL does not automatically remember that a connection was allowed.
TCP connections use temporary/ephemeral ports for return traffic. Therefore, the NACL must allow the required ephemeral port range.
Save the rules.

---
# Step 9 — Configure NACL Outbound Rules
Open:
```text

Outbound rules

```
Click:
```text

Edit outbound rules

```
Add:
| Rule | Type | Protocol | Port | Destination | Action |
|---:|---|---|---|---|---|
| 100 | All traffic | All | All | `0.0.0.0/0` | ALLOW |
| * | All traffic | All | All | `0.0.0.0/0` | DENY |
Save the changes.
The final configuration should look like:
```text

Outbound:


  

100 → All traffic → 0.0.0.0/0 → ALLOW

*   → All traffic → 0.0.0.0/0 → DENY

```
---
# Step 10 — Associate the Custom NACL with the Public Subnet
Now that the NACL has its rules, associate it with:
```text

MetaPi-Public-Subnet-A

```
Go to:
**MetaPi-Lab13-NACL → Subnet associations**
Click:
```text

Edit subnet associations

```
Select:
```text

MetaPi-Public-Subnet-A

```
Save the changes.
Expected result:
```text

MetaPi-Public-Subnet-A

        ↓

MetaPi-Lab13-NACL

```
## Important
A subnet can be associated with only one NACL at a time.
When you associate the custom NACL with the subnet, it replaces the previous NACL association for that subnet.

---
# Step 11 — Test the Website Through the Custom NACL
Open:
```text

http://<PUBLIC_IP>

```
Refresh the page.
## Expected result
The website should still load.
This confirms that:
- Security Group allows HTTP.
- NACL allows HTTP.
- NACL outbound traffic is allowed.
- The subnet association is working.
Also test SSH again if required:
```bash

ssh -i <YOUR_KEY.pem> ubuntu@<PUBLIC_IP>

```

---
# Step 12 — Add a Temporary NACL DENY Rule
Now we will demonstrate the major advantage of NACLs:
**NACLs support explicit DENY rules.**
Go to:
**MetaPi-Lab13-NACL → Inbound rules → Edit inbound rules**
Add this rule:
| Rule | Type | Protocol | Port | Source | Action |
|---:|---|---|---:|---|---|
| 90 | HTTP | TCP | 80 | `<YOUR_PUBLIC_IP>/32` | DENY |
Example:
```text

Rule:90
Type:HTTP
Protocol:TCP
Port:80
Source: 103.111.39.132/32 (Go and Search "What My IP" that will be your source here)
Action:DENY

```
Save the changes.
---
# Step 13 — Test the NACL DENY Rule
Refresh:
```text

http://<PUBLIC_IP>

```
## Expected result
The website should stop loading from your current public IP.
Why?
The rules are evaluated in numerical order.
The request matches:
```text

Rule 90 → HTTP → YOUR_IP/32 → DENY

```
Therefore, AWS stops evaluating the request before it reaches:
```text

Rule 110 → HTTP → 0.0.0.0/0 → ALLOW

```
The lower rule number wins because it is evaluated first.
The Security Group can still contain:
```text

HTTP 80 → 0.0.0.0/0 → ALLOW

```
but the NACL blocks the request before it reaches the instance.

---
# Step 14 — Remove the Temporary DENY Rule
Go back to:
**MetaPi-Lab13-NACL → Inbound rules → Edit inbound rules**
Find:
```text

Rule:90

HTTP:80

<YOUR_PUBLIC_IP>/32

DENY

```
Click:
```text

Remove

```
Save the changes.
Your inbound rules should return to:
```text

100 → SSH → 22 → YOUR_IP/32 → ALLOW
110 → HTTP → 80 → 0.0.0.0/0 → ALLOW
120 → Custom TCP → 1024-65535 → 0.0.0.0/0 → ALLOW
*   → All traffic → 0.0.0.0/0 → DENY
```
---
# Step 15 — Verify the Website Again
Refresh:
```text

http://<PUBLIC_IP>

```
## Expected result
The website should load again.
You have now successfully demonstrated:
```text

NACL Rule 90 DENY

        ↓

Website blocked

  

Remove Rule 90

        ↓

Rule 110 ALLOW

        ↓

Website works again

```
---
# Step 16 — Understand Security Group vs NACL
## Security Group
Security Groups operate at the:
```text

EC2 Instance / ENI level

```
Characteristics:
- Stateful
- Allow rules only
- No explicit deny rules
- Return traffic is automatically allowed for an established connection
- Rules apply to the network interface/instance
Example:
```text

HTTP 80 → ALLOW

SSH 22 → ALLOW

```

---
## Network ACL
Network ACLs operate at the:
```text

Subnet level

```
Characteristics:
- Stateless
- Support ALLOW and DENY
- Rules are evaluated in numerical order
- Return traffic must be explicitly allowed
- One NACL can be associated with multiple subnets
Example:
```text

Rule 90  → DENY

Rule 100 → ALLOW

Rule 110 → ALLOW

```
Rule 90 is evaluated first.

---
# Step 17 — Final Comparison
```text

                         INTERNET

                            |

                            v

                 +----------------------+

                 |     NETWORK ACL      |

                 |    SUBNET LEVEL      |

                 |  ALLOW / DENY        |

                 |     STATELESS        |

                 +----------------------+

                            |

                            v

                 +----------------------+

                 |    PUBLIC SUBNET     |

                 |    10.10.1.0/24      |

                 +----------------------+

                            |

                            v

                 +----------------------+

                 |   SECURITY GROUP     |

                 |   INSTANCE LEVEL     |

                 |   ALLOW ONLY         |

                 |     STATEFUL         |

                 +----------------------+

                            |

                            v

                 +----------------------+

                 |       EC2            |

                 | MetaPi-Lab13-Web     |

                 |     Apache :80       |

                 +----------------------+

```
---
# Step 18 — Lab Verification Checklist
Before marking the lab complete, verify each item.
## EC2
- [ ] Instance was launched in `MetaPi-VPC`
- [ ] Instance was launched in `MetaPi-Public-Subnet-A`
- [ ] Public IPv4 was enabled
- [ ] Apache was installed using User Data
- [ ] Website loaded successfully
## Security Group
- [ ] `MetaPi-Lab13-Web-SG` was created
- [ ] SSH 22 allows only My IP
- [ ] HTTP 80 allows `0.0.0.0/0`
- [ ] Removing HTTP 80 blocked the website
- [ ] Restoring HTTP 80 made the website work again
## Network ACL
- [ ] `MetaPi-Lab13-NACL` was created
- [ ] Inbound SSH rule 100 was configured
- [ ] Inbound HTTP rule 110 was configured
- [ ] Inbound ephemeral port rule 120 was configured
- [ ] Final `* DENY` rule was preserved
- [ ] Outbound rule 100 allows all traffic
- [ ] Final outbound `* DENY` rule was preserved
- [ ] Public subnet was associated with the custom NACL
- [ ] Website worked after NACL association
- [ ] Rule 90 DENY blocked the website
- [ ] Rule 90 was removed
- [ ] Website worked again
## Concepts
- [ ] Can explain stateful Security Groups
- [ ] Can explain stateless NACLs
- [ ] Can explain NACL rule ordering
- [ ] Can explain explicit DENY in NACLs
- [ ] Can explain why ephemeral ports are needed
---
# Step 19 — Troubleshooting Guide
## Problem: Website does not load after launching EC2
Check:
```text

EC2 = Running

Status checks = 2/2

Public IPv4 = Present

Security Group = HTTP 80 allowed

Subnet = MetaPi-Public-Subnet-A

Route table = Internet Gateway route exists

```

---
## Problem: Website stopped after removing the Security Group HTTP rule
This is expected.
Restore:
```text

HTTP

TCP

80

0.0.0.0/0

ALLOW

```
---
## Problem: Website stopped after associating the custom NACL
Check the NACL inbound rules:
```text

110 → HTTP 80 → 0.0.0.0/0 → ALLOW

```
Check the outbound rule:
```text

100 → All traffic → 0.0.0.0/0 → ALLOW

```
Check that the final `* DENY` rule remains below the allow rules.

---
## Problem: Website does not return after removing Rule 90
Confirm Rule 90 was actually deleted and saved.
The inbound rules should contain:
```text

100 → SSH

110 → HTTP

120 → Ephemeral ports

*   → DENY

```
There should be no:
```text

90 → HTTP → YOUR_IP → DENY

```
---
## Problem: SSH stops working
Check:
```text

Security Group:

SSH 22 → YOUR_PUBLIC_IP/32 → ALLOW

```
and:
```text

NACL:

Rule 100 → SSH 22 → YOUR_PUBLIC_IP/32 → ALLOW

```
Also verify the NACL outbound rule allows return traffic.
Remember that your public IP can change.

---
# Step 20 — Clean Up the Lab

> **Do cleanup carefully. Do not delete resources that belong to Lab 11, Lab 12, or your main VPC architecture.**
## 20.1 Restore the Original NACL
Before deleting the custom NACL:
Go to:
**VPC → Network ACLs**
Open:
```text

MetaPi-Lab13-NACL

```
Go to:
```text

Subnet associations

```
Click:
```text

Edit subnet associations

```
For:
```text

MetaPi-Public-Subnet-A

```
select the original/default NACL associated with `MetaPi-VPC`.
In this lab environment, the original NACL was:
```text

acl-0e7cf11857b23abe2

```
After saving, verify:
```text

MetaPi-Public-Subnet-A

        ↓

original/default NACL

```
Only after this verification should you delete the custom NACL.

---
## 20.2 Delete the Custom NACL
Delete:
```text

MetaPi-Lab13-NACL

```
Expected result:
```text

acl-03dee6a53bae8fb3c

```
is deleted.

---
## 20.3 Terminate the Lab EC2 Instance
Go to:
**EC2 → Instances**
Select:
```text

MetaPi-Lab13-Web

```
Choose:
```text

Instance state → Terminate instance

```
Wait until:
```text

Terminated

```
  ---
## 20.4 Delete the Lab Security Group
After the EC2 instance is terminated, go to:
**EC2 → Security Groups**
Delete:
```text

MetaPi-Lab13-Web-SG

```
If AWS says the Security Group is still in use, check whether it is attached to another network interface/resource before deleting it.

---
## 20.5 Check for Accidental Launch-Wizard Security Groups
During hands-on work, AWS may create Security Groups such as:
```text

launch-wizard-1

launch-wizard-2

launch-wizard-3

```
These are not required by this lab.
However:
**Do not automatically delete them.**
First verify:
1. They are not attached to any running EC2 instance.
2. They are not attached to another resource you still need.
3. They do not belong to Lab 11 or Lab 12.
Only then delete an unused launch-wizard Security Group.
---
# Final Lab State
After cleanup, the following Lab 13 resources should no longer exist:
```text

MetaPi-Lab13-Web

MetaPi-Lab13-Web-SG

MetaPi-Lab13-NACL

```
The following shared infrastructure should remain:
```text

MetaPi-VPC

MetaPi-Public-Subnet-A

MetaPi-Private-Subnet-A

Original/default NACL

Lab 12 resources

```
---
# Lab 13 Completed
You have successfully performed a complete hands-on demonstration of:
```text

Security Groups

       +

Network ACLs

       +

Allow rules

       +

Deny rules

       +

Rule ordering

       +

Stateful filtering

       +

Stateless filtering

       +

Subnet-level security

       +

Instance-level security

```
### Golden Rule
```text

NACL = Subnet-level filter

SG   = Instance-level filter

```
And remember:
```text

NACL → Stateless → ALLOW + DENY

SG   → Stateful  → ALLOW only

```
---
# Next Lab
**Lab 14 — Build a Multi-AZ VPC Architecture**