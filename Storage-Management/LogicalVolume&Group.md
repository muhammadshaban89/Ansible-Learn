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





