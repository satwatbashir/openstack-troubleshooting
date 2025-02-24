# Troubleshooting Guide: Expanding Disk Space in VirtualBox Ubuntu VM

## **Issue: Low Disk Space Warning on Root Filesystem (`/`)
### **Cause:**
- The virtual disk was allocated 80GB, but the root partition (`/dev/sda3`) only used 20GB.
- Unallocated space was not utilized by the system.

---

## **Solution: Expanding the Root Partition to Utilize Full Disk Space**

### **Step 1: Check Current Disk Space & Partitions**
Run the following commands to inspect disk usage and partitions:
```bash
# Check available disk space
df -h

# Check partition structure
lsblk

# Check full partition details
sudo fdisk -l
```

Expected Output:
- `df -h` should show `/dev/sda3` is nearly full.
- `lsblk` should show that `sda3` is **smaller** than the total available disk (`sda`).
- `fdisk -l` should confirm **unallocated space** exists on `sda`.

---

### **Step 2: Install Required Tools**
Ensure `growpart` is installed:
```bash
sudo apt update
sudo apt install -y cloud-guest-utils
```

Verify installation:
```bash
which growpart
```

Expected Output:
```
/usr/bin/growpart
```

---

### **Step 3: Extend the Partition (`/dev/sda3`)**
Extend the partition using `growpart`:
```bash
sudo growpart /dev/sda 3
```

Expected Output:
```
CHANGED: partition=3 start=1054720 old: size=40886272 end=41940992 new: size=166717407 end=167772127
```

Check the new partition size:
```bash
lsblk
```
✅ `/dev/sda3` should now be close to **80GB**.

---

### **Step 4: Resize the Filesystem**
After resizing the partition, we need to expand the filesystem:
```bash
sudo resize2fs /dev/sda3
```

Expected Output:
```
Filesystem at /dev/sda3 is mounted on /; online resizing required
old_desc_blocks = 3, new_desc_blocks = 10
The filesystem on /dev/sda3 is now 20839675 (4k) blocks long.
```

---

### **Step 5: Verify Changes**
Check the final disk space allocation:
```bash
df -h
```

✅ Expected Output:
```
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda3        79G   18G   58G  24% /
```

---

## **Final Check: Restart & Verify Persistence**
Reboot the system:
```bash
sudo reboot
```
After the reboot, verify the changes:
```bash
df -h
```
✅ If `/dev/sda3` still shows **79GB**, everything is successfully resized and persistent.

---

## **Summary of Commands**
| **Step** | **Command** |
|----------|------------|
| **Check disk space** | `df -h` |
| **Check partitions** | `lsblk` |
| **Check full disk details** | `sudo fdisk -l` |
| **Install required tools** | `sudo apt install -y cloud-guest-utils` |
| **Extend partition** | `sudo growpart /dev/sda 3` |
| **Expand filesystem** | `sudo resize2fs /dev/sda3` |
| **Verify new space** | `df -h` |
| **Restart system** | `sudo reboot` |


Now, you have a complete guide for fixing disk space issues in your OpenStack VM. 🚀 Let me know if you need modifications! 😊

