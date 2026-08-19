# **Lab 05 — Automate EC2 Web Server Deployment Using User Data**

### **Step 1: Open the EC2 Launch Wizard**

1. Go to **EC2 → Instances → Launch instances**.
2. Configure:
   - **Name:** `MetaPi-Lab05-UserData`
   - **AMI:** Ubuntu Server LTS
   - **Instance type:** Use the approved small training instance.
   - **Key pair:** Select your lab key.
3. Create a Security Group:
   - **SSH (22):** My IP
   - **HTTP (80):** Anywhere IPv4

---

### **Step 2: Add User Data**

1. Expand **Advanced details**.
2. Find **User data**.
3. Paste:

   ```bash
   #!/bin/bash
   apt-get update -y
   apt-get install -y apache2
   systemctl enable apache2
   systemctl start apache2

   cat > /var/www/html/index.html <<'EOF'
   <!DOCTYPE html>
   <html>
   <head>
       <title>MetaPi User Data Lab</title>
   </head>
   <body style="font-family:Arial;text-align:center;margin-top:60px;">
       <h1>MetaPi PSEB Training Program</h1>
       <h2>Lab 05 — EC2 User Data</h2>
       <p>Apache and this page were installed automatically at first boot.</p>
   </body>
   </html>
   EOF
   ```

4. Launch the instance.

---

### **Step 3: Test Automatic Deployment**

1. Wait for the instance to reach **Running**.
2. Wait a few minutes for bootstrapping to complete.
3. Open:

   ```text
   http://<INSTANCE_PUBLIC_IP>
   ```

4. Verify that the MetaPi User Data page appears without manually installing Apache.

---

### **Step 4: Connect and Verify Apache**

1. SSH to the instance:

   ```bash
   ssh -i "your-key.pem" ubuntu@<INSTANCE_PUBLIC_IP>
   ```

2. Verify Apache:

   ```bash
   sudo systemctl status apache2
   ```

3. Verify the generated page:

   ```bash
   cat /var/www/html/index.html
   ```

---

### **Step 5: Inspect Cloud-Init Logs**

Ubuntu processes EC2 user data through cloud-init.

1. View the user-data output log:

   ```bash
   sudo less /var/log/cloud-init-output.log
   ```

2. Search for Apache installation or errors.
3. Press `q` to exit.

---

### **Step 6: Confirm User Data**

Query the instance metadata service for the original user data:

```bash
TOKEN=$(curl -sS -X PUT -H "X-aws-ec2-metadata-token-ttl-seconds: 21600" \
http://169.254.169.254/latest/api/token)

curl -sS -H "X-aws-ec2-metadata-token: $TOKEN" \
http://169.254.169.254/latest/user-data
```

You should see the script that was supplied at launch.

---

### **Step 7: Create a Second Automated Instance**

1. Launch another EC2 instance.
2. Use the same User Data script.
3. Name it:

   ```text
   MetaPi-Lab05-UserData-02
   ```

4. Open its public IP in a browser.
5. Verify that the same application was deployed automatically.

This demonstrates repeatable server bootstrapping.

---

### **Step 8: Understand the Automation Flow**

```text
Launch EC2
   |
   v
First Boot
   |
   v
User Data
   |
   +--> apt update
   +--> install Apache
   +--> start service
   +--> create index.html
   |
   v
Website Ready
```

---

### **Step 9: Lab Verification**

Verify that you can:

- Add User Data during EC2 launch.
- Automatically install software.
- Automatically create application files.
- Check cloud-init output.
- Reproduce the deployment on another instance.

---

### **Step 10: Secure and Clean Up**

1. Keep SSH restricted to your IP.
2. Terminate both Lab 05 EC2 instances.
3. Remove unused Security Groups.

---

### **Lab Completed**

You have automated an EC2 web-server deployment using **EC2 User Data**.

**Next:** **Lab 06 — Create, Attach, Format, and Mount an EBS Volume**
