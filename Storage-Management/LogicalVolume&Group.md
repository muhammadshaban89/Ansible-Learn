The lvg and lvol Modules:
------------------------

- The `lvg` and `lvol` modules support the creation of logical volumes, including the configuration of physical volumes, and volume groups.
- The `lvg` takes as parameters the block devices to configure as the back end physical volumes for the volume group.
- The following table lists some of the parameters for the lvg module.

  # **LVM Volume Group Parameters (Ansible `lvg` module)**

| **Parameter** | **Meaning / Purpose** |
|--------------|------------------------|
| **pesize** | The **Physical Extent size** for the Volume Group. Must be a power of 2 or a multiple of **128 KiB**. Common values: `4M`, `8M`, `16M`. |
| **pvs** | A **list of devices** that will become **Physical Volumes**. Example: `/dev/sdb1, /dev/sdc1`. |
| **vg** | The **name of the Volume Group** you want to create or manage. Example: `vg_data`. |
| **state** | Whether the Volume Group should exist: `present` (create) or `absent` (remove). |

# **Example: Create a Volume Group**


```yaml
- name: Create volume group vg_data
  community.general.lvg:
    vg: vg_data
    pvs:
      - /dev/sdb1
      - /dev/sdc1
    pesize: 4M
    state: present
```
#  **Example: Remove a Volume Group**

```yaml
- name: Remove volume group vg_data
  community.general.lvg:
    vg: vg_data
    state: absent
```
- The `lvol` module creates logical volumes, and supports the resizing and shrinking of those volumes, and the filesystems on top of them.
- This module also supports the creation of snapshots for the logical volumes.
- Ansible lvol Module — Key Parameters Explained:

| **Parameter** | **Description** |
|---------------|------------------|
| **lv** | The **name of the logical volume** you want to create or manage. Example: `lv_data`. |
| **resizefs** | If `yes`, Ansible will **resize the filesystem** after resizing the LV. Works for growing and shrinking. |
| **shrink** | Allows shrinking the LV when set to `yes`. Disabled by default to prevent data loss. |
| **size** | The **size of the logical volume**. Supports units like `G`, `M`, `%FREE`. |
| **snapshot** | Creates a **snapshot LV** with the given name. Useful for backups or testing. |

Examples:
------------

##  **Create a Logical Volume**

```yaml
- name: Create logical volume lv_data
  community.general.lvol:
    vg: vg_data
    lv: lv_data
    size: 10G
    state: present
```

---

##  **Resize a Logical Volume (Grow)**

```yaml
- name: Grow logical volume to 20G
  community.general.lvol:
    vg: vg_data
    lv: lv_data
    size: 20G
    resizefs: yes
```

✔ LV grows  
✔ Filesystem grows automatically  

---

## **Shrink a Logical Volume (Careful!)**

```yaml
- name: Shrink logical volume to 5G
  community.general.lvol:
    vg: vg_data
    lv: lv_data
    size: 5G
    shrink: yes
    resizefs: yes
```

⚠️ Shrinking is dangerous — that’s why `shrink: yes` is required.

---

##  **Create a Snapshot**

```yaml
- name: Create snapshot of lv_data
  community.general.lvol:
    vg: vg_data
    lv: lv_data
    snapshot: lv_data_snap
    size: 2G
```

Creates a snapshot LV named `lv_data_snap`.

---

Examples:
---------

- Creates two partitions on /dev/sda, each 3 GiB
- Converts them into PVs
- Creates a VG
- Creates a 4 GiB LV
- Formats it with ext4
- Creates mount point /lv_mount
- Mounts it and persists in /etc/fstab

```yaml
- name: Create partitions, VG, LV, format and mount
  hosts: rhelnode
  become: yes

  tasks:

    # -------------------------------
    # 1. Create two partitions on /dev/sda
    # -------------------------------
    - name: Create partition 1 (3GiB)
      community.general.parted:
        device: /dev/sda
        number: 1
        state: present
        part_type: primary
        start: 1MiB
        end: 3073MiB

    - name: Create partition 2 (3GiB)
      community.general.parted:
        device: /dev/sda
        number: 2
        state: present
        part_type: primary
        start: 3073MiB
        end: 6145MiB

    # -------------------------------
    # 2. Create PVs
    # -------------------------------
    - name: Create PVs on partitions
      community.general.lvg:
        vg: vg_data
        pvs:
          - /dev/sda1
          - /dev/sda2
        pesize: 4M
        state: present

    # -------------------------------
    # 3. Create LV of 4GiB
    # -------------------------------
    - name: Create logical volume of 4GiB
      community.general.lvol:
        vg: vg_data
        lv: lv_data
        size: 4g
        state: present

    # -------------------------------
    # 4. Create ext4 filesystem
    # -------------------------------
    - name: Create ext4 filesystem on LV
      filesystem:
        fstype: ext4
        dev: /dev/vg_data/lv_data

    # -------------------------------
    # 5. Create mount point
    # -------------------------------
    - name: Create mount point
      file:
        path: /lv_mount
        state: directory
        mode: '0755'

    # -------------------------------
    # 6. Mount LV and persist in fstab
    # -------------------------------
    - name: Mount LV on /lv_mount
      mount:
        path: /lv_mount
        src: /dev/vg_data/lv_data
        fstype: ext4
        opts: defaults
        state: mounted

    - name: Persist mount in /etc/fstab
      mount:
        path: /lv_mount
        src: /dev/vg_data/lv_data
        fstype: ext4
        opts: defaults
        state: present
```
-  **Safely Extend LV to 5 GiB and Grow Filesystem**
```yaml
- name: Safely extend LV to 5GiB and grow filesystem
  hosts: rhelnode
  become: yes

  tasks:

    - name: Check if LV exists
      command: lvdisplay /dev/vg_data/lv_data
      register: lv_check
      ignore_errors: yes

    - name: Fail if LV does not exist
      fail:
        msg: "Logical volume /dev/vg_data/lv_data does not exist."
      when: lv_check.rc != 0

    - name: Extend LV to 5GiB and grow filesystem
      community.general.lvol:
        vg: vg_data
        lv: lv_data
        size: 5g
        resizefs: yes
        state: present

```
• 	Validation that the LV exists
• 	Validation that the filesystem is mounted or unmounted when required
• 	A safer shrink workflow (shrink must always be done carefully)


- **Safely Shrink LV to 3 GiB and Shrink Filesystem**

```yaml
- name: Safely shrink LV to 3GiB and shrink filesystem
  hosts: rhelnode
  become: yes

  tasks:

    - name: Check if LV exists
      command: lvdisplay /dev/vg_data/lv_data
      register: lv_check
      ignore_errors: yes

    - name: Fail if LV does not exist
      fail:
        msg: "Logical volume /dev/vg_data/lv_data does not exist."
      when: lv_check.rc != 0

    - name: Unmount filesystem before shrinking
      mount:
        path: /lv_mount
        state: unmounted

    - name: Shrink LV to 3GiB and shrink filesystem
      community.general.lvol:
        vg: vg_data
        lv: lv_data
        size: 3g
        shrink: yes
        resizefs: yes
        state: present

    - name: Remount filesystem
      mount:
        path: /lv_mount
        src: /dev/vg_data/lv_data
        fstype: ext4
        state: mounted

```
- shrink: yes → explicitly allows shrinking (otherwise Ansible refuses)
- resizefs: yes → shrinks filesystem safely with LV

