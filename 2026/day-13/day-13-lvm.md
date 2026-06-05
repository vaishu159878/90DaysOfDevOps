# Day 13 – Linux Volume Management (LVM)

## 📘 Objective

Today I learned Linux Logical Volume Management (LVM), which provides flexible storage management by allowing disks to be combined, resized, and managed without the limitations of traditional partitions.

---


# 📋 Task 1: Check Current Storage

## Commands Used

lsblk
df -h
pvs
vgs
lvs


### Purpose

These commands help identify available disks, mounted filesystems, and existing LVM configurations.

---

# 💾 Task 2: Create Physical Volumes (PV)

## Commands Used


sudo pvcreate /dev/nvme1n1 /dev/nvme2n1 /dev/nvme3n1
sudo pvs
sudo pvdisplay


### Purpose

Physical Volumes (PV) are the actual disks prepared for use by LVM.





---

# 📦 Task 3: Create Volume Group (VG)

## Commands Used


sudo vgcreate devops-vg /dev/nvme1n1 /dev/nvme2n1
sudo vgs
sudo vgdisplay


### Purpose

A Volume Group combines one or more Physical Volumes into a storage pool from which Logical Volumes can be created.



---

# 📁 Task 4: Create Logical Volume (LV)

## Commands Used


sudo lvcreate -L 5G -n vaishnavi devops-vg
sudo lvs
sudo lvdisplay


### Purpose

A Logical Volume is virtual storage created from the Volume Group.


---

# 🗂️ Task 5: Format and Mount Logical Volume

## Commands Used


sudo mkfs.ext4 /dev/devops-vg/vaishnavi

sudo mkdir -p /mnt/vaishnavi

sudo mount /dev/devops-vg/vaishnavi /mnt/vaishnavi

df -h /mnt/vaishnavi


### Purpose

The Logical Volume was formatted using the EXT4 filesystem and mounted for use.


---

# 📝 Task 6: Verify Storage

## Commands Used


echo "Hii" > /mnt/vaishnavi/hello.txt

cat /mnt/vaishnavi/hello.txt


### Purpose

Created a test file to verify the mounted storage is working correctly.


---

# 📈 Task 7: Extend Logical Volume

## Commands Used


sudo lvextend -L +2G /dev/devops-vg/vaishnavi

sudo resize2fs /dev/devops-vg/vaishnavi


### Purpose

Extended the Logical Volume from 5 GB to 7 GB and resized the filesystem to use the additional space.


---

# ✅ Final Verification

## Commands Used

df -h

sudo pvs

sudo vgs

sudo lvs


### Result

* Physical Volumes successfully created
* Volume Group successfully created
* Logical Volume successfully created
* Filesystem mounted successfully
* Logical Volume extended from 5 GB to 7 GB
* Filesystem resized successfully

---

# 🎯 Key Concepts Learned

### Physical Volume (PV)

The actual disk or partition initialized for LVM.

### Volume Group (VG)

A storage pool created by combining one or more Physical Volumes.

### Logical Volume (LV)

A virtual partition created from the Volume Group.

### Filesystem

Created on top of the Logical Volume and mounted for storing data.

---

# 💡 What I Learned

1. LVM provides flexibility by separating physical storage from logical storage.
2. Multiple disks can be combined into a single Volume Group.
3. Logical Volumes can be extended without recreating partitions.
4. Filesystems must be resized after extending a Logical Volume.
5. LVM simplifies storage management in real-world Linux environments.

---


## Screenshots

img1.png
img2.png
img3.png


