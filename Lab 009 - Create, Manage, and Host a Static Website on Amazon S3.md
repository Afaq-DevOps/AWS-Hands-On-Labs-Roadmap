### **Step 1: Prepare the MetaPi Website Files**

Your instructor will provide a ZIP file containing the website.

Example:

```text
https://github.com/Afaq-DevOps/metapi-s3-website-lab9-code.git
```

Clone the code or Download ZIP file

Extract the ZIP file

You should see:

```text
metapi-s3-website-lab9-code/
│
├── index.html
├── about.html
├── services.html
├── labs.html
├── contact.html
├── error.html
└── style.css
```

---

### **Step 2: Understand the Website Structure**

Each page is stored as a separate S3 object.

```text
index.html
```

Main website page.

```text
about.html
```

Information about MetaPi AWS training.

```text
services.html
```

Displays AWS services covered during training.

```text
labs.html
```

Displays the AWS hands-on lab learning path.

```text
contact.html
```

Student verification and navigation page.

```text
error.html
```

Custom error page.

```text
style.css
```

Shared styling for all pages.

---

### **Step 3: Create an S3 Bucket**

1. Open the AWS Console.
    
2. Search for:
    
    ```text
    S3
    ```
    
3. Click **Create bucket**.
    
4. Enter a globally unique bucket name:
    
    ```text
    metapi-s3-website-<YOUR-NAME>-<RANDOM>
    ```
    

Example:

```text
metapi-s3-website-afaq-AWS-SAA-B04-MetaPi
```

5. Select the training Region.
    
6. Keep the remaining settings at their defaults for now.
    
7. Click **Create bucket**.
    

> S3 bucket names must be globally unique.

---

### **Step 4: Upload the Website Files**

1. Open your new S3 bucket.
    
2. Click **Upload**.
    
3. Click **Add files**.
    
4. Select:
    
    ```text
    index.html
    about.html
    services.html
    labs.html
    contact.html
    error.html
    style.css
    ```
    
5. Click **Upload**.
    
6. Wait for the upload to complete.
    

---

### **Step 5: Verify the Object Structure**

Return to the bucket root.

You should see:

```text
S3 Bucket
│
├── index.html
├── about.html
├── services.html
├── labs.html
├── contact.html
├── error.html
└── style.css
```

All files should be uploaded directly into the bucket root.

Do not upload them like this:

```text
S3 Bucket
   |
   └── metapi-s3-website/
          |
          ├── index.html
          ├── about.html
          └── ...
```

For this lab, `index.html` should exist directly in the root of the bucket.

---

### **Step 6: Enable Versioning and Static Website Hosting**

1. Open the bucket **Properties** tab.
	
2.  Edit Bucket Versioning

	 Bucket Versioning
	 
		 Enable
    
3. Scroll down to:
    
    **Static website hosting**
    
4. Click **Edit**.
    
5. Select:
    
    **Enable**
    
6. Under **Hosting type**, select:
    
    **Host a static website**
    
7. For **Index document**, enter:
    
    ```text
    index.html
    ```
    
8. For **Error document**, enter:
    
    ```text
    error.html
    ```
    
9. Click **Save changes**.
    

Your configuration should now be:

```text
Static website hosting: Enabled

Index document:
index.html

Error document:
error.html
```

---

### **Step 7: Copy the Website Endpoint**

1. Stay on the **Properties** tab.
    
2. Scroll down to **Static website hosting**.
    
3. Find the:
    
    **Bucket website endpoint**
    

It will look similar to:

```text
http://<BUCKET-NAME>.s3-website-<REGION>.amazonaws.com
```

Copy the URL.

---

### **Step 8: Test the Website Before Public Access**

Open the website endpoint in your browser.

You may see:

```text
403 Forbidden
```

This happens because static website hosting is enabled, but the website objects are not yet publicly readable.

The current flow is:

```text
Browser
   |
   v
S3 Website Endpoint
   |
   X
   |
Website Objects
```

---

### **Step 9: Disable Block Public Access for the Lab**

1. Open the bucket **Permissions** tab.
    
2. Find:
    
    **Block public access**
    
3. Click **Edit**.
    
4. Disable:
    
    ```text
    Block all public access
    ```
    
5. Acknowledge the AWS warning.
    
6. Save the changes.
    

> This is only for the temporary classroom static website exercise. Do not store private or sensitive files in this bucket.

---

### **Step 10: Add the Website Bucket Policy**

In the **Permissions** tab:

1. Scroll to:
    
    **Bucket policy**
    
2. Click **Edit**.
    
3. Add:
    

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadForWebsite",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::YOUR-BUCKET-NAME/*"
    }
  ]
}
```

Replace:

```text
YOUR-BUCKET-NAME
```

with your real bucket name.

Example:

```text
arn:aws:s3:::metapi-s3-website-afaq-AWS-SAA-B04-MetaPi/*
```

Save the policy.

---

### **Step 11: Open the Website**

Return to:

**Properties → Static website hosting**

Copy the **Bucket website endpoint**.

Open it in a browser.

You should now see:

```text
MetaPi AWS Training

Learn AWS by Building
```

The S3-hosted website is now working.

---

### **Step 12: Test the Home Page**

The home page is:

```text
index.html
```

Verify that you can see:

- MetaPi AWS Training
    
- AWS training information
    
- Navigation menu
    
- Cards
    
- Styled buttons
    
- AWS-inspired page design
    

---

### **Step 13: Test the About Page**

Click:

```text
About
```

The browser should load:

```text
about.html
```

Verify that the page opens without a `403` error.

---

### **Step 14: Test the AWS Services Page**

Click:

```text
AWS Services
```

The browser should load:

```text
services.html
```

---

### **Step 15: Test the Labs Page**

Click:

```text
Labs
```

The browser should load:

```text
labs.html
```

Verify that the MetaPi AWS hands-on lab learning path appears.

---

### **Step 16: Test the Contact Page**

Click:

```text
Contact
```

The browser should load:

```text
contact.html
```

Verify that the page loads correctly.

Use the:

```text
Return Home
```

button to return to `index.html`.

---

### **Step 17: Understand the Multi-Page Navigation**

The website uses relative links.

Example:

```html
<a href="index.html">Home</a>
<a href="about.html">About</a>
<a href="services.html">AWS Services</a>
<a href="labs.html">Labs</a>
<a href="contact.html">Contact</a>
```

The structure works like this:

```text
             index.html
                 |
      +----------+----------+
      |          |          |
      v          v          v
 about.html  services.html labs.html
      |                     |
      +----------+----------+
                 |
                 v
           contact.html
```

---

### **Step 18: Verify the CSS File**

The pages use:

```text
style.css
```

The HTML loads the CSS with:

```html
<link rel="stylesheet" href="style.css">
```

The browser request looks like:

```text
Browser
   |
   +----> index.html
   |
   +----> style.css
```

If the website displays without styling, verify that:

```text
style.css
```

exists in the bucket root.

---

### **Step 19: Test the Custom Error Page**

Now test what happens when a page does not exist.

Add the following to the website URL:

```text
/not-found.html
```

Example:

```text
http://<S3-WEBSITE-ENDPOINT>/not-found.html
```

Because the object does not exist, the configured:

```text
error.html
```

page should be displayed.

You should see something similar to:

```text
404

Page Not Found

The page you requested does not exist in this S3 website.

Return to Home
```

Click:

```text
Return to Home
```

Verify that the website returns to:

```text
index.html
```

---

### **Step 20: Troubleshoot 403 Forbidden**

If you see:

```text
403 Forbidden
```

check the following.

#### Check 1 — Block Public Access

Open:

**Permissions → Block public access**

Confirm that the training bucket is allowing the public access required for this exercise.

---

#### Check 2 — Bucket Policy

Confirm that the bucket policy contains:

```json
"Action": "s3:GetObject"
```

and:

```json
"Principal": "*"
```

---

#### Check 3 — Bucket ARN**

Confirm your policy contains the correct bucket name.

Example:

```text
arn:aws:s3:::metapi-s3-website-afaq-AWS-SAA-B04-MetaPi/*
```

Do not leave:

```text
YOUR-BUCKET-NAME
```

inside the real policy.

---

#### Check 4 — Website Endpoint

Make sure you are using the:

```text
Bucket website endpoint
```

from:

**Properties → Static website hosting**

---

### **Step 21: Troubleshoot Broken Navigation**

If a menu link does not work, verify that the object exists.

For:

```text
About
```

you need:

```text
about.html
```

For:

```text
AWS Services
```

you need:

```text
services.html
```

For:

```text
Labs
```

you need:

```text
labs.html
```

For:

```text
Contact
```

you need:

```text
contact.html
```

Object names must match exactly.

For example:

```text
about.html
```

is different from:

```text
About.html
```

---

### **Step 22: Understand the S3 Static Website Architecture**

The completed architecture is:

```text
                 Internet User
                       |
                       v
             S3 Website Endpoint
                       |
                       v
                Amazon S3 Bucket
                       |
        +--------------+--------------+
        |              |              |
        v              v              v
   index.html      about.html     labs.html
        |
        +----> services.html
        |
        +----> contact.html
        |
        +----> style.css
        |
        +----> error.html
```

There is:

```text
No EC2
No Apache
No Nginx
No operating system
```

S3 serves the static content directly.

---

### **Step 23: Understand What S3 Can Host**

S3 static website hosting can serve files such as:

```text
HTML
CSS
JavaScript
Images
Fonts
Documents
```

For example:

```text
index.html
style.css
app.js
logo.png
```

S3 static website hosting does not execute server-side applications such as:

```text
PHP
Python backend code
Node.js server applications
Java server applications
MySQL
```

---

### **Step 24: Lab Verification**

Verify that you can:

- Create an S3 bucket.
    
- Upload a multi-page website.
    
- Upload HTML and CSS objects.
    
- Enable static website hosting.
    
- Configure `index.html`.
    
- Configure `error.html`.
    
- Find the S3 website endpoint.
    
- Understand a `403 Forbidden` error.
    
- Configure a bucket policy.
    
- Navigate between multiple HTML pages.
    
- Verify shared CSS.
    
- Test a custom error page.
    
- Troubleshoot broken links.
    
- Explain how S3 serves a static website.
    

---

### **Step 25: Secure and Clean Up**

If the lab is finished:

1. Open **Permissions**.
    
2. Remove the temporary public website bucket policy.
    
3. Re-enable:
    
    ```text
    Block all public access
    ```
    
4. Open **Properties**.
    
5. Disable **Static website hosting**.
    
6. Delete the website files if they are no longer needed.
    
7. Delete the bucket if it will not be reused.
    

The final secure state should be:

```text
Internet
   |
   X
   |
Private S3 Bucket
```

---

### **Lab Completed**

You have successfully built:

```text
Amazon S3
    |
    v
Static Website Hosting
    |
    +----> index.html
    +----> about.html
    +----> services.html
    +----> labs.html
    +----> contact.html
    +----> style.css
    +----> error.html
```

You have also tested:

```text
Multi-Page Navigation
        +
Shared CSS
        +
Custom Error Page
        +
403 Troubleshooting
```

