# **Lab 03 — Deploy a Website to EC2 Using SCP**

### **Step 1: Prepare the Website on Your Local Computer**

1. Create a folder named:

   ```text
   metapi-lab03
   ```

2. Inside it, create `index.html`.
3. Add:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>MetaPi AWS Lab 03</title>
    <style>
        body { font-family: Arial, sans-serif; background-color: #f4f6f8; margin: 0; text-align: center; }
        header { background-color: #232f3e; color: white; padding: 30px; }
        main { padding: 50px; }
        .card { background-color: white; max-width: 700px; margin: auto; padding: 30px; border-radius: 10px; }
        h2 { color: #232f3e; }
        footer { margin-top: 50px; background-color: #232f3e; color: white; padding: 15px; }
    </style>
</head>
<body>
<header><h1>MetaPi PSEB Training Program</h1></header>
<main>
    <div class="card">
        <h2>AWS Solutions Architect Associate</h2>
        <h3>Lab 03 — SCP Deployment</h3>
        <p>This website was created on a local computer and transferred to an Amazon EC2 instance using SCP.</p>
        <p>Congratulations! Your remote deployment is working.</p>
    </div>
</main>
<footer>MetaPi AWS Hands-On Training</footer>
</body>
</html>
```

---

### **Step 2: Verify the Website Locally**

Open `index.html` in a browser and verify the page works before uploading it.

---

### **Step 3: Launch an Ubuntu EC2 Instance**

1. Go to **EC2 → Instances → Launch instances**.
2. Name it:

   ```text
   MetaPi-Lab03-WebServer
   ```

3. Select **Ubuntu Server LTS**.
4. Select the approved training instance type. Example: `t3.micro`.
5. Select or create a key pair, e.g. `metapi-lab03-key`.

---

### **Step 4: Configure the Security Group**

Create:

```text
MetaPi-Lab03-Web-SG
```

Inbound rules:
- **SSH / TCP 22 / My IP**
- **HTTP / TCP 80 / Anywhere IPv4 (`0.0.0.0/0`)**

Launch the instance.

---

### **Step 5: Connect to the EC2 Instance**

```bash
chmod 400 metapi-lab03-key.pem
ssh -i "metapi-lab03-key.pem" ubuntu@<INSTANCE_PUBLIC_IP>
```

---

### **Step 6: Update EC2 and Install Apache**

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install apache2 -y
sudo systemctl start apache2
sudo systemctl enable apache2
sudo systemctl status apache2
```

---

### **Step 7: Test Apache**

Open:

```text
http://<INSTANCE_PUBLIC_IP>
```

You should see the Apache default page.

---

### **Step 8: Disconnect Before Using SCP**

```bash
exit
```

> Run SCP from your local computer when uploading local files to EC2.

---

### **Step 9: Understand the SCP Command**

Basic syntax:

```bash
scp -i <KEY_FILE> <LOCAL_FILE> <REMOTE_USER>@<REMOTE_IP>:<REMOTE_PATH>
```

Example:

```bash
scp -i "metapi-lab03-key.pem" index.html ubuntu@<INSTANCE_PUBLIC_IP>:/tmp/
```

SCP securely transfers files over SSH (TCP port 22).

---

### **Step 10: Upload index.html Using SCP**

Navigate to the website folder on your local computer and run:

```bash
scp -i "/path/to/metapi-lab03-key.pem" index.html ubuntu@<INSTANCE_PUBLIC_IP>:/tmp/
```

---

### **Step 11: Connect Back to EC2 and Verify the Upload**

```bash
ssh -i "metapi-lab03-key.pem" ubuntu@<INSTANCE_PUBLIC_IP>
ls -l /tmp/
```

You should see `index.html`.

---

### **Step 12: Move the Website into Apache's Web Directory**

```bash
sudo mv /tmp/index.html /var/www/html/index.html
sudo chmod 644 /var/www/html/index.html
ls -l /var/www/html/
sudo systemctl restart apache2
```

---

### **Step 13: Test the Uploaded Website**

Open:

```text
http://<INSTANCE_PUBLIC_IP>
```

Verify the MetaPi Lab 03 page appears.

---

### **Step 14: Upload a CSS File Separately**

On your local computer create `style.css`, then upload it:

```bash
scp -i "metapi-lab03-key.pem" style.css ubuntu@<INSTANCE_PUBLIC_IP>:/tmp/
```

Connect and move it:

```bash
sudo mv /tmp/style.css /var/www/html/
ls -l /var/www/html/
```

---

### **Step 15: Upload an Entire Website Directory**

Example local structure:

```text
website/
├── index.html
├── style.css
└── images/
```

Upload recursively:

```bash
scp -i "metapi-lab03-key.pem" -r website ubuntu@<INSTANCE_PUBLIC_IP>:/tmp/
```

Copy its contents into Apache's web directory:

```bash
sudo cp -r /tmp/website/* /var/www/html/
```

---

### **Step 16: Download a File from EC2 Using SCP**

On EC2:

```bash
echo "This file was created on EC2" > ec2-file.txt
exit
```

On your local computer:

```bash
scp -i "metapi-lab03-key.pem" ubuntu@<INSTANCE_PUBLIC_IP>:/home/ubuntu/ec2-file.txt .
cat ec2-file.txt
```

Expected output:

```text
This file was created on EC2
```

---

### **Step 17: Understand Upload and Download Direction**

Upload:

```text
Local Computer → SCP/SSH Port 22 → Amazon EC2
```

Download:

```text
Amazon EC2 → SCP/SSH Port 22 → Local Computer
```

---

### **Step 18: Troubleshoot SCP Errors**

If you see `Permission denied (publickey)`, check:
- Correct `.pem` key.
- Correct username (`ubuntu`).
- Security Group allows SSH from your current IP.
- Key permissions are correct:

```bash
chmod 400 metapi-lab03-key.pem
```

If the website does not load, check:

```bash
sudo systemctl status apache2
ls -l /var/www/html/
sudo ss -tulnp | grep :80
sudo tail /var/log/apache2/error.log
```

---

### **Step 19: Lab Verification**

Verify that you can:
- Launch EC2 and install Apache.
- Connect with SSH.
- Upload files with SCP.
- Upload directories with `scp -r`.
- Download files from EC2.
- Deploy files into `/var/www/html`.
- Troubleshoot basic SSH/SCP and Apache problems.

---

### **Step 20: Secure and Clean Up**

1. Verify SSH is restricted to `Your-IP/32`.
2. Disconnect:

```bash
exit
```

3. Terminate `MetaPi-Lab03-WebServer` when finished.
4. Keep local website files for later labs.

---

### **Lab Completed**

You have completed **Lab 03 — Deploy a Website to EC2 Using SCP**.

Next: **Lab 04 — Deploy an HTML Website Using Nginx**
