# **Lab 06 — Create Attach Format and Mount an EBS Volume**

### **Step 1: Launch an EC2 Instance**

1. Launch an Ubuntu EC2 instance named:

   ```text
   MetaPi-Lab06-EBS
   ```

2. Use the approved training instance type.
3. Allow **SSH (22)** from **My IP**.
4. Note the instance's **Availability Zone**.

> An EBS volume must be in the same Availability Zone as the EC2 instance to attach normally.

---

### **Step 2: Create an EBS Volume**

1. Go to **EC2 → Elastic Block Store → Volumes**.
2. Click **Create volume**.
3. Configure:
   - **Volume type:** General Purpose SSD
   - **Size:** `5 GiB`
   - **Availability Zone:** Same AZ as `MetaPi-Lab06-EBS`
4. Add the Name tag:

   ```text
   MetaPi-Lab06-Data
   ```

5. Create the volume.

---

### **Step 3: Attach the EBS Volume**

1. Select `MetaPi-Lab06-Data`.
2. Choose **Actions → Attach volume**.
3. Select `MetaPi-Lab06-EBS`.
4. Accept the suggested device name.
5. Click **Attach volume**.

---

### **Step 4: Connect to EC2 and Identify the Disk**

1. SSH to the instance.
2. Run:

   ```bash
   lsblk
   ```

3. Identify the new unmounted disk, for example:

   ```text
   nvme1n1
   ```

> Device names can differ by instance type. Always use `lsblk` before formatting.

---

### **Step 5: Check Whether the Disk Has a File System**

Replace the device name if yours is different:

```bash
sudo file -s /dev/nvme1n1
```

A new blank volume normally shows that it has no filesystem.

---

### **Step 6: Format the EBS Volume**

Create an ext4 filesystem:

```bash
sudo mkfs.ext4 /dev/nvme1n1
```

> Formatting destroys existing data. Only format a new blank lab volume.

---

### **Step 7: Create a Mount Point**

Create a directory:

```bash
sudo mkdir -p /data
```

Mount the volume:

```bash
sudo mount /dev/nvme1n1 /data
```

Verify:

```bash
df -h
```

---

### **Step 8: Store Data on the EBS Volume**

Create a file:

```bash
echo "MetaPi EBS Lab Data" | sudo tee /data/metapi.txt
```

Read it:

```bash
cat /data/metapi.txt
```

Expected output:

```text
MetaPi EBS Lab Data
```

---

### **Step 9: Configure Persistent Mounting**

Get the UUID:

```bash
sudo blkid /dev/nvme1n1
```

Copy the UUID.

Back up `/etc/fstab`:

```bash
sudo cp /etc/fstab /etc/fstab.backup
```

Edit:

```bash
sudo nano /etc/fstab
```

Add a line using your actual UUID:

```text
UUID=<YOUR_UUID>  /data  ext4  defaults,nofail  0  2
```

Save the file.

Test the configuration:

```bash
sudo umount /data
sudo mount -a
df -h
```

Verify that `/data` is mounted again.

---

### **Step 10: Reboot and Verify**

Reboot:

```bash
sudo reboot
```

Reconnect after the instance starts.

Run:

```bash
df -h
cat /data/metapi.txt
```

The volume should be mounted and the file should still exist.

---

### **Step 11: Observe EC2 and EBS Separation**

```text
EC2 Instance
     |
     +------ Root EBS Volume
     |
     +------ MetaPi-Lab06-Data
                 |
                 v
               /data
```

The additional EBS volume is separate from the operating-system root volume.

---

### **Step 12: Lab Verification**

Verify that you can:

- Create an EBS volume.
- Match its Availability Zone with EC2.
- Attach a volume.
- Identify disks with `lsblk`.
- Create a filesystem.
- Mount a volume.
- Configure `/etc/fstab`.
- Verify data after reboot.

---

### **Step 13: Secure and Clean Up**

1. If keeping the data for Lab 07, do **not** delete `MetaPi-Lab06-Data`.
2. Otherwise:
   - Unmount `/data`.
   - Terminate the EC2 instance.
   - Delete the extra EBS volume after it is detached.
3. Verify no unused volumes remain.

---

### **Lab Completed**

You have attached and configured persistent block storage using **Amazon EBS**.

**Next:** **Lab 07 — Create an EBS Snapshot and Restore Data**
