## Day 13 – LVM with AWS EBS (Hands-on)

Today I practiced **Logical Volume Management (LVM)** using multiple AWS EBS volumes on an EC2 instance.

### What I learned
- Difference between Physical Volume, Volume Group, and Logical Volume
- How AWS EBS works as block storage for EC2
- Creating and managing LVM for dynamic storage
- Extending Logical Volume without data loss

### Hands-on tasks
- Created and attached multiple EBS volumes
- Initialized Physical Volumes using `pvcreate`
- Created Volume Group and Logical Volume
- Formatted and mounted LVM
- Extended Logical Volume and verified changes


### LVM Concepts

LVM (Logical Volume Manager) allows you to manage disk storage flexibly.
resize storage on the fly without reinstalling OS or losing data.

## Physical Volume (PV)

- Actual disk (EBS volume)
- Example: `/dev/nvme1n1`

## Volume Group (VG)

- Pool of multiple Physical Volumes
- Example: `data_vg`

## Logical Volume (LV)

- Virtual partition created from VG
- Example: `data_lv`

### Flow: 
`EBS Volume → Physical Volume → Volume Group → Logical Volume → Mount`

### AWS EBS Volume – Create & Attach

1. Go to AWS Console → EC2 → Volumes
2. Click Create volume
3. Select:
- Volume type: gp3
- Size: as required
- Availability Zone: same as EC2
4. Attach volume to EC2 instance
5. Volume appears in OS as /dev/nvmeXn1
<img width="1292" height="373" alt="EBS volumes" src="https://github.com/user-attachments/assets/ed95826a-5ec4-4c93-bc9c-4d2405f4f86a" />

### verified this using 
`lsblk`  

<img width="728" height="336" alt="lsblk" src="https://github.com/user-attachments/assets/8d7a77ae-3744-47f8-b0ce-5a3572552a94" />

### Commands I Used + Purpose

## Disk & Storage Checks 

- Shows all disks, partitions, and mount points `lsblk`
- Displays filesystem usage in human-readable format `df -h`

## LVM Commands

- Initialize disks as Physical Volumes `pvcreate /dev/nvme1n1 /dev/nvme2n1 /dev/nvme3n1`
  <img width="727" height="563" alt="pvcreate" src="https://github.com/user-attachments/assets/c42d605f-8452-4efa-a14f-312dc405dd56" />

- Show Physical Volume details `pvs`
- Create Volume Group from PVs `vgcreate data_vg /dev/nvme1n1 /dev/nvme2n1`
- Display Volume Group information `vgs` `vgdisplay`
- Create Logical Volume of 8GB `lvcreate -L 8G -n data_lv data_vg`
<img width="727" height="180" alt="lvcreate" src="https://github.com/user-attachments/assets/8ba65752-1da1-4985-9d40-bde676b69ca0" />

- Show Logical Volume details `lvs`
- Extend Logical Volume size dynamically `lvextend -L +2G /dev/data_vg/data_lv`

## Filesystem & Mounting

- Create filesystem on Logical Volume `mkfs.ext4 /dev/data_vg/data_lv`
<img width="991" height="700" alt="mount" src="https://github.com/user-attachments/assets/24228e0d-212f-4708-837d-06d24289112c" />

- Mount LVM volume `mkdir /mnt/data_lv_mount` `mount /dev/data_vg/data_lv /mnt/data_lv_mount`
<img width="991" height="700" alt="mount" src="https://github.com/user-attachments/assets/1c657d1d-47fd-45d0-b1fb-26ee20ff4ca8" />

- Direct disk formatting & mounting (without LVM) `mkfs -t ext4 /dev/nvme3n1` `mkdir /mnt/data_disk_mount` `mount /dev/nvme3n1 /mnt/data_disk_mount`
  <img width="996" height="461" alt="disk mount1" src="https://github.com/user-attachments/assets/703f7b7d-22a0-402f-b894-aef099451be3" />
  <img width="996" height="348" alt="disk mount 2" src="https://github.com/user-attachments/assets/49f0d850-e976-4442-9073-ff06b4d9d783" />


### Why LVM is Important?
- Resize storage without downtime
- Combine multiple disks easily
- Better storage management in cloud
- Used heavily in production servers
  
### This practice helped me understand **real-world storage management** used in production systems.

