# **Lab 07 — Create an EBS Snapshot and Restore Data**

### **Step 1: Prepare an EBS Volume with Test Data**

1. Use the EBS volume from Lab 06 or create a new 5 GiB volume.
2. Attach and mount it at `/data`.
3. Create test files:

   ```bash
   echo "MetaPi Snapshot Test" | sudo tee /data/snapshot-test.txt
   sudo mkdir -p /data/project
   echo "Important project data" | sudo tee /data/project/data.txt
   ```

4. Verify:

   ```bash
   find /data -maxdepth 2 -type f -print
   ```

---

### **Step 2: Create an EBS Snapshot**

1. Go to **EC2 → Elastic Block Store → Volumes**.
2. Select the data volume.
3. Choose **Actions → Create snapshot**.
4. Add:
   - **Description:** `MetaPi Lab 07 backup`
   - **Name tag:** `MetaPi-Lab07-Snapshot`
5. Create the snapshot.

---

### **Step 3: Wait for the Snapshot**

1. Go to **EC2 → Snapshots**.
2. Select `MetaPi-Lab07-Snapshot`.
3. Wait until the status becomes **Completed**.

An EBS snapshot is a point-in-time backup of the volume.

---

### **Step 4: Simulate Data Loss**

On the EC2 instance:

```bash
sudo rm -f /data/snapshot-test.txt
sudo rm -rf /data/project
```

Verify:

```bash
ls -la /data
```

The test data should be gone.

---

### **Step 5: Create a New Volume from the Snapshot**

1. In **EC2 → Snapshots**, select `MetaPi-Lab07-Snapshot`.
2. Choose **Actions → Create volume from snapshot**.
3. Configure:
   - **Availability Zone:** Same AZ as your EC2 instance
   - **Volume type:** General Purpose SSD
4. Name it:

   ```text
   MetaPi-Lab07-Restored
   ```

5. Create the volume.

---

### **Step 6: Attach the Restored Volume**

1. Go to **EC2 → Volumes**.
2. Select `MetaPi-Lab07-Restored`.
3. Choose **Actions → Attach volume**.
4. Select the lab EC2 instance.
5. Attach it.

---

### **Step 7: Identify the Restored Disk**

On EC2:

```bash
lsblk -f
```

Identify the restored volume by its filesystem and size.

Example device:

```text
/dev/nvme2n1
```

Do **not** run `mkfs` on the restored volume because it already contains the filesystem from the snapshot.

---

### **Step 8: Mount the Restored Volume**

Create a mount point:

```bash
sudo mkdir -p /restore
```

Mount the restored disk:

```bash
sudo mount /dev/nvme2n1 /restore
```

Verify:

```bash
df -h
```

---

### **Step 9: Verify Restored Data**

Run:

```bash
cat /restore/snapshot-test.txt
cat /restore/project/data.txt
```

Expected:

```text
MetaPi Snapshot Test
Important project data
```

The files deleted from the original volume exist on the restored volume because the snapshot was created before deletion.

---

### **Step 10: Understand Backup and Restore**

```text
EBS Volume
    |
    v
 Snapshot
    |
    v
New EBS Volume
    |
    v
 Restored Data
```

---

### **Step 11: Optional Recovery Exercise**

Copy the recovered data back:

```bash
sudo cp -a /restore/snapshot-test.txt /data/
sudo cp -a /restore/project /data/
```

Verify:

```bash
find /data -maxdepth 2 -type f -print
```

---

### **Step 12: Lab Verification**

Verify that you can:

- Create an EBS snapshot.
- Wait for snapshot completion.
- Create a volume from a snapshot.
- Attach a restored volume.
- Mount it without formatting.
- Recover deleted files.

---

### **Step 13: Secure and Clean Up**

1. Unmount:

   ```bash
   sudo umount /restore
   ```

2. Terminate the EC2 instance if no longer needed.
3. Delete temporary EBS volumes.
4. Delete the snapshot if your instructor does not require it for later work.

---

### **Lab Completed**

You have created a point-in-time EBS backup and restored data from it.

**Next:** **Lab 08 — Create IAM Users, Groups, Policies, and Roles**
