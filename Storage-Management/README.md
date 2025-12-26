CONFIGURING STORAGE WITH ANSIBLE MODULES:
-------------------------------------------
- Red Hat Ansible Engine provides a collection of modules to configure storage devices on managed hosts.
-  Those modules support partitioning devices, creating logical volumes, and creating and mounting filesystems.
- The `parted` module supports the partition of block devices.
- This module includes the functionality of the `parted` command, and allows to create partitions with a specific size, flag, and alignment.
---

# What the `parted` Module Does ?

- The **Ansible `community.general.parted` module** is used to **create, delete, and manage disk partitions** on Linux systems using the GNU `parted` tool.

It allows you to automate tasks such as:

- Creating new partitions  
- Resizing partitions  
- Setting filesystem flags  
- Removing partitions  
- Preparing disks for LVM, RAID, or filesystems  

This is much safer and cleaner than using raw `fdisk` commands.

---

#  Requirements  

According to the documentation:

- You must have the **community.general** collection installed:
  ```
  ansible-galaxy collection install community.general
  ```
- The target system must have the **parted** command available.

---

#**Ansible `parted` Module — Key Parameters Explained**


| **Parameter** | **What It Does** |
|--------------|------------------|
| **align** | Controls how the partition is aligned on disk (e.g., optimal, minimal). Helps with SSD/HDD performance. |
| **device** | The block device you want to manage, such as `/dev/sda`, `/dev/sdb`. |
| **flags** | Special attributes for the partition (e.g., `boot`, `lvm`, `raid`, `swap`). |
| **number** | The partition number (1, 2, 3…). Required when creating or deleting a specific partition. |
| **part_end** | Where the partition ends, measured from the start of the disk. Supports units like `MiB`, `GiB`, `%`. |
| **state** | Whether the partition should exist (`present`) or be removed (`absent`). |
| **unit** | The unit used for partition sizes (e.g., `MiB`, `GiB`, `s` for sectors). |

---

# Basic Example — Create a Partition

- A simple, correct example of creating a new partition on `/dev/sdb`:

```yaml
- name: Create primary partition on /dev/sdb
  community.general.parted:
    device: /dev/sdb
    number: 1
    state: present
    part_type: primary
    fs_type: ext4
    start: 1MiB
    end: 2GiB
```

### What this does:
- Creates partition **#1**  
- On disk **/dev/sdb**  
- From **1 MiB → 2 GiB**  
- Type: **primary**  
- Filesystem: **ext4**


# Example — Create an LVM Partition  
From the search results, LVM partitioning is also supported:

```yaml
- name: Create LVM partition on /dev/sdb
  community.general.parted:
    device: /dev/sdb
    number: 1
    state: present
    part_type: primary
    flags:
      - lvm
    start: 1MiB
    end: 100%
```

# Example — Remove a Partition

```yaml
- name: Remove partition 1
  community.general.parted:
    device: /dev/sdb
    number: 1
    state: absent
```

---

# Example — Print Partition Table

```yaml
- name: Print partition table
  community.general.parted:
    device: /dev/sdb
    state: info
```

** Why Use `parted` Instead of Shell Commands?**

- using `shell` commands like `fdisk` is unreliable and error‑prone, while `parted` is the correct automated approach.


The filesystem Module
----------------------

- The `filesystem` module supports both creating and resizing a filesystem.
-  This module supports filesystem resizing for ext2, ext3, ext4, ext4dev, f2fs, lvm, xfs, and vfat.

  **Ansible `filesystem` Module — Key Parameters**

| **Parameter** | **Description** |
|---------------|------------------|
| **dev** | The **block device** on which the filesystem will be created or resized. Example: `/dev/sdb1`. |
| **fstype** | The **filesystem type** to create. Common values: `ext4`, `xfs`, `ext3`, `btrfs`. |
| **resizefs** | If `yes`, Ansible will **grow the filesystem** to match the size of the underlying block device. Useful after extending an LV or partition. |

---

#  **Example: Create an ext4 Filesystem**

```yaml
- name: Create ext4 filesystem on /dev/sdb1
  filesystem:
    fstype: ext4
    dev: /dev/sdb1
```

The mount Module:
----------------
- The `mount` module supports the configuration of mount points on /etc/fstab.
*Mount Module — Key Parameters**

| **Parameter** | **Description** |
|--------------|------------------|
| **path** | The mount point directory (e.g., `/data`). |
| **src** | The device to mount (e.g., `/dev/vg1/lv1`). |
| **fstype** | Filesystem type (must match what you created). |
| **opts** | Mount options (e.g., `defaults`, `noatime`). |
| **state** | `mounted` mounts + writes to `/etc/fstab`; `absent` unmounts + removes from `/etc/fstab`. |

### Mount the LV and Persist in /etc/fstab

```yaml
- name: Mount LV on /data
  mount:
    path: /data
    src: /dev/vg1/lv1
    fstype: xfs
    opts: defaults
    state: mounted
```

✔ Mounts the LV  
✔ Writes entry to `/etc/fstab`  
✔ Ensures it mounts on reboot  

