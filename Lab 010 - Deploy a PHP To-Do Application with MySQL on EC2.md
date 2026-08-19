# **Lab 10 — Deploy a PHP To-Do Application with MySQL on EC2**

### **Step 1: Launch an Ubuntu EC2 Instance**

1. Go to **EC2 → Instances → Launch instances**.
2. Configure:
   - **Name:** `MetaPi-Lab10-Todo`
   - **AMI:** Ubuntu Server LTS
   - **Instance type:** Approved small training instance
   - **Key pair:** Your lab key
3. Create a Security Group:
   - **SSH (22):** My IP
   - **HTTP (80):** Anywhere IPv4
4. Launch the instance.

---

### **Step 2: Connect and Update the Server**

Connect:

```bash
ssh -i "your-key.pem" ubuntu@<INSTANCE_PUBLIC_IP>
```

Update:

```bash
sudo apt update
sudo apt upgrade -y
```

---

### **Step 3: Install Apache, PHP, MySQL, and Git**

Install packages:

```bash
sudo apt install apache2 php libapache2-mod-php php-mysql mysql-server git -y
```

Enable Apache:

```bash
sudo systemctl enable --now apache2
```

Check:

```bash
sudo systemctl status apache2
sudo systemctl status mysql
```

---

### **Step 4: Create the Database**

Open MySQL:

```bash
sudo mysql
```

Create the database and application user:

```sql
CREATE DATABASE todo;
CREATE USER 'usertodo'@'localhost' IDENTIFIED BY '<STRONG_PASSWORD>';
GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, ALTER
ON todo.*
TO 'usertodo'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

> Replace `<STRONG_PASSWORD>` with a lab password that does not contain personal or organization-specific credentials.

---

### **Step 5: Clone the To-Do Application**

Clone the training repository used for this course:

```bash
cd /tmp
git clone https://github.com/ali-azgar-rakib/Todo-list-with-php.git
cd Todo-list-with-php
```

List files:

```bash
ls -la
```

---

### **Step 6: Import the Database Schema**

If `todo.sql` exists:

```bash
sudo mysql todo < todo.sql
```

Verify:

```bash
sudo mysql -e "USE todo; SHOW TABLES;"
```

---

### **Step 7: Find the Database Configuration**

Search the project for MySQL connection settings:

```bash
grep -RniE "localhost|mysqli|mysql|DB_HOST|DB_USER|DB_PASS" .
```

Open the relevant configuration file and update it to use:

```text
Database: todo
Username: usertodo
Password: <STRONG_PASSWORD>
Host: localhost
```

Save the file.

---

### **Step 8: Deploy the Application**

Remove the Apache default page:

```bash
sudo rm -f /var/www/html/index.html
```

Copy the application:

```bash
sudo cp -a /tmp/Todo-list-with-php/. /var/www/html/
```

Set ownership and permissions:

```bash
sudo chown -R www-data:www-data /var/www/html
sudo find /var/www/html -type d -exec chmod 755 {} \;
sudo find /var/www/html -type f -exec chmod 644 {} \;
```

Restart Apache:

```bash
sudo systemctl restart apache2
```

---

### **Step 9: Test the To-Do Application**

Open:

```text
http://<INSTANCE_PUBLIC_IP>
```

Test:

- Add a task.
- View tasks.
- Edit a task.
- Delete a task.

---

### **Step 10: Verify Database Records**

Open MySQL:

```bash
sudo mysql todo
```

List tables:

```sql
SHOW TABLES;
```

Use the appropriate table name from the output and inspect records:

```sql
SELECT * FROM <TABLE_NAME>;
```

Exit:

```sql
EXIT;
```

---

### **Step 11: Check Apache and PHP Errors**

Apache error log:

```bash
sudo tail -n 50 /var/log/apache2/error.log
```

PHP information:

```bash
php -v
```

MySQL connectivity:

```bash
mysql -u usertodo -p todo
```

---

### **Step 12: Understand the Current Architecture**

```text
User Browser
     |
     v
Public EC2
     |
     +--> Apache
     |
     +--> PHP Application
     |
     +--> MySQL
```

In this beginner lab, the web server and database run on the same EC2 instance.

Later, the database will be moved to Amazon RDS.

---

### **Step 13: Lab Verification**

Verify that you can:

- Install a LAMP-style application stack.
- Create a MySQL database and user.
- Import an SQL schema.
- Deploy a PHP application.
- Configure application database credentials.
- Perform CRUD operations in the browser.
- Troubleshoot Apache/MySQL errors.

---

### **Step 14: Secure and Clean Up**

1. Ensure SSH is limited to your IP.
2. Do not expose MySQL port 3306 publicly.
3. Do not store real production credentials in this lab application.
4. Terminate the instance when finished unless your instructor asks you to keep it for Lab 19.

---

### **Lab Completed**

You have deployed a working PHP + MySQL application on Amazon EC2.
