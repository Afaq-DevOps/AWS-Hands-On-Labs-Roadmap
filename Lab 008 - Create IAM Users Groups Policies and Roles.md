# **Lab 08 — Create IAM Users Groups Policies and Roles**

### **Step 1: Open IAM**

1. Log in using the account provided for the training program.
2. Search for **IAM**.
3. Open the **IAM Dashboard**.

> Do not use the root user for normal lab administration.

---

### **Step 2: Create an IAM Group**

1. Go to **IAM → User groups**.
2. Click **Create group**.
3. Enter:

   ```text
   MetaPi-S3-ReadOnly-Group
   ```

4. Attach the AWS managed policy:

   ```text
   AmazonS3ReadOnlyAccess
   ```

5. Create the group.

---

### **Step 3: Create a Lab IAM User**

1. Go to **IAM → Users → Create user**.
2. Enter:

   ```text
   metapi-lab-user
   ```

3. Create the user.
4. Add the user to:

   ```text
   MetaPi-S3-ReadOnly-Group
   ```

> This IAM user is for permission testing inside the lab. In production environments, workforce federation and short-lived credentials are generally preferred.

---

### **Step 4: Review Effective Permissions**

1. Open `metapi-lab-user`.
2. Open the **Permissions** tab.
3. Verify that S3 read-only permissions are inherited from the group.
4. Review the attached policy.

---

### **Step 5: Create a Custom IAM Policy**

1. Go to **IAM → Policies → Create policy**.
2. Choose **JSON**.
3. Enter:

   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Action": [
           "s3:ListAllMyBuckets"
         ],
         "Resource": "*"
       }
     ]
   }
   ```

4. Name it:

   ```text
   MetaPi-List-S3-Buckets
   ```

5. Create the policy.

---

### **Step 6: Attach and Test the Custom Policy**

1. Return to `metapi-lab-user`.
2. Add permissions.
3. Attach `MetaPi-List-S3-Buckets`.
4. Review the user's permissions.

Students should notice that permissions can come from both groups and directly attached policies.

---

### **Step 7: Create an EC2 IAM Role**

1. Go to **IAM → Roles → Create role**.
2. Select:
   - **Trusted entity type:** AWS service
   - **Use case:** EC2
3. Attach:

   ```text
   AmazonS3ReadOnlyAccess
   ```

4. Name the role:

   ```text
   MetaPi-EC2-S3-ReadOnly-Role
   ```

5. Create the role.

---

### **Step 8: Review the Role Trust Policy**

1. Open `MetaPi-EC2-S3-ReadOnly-Role`.
2. Open **Trust relationships**.
3. Review the trust policy.

You should see that the EC2 service is trusted to assume the role.

---

### **Step 9: Understand IAM Relationships**

```text
IAM User
   |
   +--> IAM Group
   |       |
   |       +--> Policies
   |
   +--> Direct Policies

EC2 Instance
   |
   v
IAM Role
   |
   v
Temporary Credentials
   |
   v
AWS API Permissions
```

---

### **Step 10: Compare User and Role Usage**

- **IAM user:** Identity for a person or workload that receives permissions.
- **IAM group:** Collection used to manage permissions for multiple IAM users.
- **IAM policy:** JSON permission document.
- **IAM role:** Assumable identity that provides temporary credentials.

Later labs will attach `MetaPi-EC2-S3-ReadOnly-Role` to EC2.

---

### **Step 11: Lab Verification**

Verify that you can:

- Create an IAM group.
- Add a user to a group.
- Attach managed policies.
- Create a custom policy.
- Create an EC2 IAM role.
- Review a trust relationship.
- Explain least privilege.

---

### **Step 12: Secure and Clean Up**

1. Remove the test user if it will not be reused.
2. Delete the test group after removing members.
3. Keep `MetaPi-EC2-S3-ReadOnly-Role` if it will be reused in Lab 23.
4. Delete unused custom policies.

---

### **Lab Completed**

You have practiced AWS identity and authorization using **IAM users, groups, policies, and roles**.

**Next:** **Lab 09 — Create and Manage an Amazon S3 Bucket**
