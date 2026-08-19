# **Lab 11 — Build a Custom VPC with Public and Private Subnets**

### **Step 1: Open the VPC Dashboard**

1. Log in to the AWS Console.
    
2. Search for **VPC**.
    
3. Open **Your VPCs**.
    
4. Click **Create VPC**.
    
5. Choose **VPC only**.
    

> **💡 Important:** In this lab, we will create **one VPC** and then create both a **public subnet** and a **private subnet inside the same VPC**.

---

### **Step 2: Create the VPC**

Configure:

- **Name:** `MetaPi-VPC`
    
- **IPv4 CIDR:** `10.10.0.0/16`
    
- **IPv6 CIDR:** No IPv6 CIDR block for this lab
    
- **Tenancy:** Default
    

Click **Create VPC**.

> **💡 Important:** `MetaPi-VPC` is the main network boundary for this lab. Both `MetaPi-Public-Subnet-A` and `MetaPi-Private-Subnet-A` will belong to this same VPC.

---

### **Step 3: Create the Public Subnet**

1. Go to **VPC → Subnets → Create subnet**.
    
2. Select `MetaPi-VPC`.
    
3. Configure:
    
    - **Subnet name:** `MetaPi-Public-Subnet-A`
        
    - **Availability Zone:** Choose one AZ
        
    - **IPv4 CIDR:** `10.10.1.0/24`
        
4. Create the subnet.
    

> **💡 Important:** A subnet is not automatically public just because its name contains **Public**. Later, we will associate it with a route table containing a route to the **Internet Gateway**.

---

### **Step 4: Create the Private Subnet**

1. Click **Create subnet**.
    
2. Select `MetaPi-VPC`.
    
3. Configure:
    
    - **Subnet name:** `MetaPi-Private-Subnet-A`
        
    - **Availability Zone:** Same AZ as the public subnet
        
    - **IPv4 CIDR:** `10.10.11.0/24`
        
4. Create the subnet.
    

> **💡 Important:** The private subnet is also inside `MetaPi-VPC`. We are **not creating a separate VPC** for the private subnet.

---

### **Step 5: Enable Public IPv4 Assignment for the Public Subnet**

1. Select `MetaPi-Public-Subnet-A`.
    
2. Choose **Actions → Edit subnet settings**.
    
3. Enable:
    
    ```text
    Auto-assign public IPv4 address
    ```
    
4. Save.
    

Do not enable this setting on the private subnet.

> **💡 Important:** Auto-assigning a public IPv4 address alone does **not** make a subnet public. The subnet must also have a route through an **Internet Gateway**. We will configure that route in the next steps.

---

### **Step 6: Create and Attach an Internet Gateway**

1. Go to **Internet gateways**.
    
2. Click **Create internet gateway**.
    
3. Name it:
    
    ```text
    MetaPi-IGW
    ```
    
4. Create it.
    
5. Choose **Actions → Attach to a VPC**.
    
6. Select `MetaPi-VPC`.
    
7. Attach.
    

> **💡 Important:** The Internet Gateway is attached to the **VPC**, not directly to an individual subnet.

---

### **Step 7: Create a Public Route Table**

1. Go to **Route tables → Create route table**.
    
2. Configure:
    
    - **Name:** `MetaPi-Public-RT`
        
    - **VPC:** `MetaPi-VPC`
        
3. Create it.
    
4. Open **Routes → Edit routes**.
    
5. Add:
    
    ```text
    Destination: 0.0.0.0/0
    Target: MetaPi-IGW
    ```
    
6. Save.
    

> **💡 Important:** Make sure the **VPC** is `MetaPi-VPC` and the target is **Internet Gateway → `MetaPi-IGW`**.
> 
> The route table is being configured here, but the public subnet will be associated with this route table in **Step 8**.

---

### **Step 8: Associate the Public Subnet**

1. Open `MetaPi-Public-RT`.
    
2. Go to **Subnet associations**.
    
3. Click **Edit subnet associations**.
    
4. Select `MetaPi-Public-Subnet-A`.
    
5. Save.
    

> **💡 Important:** This association connects the public subnet to the route table that contains:
> 
> ```text
> 0.0.0.0/0 → MetaPi-IGW
> ```
> 
> This routing path is what allows resources in the subnet to communicate with the Internet when the required public IP and security configuration are also present.

---

### **Step 9: Create a Private Route Table**

1. Create another route table:
    
    - **Name:** `MetaPi-Private-RT`
        
    - **VPC:** `MetaPi-VPC`
        
2. Do not add an Internet Gateway route.
    
3. Associate:
    
    ```text
    MetaPi-Private-Subnet-A
    ```
    

The private route table should currently contain only the local VPC route.

> **💡 Important:** Do not add:
> 
> ```text
> 0.0.0.0/0 → MetaPi-IGW
> ```
> 
> to the private route table. This lab intentionally keeps the private subnet without a direct Internet Gateway route.
> 
> **Note:** Internet access for the private subnet will be configured in the next lab using a **NAT Gateway**.

---

### **Step 10: Launch an EC2 Instance in the Public Subnet**

1. Go to **EC2 → Launch instances**.
    
2. Configure:
    
    - **Name:** `MetaPi-Lab11-Public-EC2`
        
    - **AMI:** Ubuntu Server LTS
        
    - **VPC:** `MetaPi-VPC`
        
    - **Subnet:** `MetaPi-Public-Subnet-A`
        
    - **Auto-assign public IP:** Enabled
        
3. Create a Security Group:
    
    - **SSH (22):** My IP
        
4. Launch the instance.
    

> **🔐 Security Note:** For this lab, use **My IP** for SSH access. Do not use `0.0.0.0/0` for SSH unless there is a specific controlled reason to do so.

---

### **Step 11: Test the Public Subnet**

1. Copy the public IPv4 address.
    
2. SSH:
    
    ```bash
    ssh -i "your-key.pem" ubuntu@<PUBLIC_IP>
    ```
    
3. Test outbound connectivity:
    
    ```bash
    curl -I https://aws.amazon.com
    ```
    
4. Exit:
    
    ```bash
    exit
    ```
    

> **💡 Expected Result:** The public EC2 should be reachable because it is in the public subnet, has a public IPv4 address, and its subnet uses a route table with:
> 
> ```text
> 0.0.0.0/0 → MetaPi-IGW
> ```

---

### **Step 12: Launch an EC2 Instance in the Private Subnet**

1. Launch another Ubuntu instance:
    
    - **Name:** `MetaPi-Lab11-Private-EC2`
        
    - **VPC:** `MetaPi-VPC`
        
    - **Subnet:** `MetaPi-Private-Subnet-A`
        
    - **Auto-assign public IP:** Disabled
        
2. Do not add a public HTTP rule.
    
3. Launch it.
    

Verify in the EC2 console that it has a **private IPv4 address** but no public IPv4 address.

> **⚠️ Important:** The private EC2 is intentionally configured **without a public IPv4 address** and is located in the private subnet.
> 
> Therefore, the normal **EC2 Instance Connect** browser option may not be available, and the **Connect** button may be disabled. **This is expected and does not mean the EC2 instance failed to launch.**
> 
> The instance can be accessed later using mechanisms such as **EC2 Instance Connect Endpoint**, **Systems Manager Session Manager**, or a **bastion host**, when the required configuration is provided.
> 
> **Do not make the private EC2 public just to make the Connect button work.** That would change the intended architecture of this lab.

---

### **Step 13: Review the Routing**

```text
Internet
   |
   v
MetaPi-IGW
   |
   v
MetaPi-Public-RT
   |
   v
10.10.1.0/24 Public Subnet
   |
   v
Public EC2
```

Private subnet:

```text
10.10.11.0/24 Private Subnet
   |
   v
MetaPi-Private-RT
   |
   +--> 10.10.0.0/16 local
   |
   X  No 0.0.0.0/0 Internet route
   |
   v
Private EC2
```

> **💡 Key Concept:**
> 
> A **public subnet** and **private subnet** can exist inside the **same VPC**.
> 
> In this lab:
> 
> ```text
> MetaPi-VPC
> │
> ├── MetaPi-Public-Subnet-A
> │       └── Public Route Table → IGW
> │
> └── MetaPi-Private-Subnet-A
>         └── Private Route Table → Local only
> ```
> 
> The subnet's routing determines whether it has a direct path to an Internet Gateway.

---

### **Step 14: Lab Verification**

Verify:

- VPC CIDR is `10.10.0.0/16`.
    
- Public subnet is `10.10.1.0/24`.
    
- Private subnet is `10.10.11.0/24`.
    
- Internet Gateway is attached.
    
- Public route table has `0.0.0.0/0 → IGW`.
    
- Private route table does not have an Internet Gateway route.
    
- Public EC2 has a public IP.
    
- Private EC2 does not.
    
- Public EC2 can be connected through the configured SSH/connection method.
    
- Private EC2 remains without a public IP.
    

> **⚠️ Expected Behavior:** Do not consider the private EC2's unavailable normal EC2 Instance Connect option to be a failure. The private EC2 is intentionally isolated from direct Internet access in this lab.

---

### **Step 15: Secure and Clean Up**

Keep the VPC resources if you will continue directly to Lab 12. Otherwise:

1. Terminate both EC2 instances.
    
2. Delete custom route tables after associations are removed.
    
3. Detach and delete the Internet Gateway.
    
4. Delete subnets.
    
5. Delete the VPC.
    

> **💡 Important:** If you are continuing to **Lab 12 — Configure NAT Gateway for a Private EC2 Instance**, keep the VPC, subnets, route tables, and other required resources instead of deleting them.

---

### **Lab Completed**

You have built a custom VPC with separate public and private network segments.

You should now understand:

- How a VPC contains multiple subnets.
    
- How public and private subnets can exist in the same VPC.
    
- How an Internet Gateway provides the VPC-level gateway to the Internet.
    
- How route tables determine the traffic path for each subnet.
    
- Why the public EC2 can have a public IP and direct Internet connectivity.
    
- Why the private EC2 intentionally has no public IP or direct Internet Gateway route.
    
- Why the normal EC2 Instance Connect option may not be available for the private EC2.
    
- How the private subnet will later use a **NAT Gateway** for outbound Internet connectivity.