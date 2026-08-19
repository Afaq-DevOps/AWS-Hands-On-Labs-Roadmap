# **Lab 04 — Deploy an HTML Website Using Nginx**

### **Step 1: Launch an Ubuntu EC2 Instance**

1. Log in to the **AWS Management Console**.
2. Navigate to **EC2 → Instances → Launch instances**.
3. Configure:
   - **Name:** `MetaPi-Lab04-Nginx`
   - **AMI:** Ubuntu Server LTS
   - **Instance type:** Use the small instance type approved for your training account.
   - **Key pair:** Select or create a key pair.
4. Create a Security Group named:

   ```text
   MetaPi-Lab04-Nginx-SG
   ```

5. Add inbound rules:
   - **SSH (22):** My IP
   - **HTTP (80):** Anywhere IPv4
6. Launch the instance.

---

### **Step 2: Connect to the EC2 Instance**

1. Copy the **Public IPv4 address**.
2. Connect using SSH:

   ```bash
   ssh -i "your-key.pem" ubuntu@<INSTANCE_PUBLIC_IP>
   ```

3. Update the system:

   ```bash
   sudo apt update
   sudo apt upgrade -y
   ```

---

### **Step 3: Install Nginx**

1. Install Nginx:

   ```bash
   sudo apt install nginx -y
   ```

2. Start Nginx:

   ```bash
   sudo systemctl start nginx
   ```

3. Enable it at boot:

   ```bash
   sudo systemctl enable nginx
   ```

4. Check the service:

   ```bash
   sudo systemctl status nginx
   ```

5. Confirm:

   ```text
   Active: active (running)
   ```

---

### **Step 4: Test the Default Nginx Page**

1. Open:

   ```text
   http://<INSTANCE_PUBLIC_IP>
   ```

2. You should see the default Nginx page.
3. If it does not load, verify:
   - Nginx is running.
   - Port 80 is open in the Security Group.
   - The EC2 instance has a public IPv4 address.

---

### **Step 5: Create the MetaPi Website**

1. Replace the default page:

   ```bash
   sudo nano /var/www/html/index.html
   ```

2. Add:

   ```html
   <!DOCTYPE html>
   <html lang="en">
   <head>
       <meta charset="UTF-8">
       <meta name="viewport" content="width=device-width, initial-scale=1.0">
       <title>MetaPi Nginx Lab</title>
       <style>
           body { font-family: Arial, sans-serif; margin: 0; background: #f4f6f8; text-align: center; }
           header { background: #232f3e; color: white; padding: 30px; }
           main { padding: 50px; }
           .card { max-width: 720px; margin: auto; background: white; padding: 30px; border-radius: 10px; }
       </style>
   </head>
   <body>
       <header><h1>MetaPi PSEB Training Program</h1></header>
       <main>
           <div class="card">
               <h2>AWS Solutions Architect Associate</h2>
               <h3>Lab 04 — Nginx on Amazon EC2</h3>
               <p>This website is being served by Nginx.</p>
           </div>
       </main>
   </body>
   </html>
   ```

3. Save and exit Nano.

---

### **Step 6: Test the Custom Website**

1. Reload Nginx:

   ```bash
   sudo nginx -t
   sudo systemctl reload nginx
   ```

2. Refresh:

   ```text
   http://<INSTANCE_PUBLIC_IP>
   ```

3. Verify that the MetaPi page appears.

---

### **Step 7: Check Nginx Logs**

1. View access logs:

   ```bash
   sudo tail /var/log/nginx/access.log
   ```

2. Refresh the site and check again.
3. View errors:

   ```bash
   sudo tail /var/log/nginx/error.log
   ```

---

### **Step 8: Test Service Failure and Recovery**

1. Stop Nginx:

   ```bash
   sudo systemctl stop nginx
   ```

2. Refresh the site. It should fail.
3. Start Nginx again:

   ```bash
   sudo systemctl start nginx
   ```

4. Refresh the site and confirm that it works.

---

### **Step 9: Compare Apache and Nginx**

Students should now recognize that both Apache and Nginx can listen on TCP port 80 and serve website files from an EC2 instance.

```text
Browser
   |
   v
Security Group :80
   |
   v
EC2
   |
   v
Nginx
   |
   v
/var/www/html/index.html
```

---

### **Step 10: Secure and Clean Up**

1. Keep SSH limited to **My IP**.
2. Disconnect:

   ```bash
   exit
   ```

3. Terminate `MetaPi-Lab04-Nginx` when the lab is complete.
4. Delete any unused Security Group after the instance is terminated.

---

### **Lab Completed**

You have deployed and tested a website on Amazon EC2 using **Nginx**.

**Next:** **Lab 05 — Automate EC2 Web Server Deployment Using User Data**
