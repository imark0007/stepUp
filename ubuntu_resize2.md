# Resizing Ubuntu 20.04 Disk Space in Proxmox VM

Resizing the disk space of an Ubuntu 20.04 virtual machine (VM) in Proxmox involves two main steps:
1. **Increasing the disk size in Proxmox** (host-level change)
2. **Resizing the partitions and file system in Ubuntu** (guest-level change)

---

## **Step 1: Increase Disk Size in Proxmox**
1. **Log in to the Proxmox Web UI**.
2. Select the **VM** from the left panel.
3. Go to **"Hardware"** and select the disk (e.g., `virtio0` or `scsi0`).
4. Click **"Resize Disk"**, enter the additional disk size (e.g., `+10G`), and click **"Resize"**.

At this point, the VM's disk has been enlarged, but Ubuntu is unaware of this change. Now, we need to extend the partition inside the Ubuntu VM.

---

## **Step 2: Resize the Partition in Ubuntu**

### **1. Check the Current Disk Space**
Run the following command inside the Ubuntu VM:
```bash
lsblk
```
Example output:
```
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
vda    254:0    0   50G  0 disk
├─vda1 254:1    0  512M  0 part /boot
└─vda2 254:2    0   40G  0 part /
```
If you increased the disk from **40GB to 50GB**, the extra **10GB will be unallocated**.

---

### **2. Resize the Partition**
#### **Option 1: Using `growpart` (Recommended for Most Setups)**
1. Install `cloud-guest-utils` if not already installed:
   ```bash
   sudo apt update && sudo apt install -y cloud-guest-utils
   ```
2. Resize the partition:
   ```bash
   sudo growpart /dev/vda 2
   ```
   - Replace `/dev/vda 2` with your correct partition (use `lsblk` to check).

#### **Option 2: Using `fdisk` (If `growpart` Doesn’t Work)**
1. Run `fdisk` on the disk:
   ```bash
   sudo fdisk /dev/vda
   ```
2. Inside `fdisk`:
   - Press `p` to print the current partition table.
   - Press `d` to delete the root partition (**Don't worry, we will recreate it**).
   - Press `n` to create a new partition.
   - Accept the default values (this will use the full disk size).
   - Press `w` to write changes.
3. Reboot the system:
   ```bash
   sudo reboot
   ```

---

### **3. Resize the File System**
Once the partition is extended, resize the file system accordingly.

#### **For ext4 File System (Standard Ubuntu Setup):**
```bash
sudo resize2fs /dev/vda2
```
Replace `/dev/vda2` with your correct partition name.

#### **For LVM Setup:**
```bash
sudo lvextend -l +100%FREE /dev/ubuntu-vg/root
sudo resize2fs /dev/ubuntu-vg/root
```

---

## **Step 3: Verify Changes**
Run:
```bash
df -h
```
Check if the root (`/`) partition reflects the new size.

---
