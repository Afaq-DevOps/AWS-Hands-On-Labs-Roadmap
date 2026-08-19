# **Lab 01 — Launch Your First EC2 Instance and Connect Using SSH**

### **Step 1: Open the AWS EC2 Dashboard**

1. **Log in to the AWS Management Console.**
2. In the AWS search bar, search for:

   ```
   EC2
   ```

3. Open the **EC2 Dashboard**.
4. Make sure you are working in the AWS Region selected by your instructor.

---

### **Step 2: Launch an EC2 Instance**

1. Click **EC2 → Instances → Launch instances**.
2. Enter the following instance name:

   ```
   MetaPi-Lab01-EC2
   ```

3. Under **Application and OS Images (Amazon Machine Image)**, select **Ubuntu Server LTS**.
4. Under **Instance Type**, select the small instance type approved for your training account. Example:

   ```
   t3.micro
   ```

5. Under **Key Pair**, click **Create new key pair**.
6. Configure:
   - **Key pair name:** `metapi-lab01-key`
   - **Key pair type:** RSA
   - **Private key format:** `.pem`
7. Click **Create key pair**.
8. The `.pem` file will download to your computer.

> Keep your private key secure. Do not upload it to GitHub or share it with anyone.

---

### **Step 3: Configure the Security Group**

1. Under **Network Settings**, create a new Security Group.
2. Enter the Security Group name:

   ```
   MetaPi-Lab01-SG
   ```

3. Configure the following inbound rule:
   - **Type:** SSH
   - **Protocol:** TCP
   - **Port:** 22
   - **Source:** My IP
4. Do **not** configure SSH access from `0.0.0.0/0` unless specifically instructed for a temporary classroom exercise.
5. Review the instance configuration.
6. Click **Launch instance**.

---

### **Step 4: Wait for the EC2 Instance**

1. Go to **EC2 → Instances**.
2. Select `MetaPi-Lab01-EC2`.
3. Wait until **Instance State** becomes `Running`.
4. Wait until the **Status Checks** complete successfully.
5. Copy the **Public IPv4 address**.

---

### **Step 5: Prepare the SSH Key**

Open a terminal on your computer and navigate to the directory where your `.pem` file was downloaded.

```bash
cd ~/Downloads
chmod 400 metapi-lab01-key.pem
```

---

### **Step 6: Connect to the EC2 Instance Using SSH**

```bash
ssh -i "metapi-lab01-key.pem" ubuntu@<INSTANCE_PUBLIC_IP>
```

The first time you connect, type `yes` when asked to trust the host fingerprint.

If successful, you should see a prompt similar to:

```text
ubuntu@ip-10-0-0-10:~$
```

---

### **Step 7: Check the EC2 Instance Information**

Run:

```bash
hostname
whoami
cat /etc/os-release
hostname -I
uptime
free -h
df -h
```

Expected username:

```text
ubuntu
```

---

### **Step 8: Update the Ubuntu System**

```bash
sudo apt update
sudo apt upgrade -y
```

---

### **Step 9: Create Your First File on EC2**

```bash
mkdir metapi-lab01
cd metapi-lab01
echo "Welcome to MetaPi AWS Solutions Architect Training" > welcome.txt
cat welcome.txt
ls -l
```

Expected output:

```text
Welcome to MetaPi AWS Solutions Architect Training
```

---

### **Step 10: Check the EC2 Instance Private and Public IP**

Inside EC2:

```bash
hostname -I
```

Then go to **AWS Console → EC2 → Instances** and compare:
- **Private IPv4 address**
- **Public IPv4 address**

---

### **Step 11: Disconnect from the EC2 Instance**

```bash
exit
```

---

### **Step 12: Stop and Start the EC2 Instance**

1. Select the instance.
2. Click **Instance state → Stop instance**.
3. Wait until it is `Stopped`.
4. Click **Instance state → Start instance**.
5. Wait until it is `Running`.
6. Check the **Public IPv4 address** again.

> A normal public IPv4 address may change after stopping and starting an EC2 instance.

---

### **Step 13: Connect Again Using SSH**

```bash
ssh -i "metapi-lab01-key.pem" ubuntu@<NEW_PUBLIC_IP>
cd metapi-lab01
cat welcome.txt
```

The file should still exist after a normal stop/start.

---

### **Step 14: Test Security Group Access**

1. Go to **EC2 → Security Groups**.
2. Select `MetaPi-Lab01-SG`.
3. Confirm SSH is restricted to your IP, similar to:

```text
SSH   TCP   22   Your-IP/32
```

---

### **Step 15: Lab Verification**

Verify that you can:
- Launch an EC2 instance.
- Create and use a key pair.
- Configure a Security Group.
- Identify public and private IP addresses.
- Connect using SSH.
- Run basic Linux commands.
- Create files.
- Stop/start the instance and reconnect.

---

### **Step 16: Secure and Clean Up**

1. Disconnect:

```bash
exit
```

2. Go to **AWS Console → EC2 → Instances**.
3. Select `MetaPi-Lab01-EC2`.
4. Click **Instance state → Terminate instance**.
5. Confirm termination.
6. Keep or securely remove the `.pem` key according to instructor instructions.

---

### **Lab Completed**

You have completed **Lab 01 — Launch Your First EC2 Instance and Connect Using SSH**.

Next: **Lab 02 — Deploy an HTML/CSS Website on EC2 Using Apache**
