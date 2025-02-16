---

# **Resizing Ubuntu 20.04 Disk Space in a Proxmox VM**  
This guide covers increasing disk space in an Ubuntu 20.04 virtual machine (VM) running on **Proxmox**.

---

## **Step 1: Increase Disk Size in Proxmox**  
Before resizing the partition inside Ubuntu, you must first **increase the virtual disk size** in Proxmox.

### **1. Increase Disk Space in Proxmox Web UI**
1. **Log in to Proxmox Web UI.**  
2. **Select your VM** from the left panel.  
3. Click **"Hardware"** → Select the disk (e.g., `virtio0`, `scsi0`).  
4. Click **"Resize Disk"** and **add the desired extra space** (e.g., `+10G`).  
5. Click **"Resize"** to apply the changes.  

💡 **Note:** This only increases the disk size at the virtualization level. You still need to resize the partition inside Ubuntu.

---

## **Step 2: Verify the New Disk Size in Ubuntu**
### **1. Check Available Disk Space**
Run the following command inside the Ubuntu VM:
```bash
lsblk
```
Sample output:
```
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
vda    254:0    0   50G  0 disk
├─vda1 254:1    0  512M  0 part /boot
└─vda2 254:2    0   40G  0 part /
```
If the original disk was **40GB** and you increased it to **50GB**, the extra **10GB** will be **unallocated**.

---

## **Step 3: Resize the Partition**
Depending on whether your setup is using **LVM** or a **standard partition**, follow the correct method below.

---

### **Option 1: For Standard Partitions (Non-LVM)**
#### **1. Resize the Partition using `growpart`**
First, install the necessary tool:
```bash
sudo apt update && sudo apt install -y cloud-guest-utils
```
Now, resize the partition:
```bash
sudo growpart /dev/vda 2
```
_(Replace `vda` with the correct disk name and `2` with the correct partition number from `lsblk`.)_

#### **2. Resize the File System**
- **If you are using `ext4` file system, run:**
  ```bash
  sudo resize2fs /dev/vda2
  ```
- Verify the new size:
  ```bash
  df -h
  ```

---

### **Option 2: For LVM Partitions**
If your VM uses **LVM (Logical Volume Manager)**, follow these steps:

#### **1. Resize the Physical Partition**
```bash
sudo growpart /dev/vda 2
```

#### **2. Extend the Logical Volume**
```bash
sudo pvresize /dev/vda2
```
Find the logical volume name:
```bash
sudo lvdisplay
```
Now, extend it:
```bash
sudo lvextend -l +100%FREE /dev/ubuntu-vg/root
```

#### **3. Resize the File System**
- **For ext4 file system:**
  ```bash
  sudo resize2fs /dev/ubuntu-vg/root
  ```
- **For XFS file system:**
  ```bash
  sudo xfs_growfs /dev/ubuntu-vg/root
  ```

---

## **Step 4: Verify the Changes**
After resizing, check if the changes were applied successfully.

### **1. Confirm the Partition Size**
```bash
lsblk
```
You should now see the partition using the full disk size.

### **2. Check the File System Size**
```bash
df -h
```
It should now show the new disk space available on `/`.

---

## **Step 5: Reboot the System (If Necessary)**
If the changes do not reflect, reboot your VM:
```bash
sudo reboot
```

---

## **Troubleshooting**
### **1. `growpart` Command Not Found**
Run:
```bash
sudo apt install -y cloud-guest-utils
```
Then retry:
```bash
sudo growpart /dev/vda 2
```

### **2. `resize2fs` Reports "Nothing to do"**
If using LVM, ensure you extended the logical volume first:
```bash
sudo lvextend -l +100%FREE /dev/ubuntu-vg/root
```
Then retry:
```bash
sudo resize2fs /dev/ubuntu-vg/root
```

---
