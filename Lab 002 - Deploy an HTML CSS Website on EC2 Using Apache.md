# **Lab 02 — Deploy an HTML/CSS Website on EC2 Using Apache**

### **Step 1: Launch an Ubuntu EC2 Instance**

1. Log in to the **AWS Management Console** and open the **EC2 Dashboard**.
2. Click **EC2 → Instances → Launch instances**.
3. Enter the instance name:

   ```
   MetaPi-Lab02-WebServer
   ```

4. Select **Ubuntu Server LTS**.
5. Select the small instance type approved for the training account. Example: `t3.micro`.
6. Select the key pair created in Lab 01 or create a new one.

---

### **Step 2: Configure the Security Group**

Create:

```text
MetaPi-Lab02-Web-SG
```

Inbound rules:

- **SSH / TCP 22 / My IP**
- **HTTP / TCP 80 / Anywhere IPv4 (`0.0.0.0/0`)**

Click **Launch instance**.

---

### **Step 3: Connect to the EC2 Instance**

Copy the Public IPv4 address and connect:

```bash
chmod 400 metapi-lab02-key.pem
ssh -i "metapi-lab02-key.pem" ubuntu@<INSTANCE_PUBLIC_IP>
```

---

### **Step 4: Update the Ubuntu System**

```bash
sudo apt update
sudo apt upgrade -y
```

---

### **Step 5: Install Apache Web Server**

```bash
sudo apt install apache2 -y
sudo systemctl start apache2
sudo systemctl enable apache2
sudo systemctl status apache2
```

You should see:

```text
Active: active (running)
```

Press `q` to exit the status screen.

---

### **Step 6: Test the Apache Web Server**

Open:

```text
http://<INSTANCE_PUBLIC_IP>
```

You should see the Apache default page.

---

### **Step 7: Check the Apache Web Directory**

```bash
ls -l /var/www/html/
```

Apache serves files from:

```text
/var/www/html/
```

---

### **Step 8: Create the MetaPi HTML Website**

Open the main page:

```bash
sudo nano /var/www/html/index.html
```

Replace its content with:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>MetaPi AWS Training</title>
    <style>
        body { font-family: Arial, sans-serif; background-color: #f4f4f4; margin: 0; padding: 0; }
        header { background-color: #232f3e; color: white; padding: 25px; text-align: center; }
        main { padding: 50px; text-align: center; }
        h1 { margin: 0; }
        h2 { color: #232f3e; }
        p { font-size: 18px; color: #555; }
        .lab-box { background-color: white; margin: 30px auto; padding: 25px; max-width: 700px; border-radius: 8px; }
        footer { background-color: #232f3e; color: white; padding: 15px; text-align: center; margin-top: 40px; }
    </style>
</head>
<body>
<header>
    <h1>MetaPi PSEB Training Program</h1>
</header>
<main>
    <div class="lab-box">
        <h2>AWS Solutions Architect Associate</h2>
        <p>Welcome to AWS Hands-On Lab 02.</p>
        <p>This website is running on an Amazon EC2 instance using the Apache Web Server.</p>
        <p>Congratulations! You successfully deployed your first website on AWS.</p>
    </div>
</main>
<footer>MetaPi AWS Training Program</footer>
</body>
</html>
```

Save with `CTRL + O`, `Enter`, then exit with `CTRL + X`.

---

### **Step 9: Check and Restart Apache**

```bash
cat /var/www/html/index.html
sudo systemctl restart apache2
sudo systemctl status apache2
```

---

### **Step 10: Test the MetaPi Website**

Open:

```text
http://<INSTANCE_PUBLIC_IP>
```

Verify the MetaPi training page appears.

---

### **Step 11: Understand the Request Flow**

```text
Your Computer
      |
      v
   Internet
      |
      v
EC2 Public IP
      |
      v
Security Group (Port 80)
      |
      v
Apache Web Server
      |
      v
/var/www/html/index.html
      |
      v
Website Displayed
```

---

### **Step 12: View Apache Logs**

Access log:

```bash
sudo tail /var/log/apache2/access.log
```

Error log:

```bash
sudo tail /var/log/apache2/error.log
```

Refresh the page and observe new access log entries.

---

### **Step 13: Check Listening Ports**

```bash
sudo ss -tulnp
```

Look for ports `22` and `80`.

---

### **Step 14: Test Apache Stop/Start**

```bash
sudo systemctl stop apache2
```

Refresh the website; it should fail to load. Then start Apache again:

```bash
sudo systemctl start apache2
```

Refresh the browser and verify the page works again.

---

### **Step 15: Verify the Security Group**

Confirm:

```text
SSH   TCP   22   Your-IP/32
HTTP  TCP   80   0.0.0.0/0
```

---

### **Step 16: Lab Verification**

Verify that you can:
- Launch Ubuntu EC2.
- Connect over SSH.
- Install and manage Apache.
- Allow HTTP through a Security Group.
- Create an HTML/CSS website.
- View Apache logs.
- Identify ports 22 and 80.

---

### **Step 17: Secure and Clean Up**

1. Keep SSH restricted to your IP.
2. Disconnect:

```bash
exit
```

3. Terminate `MetaPi-Lab02-WebServer` when finished.

---

### **Lab Completed**

You have completed **Lab 02 — Deploy an HTML/CSS Website on EC2 Using Apache**.

Next: **Lab 03 — Deploy a Website to EC2 Using SCP**
