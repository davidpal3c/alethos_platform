# Final Expected Disk Layout (All Drives)

> **alethos-node-01 (post TASK-0001):** Boot RAID1 = **sda** + **sdc** (Intel SSDs). **1TB backup HDD = sdb** → use **`/dev/sdb1`** (or discovered partition) for `/backups`. **Do not** put backups on **sdc** (RAID member). TASK-0002 and `context/project-context.json` are authoritative for Phase 0B.

---

## Boot Tier — RAID1 SATA SSDs

Two drives:

- Intel SSD Pro 1500 — 180GB  
- Intel SSD 520 — 180GB  

These mirror the OS using **mdadm**. :contentReference[oaicite:0]{index=0}

---

# Use Dual EFI Partitions (Not One)

Earlier approach:
/dev/sda1 → /boot/efi
/dev/sdb1 → backup EFI

Better approach:
/dev/sda1 → EFI
/dev/sdb1 → EFI

Both disks contain their own **active EFI partition**, and **GRUB is installed on both disks**.

### Why this is better

- If disk A dies → system still boots from disk B  
- No manual EFI syncing required  
- Cleaner recovery  

This is the standard pattern used in **production Linux RAID boot setups**. :contentReference[oaicite:1]{index=1}

---

# Updated Boot Tier Layout

Per disk (`sda` and `sdb`):
512M EFI System Partition
2G RAID member (/boot)
50G RAID member (/)
rest RAID member (/var)

### RAID arrays
/dev/md0 → /boot
/dev/md1 → /
/dev/md2 → /var

### EFI
/dev/sda1 → /boot/efi
/dev/sdb1 → secondary EFI (bootable)

Both disks contain a **full bootloader**. :contentReference[oaicite:2]{index=2}

---

# Important Installer Tip

During Ubuntu install:

Assign **only one EFI partition**:
/boot/efi → sda1

Leave `sdb1` unused temporarily.

After installation, install GRUB on both disks.

---

# Post-Install Step (Critical)

After the first boot run:

```bash
sudo grub-install /dev/sda
sudo grub-install /dev/sdb
sudo update-grub
```

## Mount second EFI:
sudo mkdir /mnt/efi2
sudo mount /dev/sdb1 /mnt/efi2


## Copy bootloader:
sudo rsync -av /boot/efi/ /mnt/efi2/


## Unmount:
sudo umount /mnt/efi2
Now both disks are bootable. 

Resulting Boot Resilience
If /dev/sda fails:
BIOS loads EFI from /dev/sdb1
GRUB loads kernel
RAID continues degraded
System still boots
You can replace the failed disk later and rebuild RAID. 

Another Small Improvement (Very Worth It)
Before installing Ubuntu, wipe disks completely.
From installer shell (Ctrl + Alt + F2):
sudo wipefs -a /dev/sda
sudo wipefs -a /dev/sdb
This avoids:
old RAID metadata
ghost partitions
installer confusion 

Final Boot Tier Layout (Clean Version)
Disk sda
---------
sda1  512M   EFI
sda2  2G     RAID (/boot)
sda3  50G    RAID (/)
sda4  rest   RAID (/var)

Disk sdb
---------
sdb1  512M   EFI
sdb2  2G     RAID (/boot)
sdb3  50G    RAID (/)
sdb4  rest   RAID (/var)
RAID arrays
md0 → /boot
md1 → /
md2 → /var
This boot setup enforces the first failure domain: Boot tier resilience. 

Platform Tier — 240GB NVMe
Drive:
WD PC SN530 NVMe 240GB
Expected device:
/dev/nvme0n1
Partition layout:
/dev/nvme0n1p1
Filesystem:
ext4
Mount:
/platform
Platform Directories
/platform/rancher
/platform/kubelet
/platform/containerd
/platform/prometheus
/platform/loki
Bind mounts:
/platform/rancher      → /var/lib/rancher
/platform/kubelet      → /var/lib/kubelet
/platform/containerd   → /var/lib/containerd
/platform/prometheus   → /var/lib/prometheus
/platform/loki         → /var/lib/loki
Purpose:
high-write workloads
metrics
logs
container runtime
Isolation protects the OS disk. 

Database Tier — 1TB NVMe
Drive:
1TB NVMe
Expected device:
/dev/nvme1n1
Partition:
/dev/nvme1n1p1
Filesystem:
ext4
Mount:
/data
Database directories
/data/postgres
/data/postgres_wal
/data/redis
Purpose:
stateful workloads
low latency storage
DB + WAL isolation
Mount separation prepares for future disk-level WAL separation. 

Backup Tier — 1TB HDD
Drive:
1TB SATA HDD
Expected device:
/dev/sdc
Partition:
/dev/sdc1
Filesystem:
ext4
Mount:
/backups
Purpose:
database dumps
snapshot exports
retention archives
Backups must not compete with live workloads. 

Complete Mount Layout Summary
BOOT RAID
/dev/md0  → /boot
/dev/md1  → /
/dev/md2  → /var
EFI
/dev/sda1 → /boot/efi
/dev/sdb1 → backup EFI
PLATFORM TIER (240GB NVMe)
/dev/nvme0n1p1 → /platform
Bind mounts:
/platform/rancher      → /var/lib/rancher
/platform/kubelet      → /var/lib/kubelet
/platform/containerd   → /var/lib/containerd
/platform/prometheus   → /var/lib/prometheus
/platform/loki         → /var/lib/loki
DATABASE TIER (1TB NVMe)
/dev/nvme1n1p1 → /data
/data/postgres
/data/postgres_wal
/data/redis
BACKUP TIER (1TB HDD)
/dev/sdc1 → /backups
Visual Overview
            RAID1 BOOT TIER
      +---------------------------+
      | Intel SSD 180GB (sda)     |
      | Intel SSD 180GB (sdb)     |
      +---------------------------+

        md0 → /boot
        md1 → /
        md2 → /var


          PLATFORM NVMe
      +--------------------+
      | 240GB NVMe         |
      +--------------------+

        /platform
        ├─ rancher
        ├─ kubelet
        ├─ containerd
        ├─ prometheus
        └─ loki


           DATABASE NVMe
      +--------------------+
      | 1TB NVMe           |
      +--------------------+

        /data
        ├─ postgres
        ├─ postgres_wal
        └─ redis


           BACKUP HDD
      +--------------------+
      | 1TB HDD            |
      +--------------------+

        /backups
Why This Layout Is Excellent (Architecturally)
This layout enforces four separate failure domains:
OS
Platform runtime
Database
Backups
This mirrors mini production cluster architecture, far superior to typical homelab designs where everything shares one disk. 

The homelab is intentionally structured as a mini production platform with:
isolated storage tiers
failure-domain separation
recoverable infrastructure