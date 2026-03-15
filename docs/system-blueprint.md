# System Blueprint

**Purpose:** Current architecture truth for the Alethos Platform. Describes what exists, how it fits together, trust boundaries, and deployment model so that any future agent or operator can reconstruct the system.

**Last updated:** 2026-03-15 (TASK-0001 documentation sync)

---

## 1. Platform identity

- **Machine:** Lenovo ThinkStation P510 (Xeon E5-2640 v4, 32GB RAM).
- **OS:** Ubuntu Server LTS (headless, SSH-accessible).
- **Kubernetes:** k3s (not yet deployed; Phase 0 foundation in progress).
- **Role:** Single-node homelab designed as a "mini production cluster" via storage-tier and failure-domain separation.

---

## 2. Storage architecture

Storage is organized into four tiers. Only the **boot tier** is implemented as of TASK-0001; the others are reserved and will be configured in TASK-0002.

### 2.1 Boot tier (implemented)

**Status:** Implemented on alethos-node-01 (verified 2026-03-15).

**Hardware mapping (this machine):**

| Device | Model | Role |
|--------|--------|------|
| `/dev/sda` | Intel SSD Pro 1500 (~180GB) | Boot SSD 1 |
| `/dev/sdc` | Intel SSD 520 (~180GB) | Boot SSD 2 |
| `/dev/sdb` | WDC WD10SPZX 1TB HDD | Backup tier only — **not** part of boot; reserved for TASK-0002 `/backups` |

**Layout:**

- **Boot mode:** UEFI only (no Legacy/CSM).
- **RAID:** mdadm software RAID1 over the two Intel SATA SSDs only.
- **Arrays:**
  - `/dev/md0` → `/boot` (ext4, 2GiB) — members `sda2`, `sdc2`
  - `/dev/md1` → `/` (ext4, 50GiB) — members `sda3`, `sdc3`
  - `/dev/md2` → `/var` (ext4, remainder) — members `sda4`, `sdc4`
- **EFI:**
  - Primary: `/dev/sda1` (512MiB vfat) mounted at `/boot/efi`
  - Secondary: `/dev/sdc1` (512MiB vfat) — second ESP, same contents as primary; not mounted by default
- **GRUB:** Installed on both `/dev/sda` and `/dev/sdc` so the system can boot from either disk if one fails.

**Design rationale:** Boot tier is isolated from platform, database, and backup IO. `/var` is separate from `/` to limit blast radius from logs and system churn. No OS use of NVMe or HDD for root/boot/var.

**Failure domain:** Single node; one SSD failure is tolerated by RAID1; both disks are bootable.

### 2.2 Platform tier (reserved for TASK-0002)

- **Devices:** NVMe (e.g. `/dev/nvme0n1`, `/dev/nvme1n1`) — exact mapping to be confirmed in TASK-0002.
- **Mount:** `/platform` (and subdirs: rancher, containerd, kubelet, prometheus, loki).
- **Role:** k3s runtime, container images, observability stack storage. High-churn, write-heavy; isolated from OS and DB.

### 2.3 Database tier (reserved for TASK-0002)

- **Device:** 1TB NVMe (to be assigned in TASK-0002).
- **Mount:** `/data` (postgres, postgres_wal, redis).
- **Role:** PostgreSQL/PostGIS, Redis; latency-sensitive, isolated from platform churn.

### 2.4 Backup tier (reserved for TASK-0002)

- **Device:** `/dev/sdb` (1TB HDD).
- **Mount:** `/backups`.
- **Role:** Database dumps, snapshot exports, long-term retention; no OS or boot use.

**Critical:** TASK-0002 must partition and mount **only** `nvme0n1`, `nvme1n1`, and `sdb`. The second boot SSD is `sdc` — it must **not** be used for platform/data/backup.

---

## 3. Trust and operational boundaries

- **Boot tier:** OS and base system; RAID1 + dual ESP for resilience.
- **Platform/DB/Backup tiers:** Isolated from each other and from boot; failure in one tier does not take down the OS or other tiers.
- **Observability:** SMART enabled on boot disks; disk usage and RAID status visible via `lsblk`, `df -h`, `cat /proc/mdstat`, `mdadm --detail`. Metrics/alerting planned for later phases.

---

## 4. Runtime and deployment model (current)

- **Node:** Single physical host (alethos-node-01).
- **Services:** Ubuntu Server base, OpenSSH, base tooling (curl, git, htop, nvme-cli, smartmontools). k3s and workloads not yet deployed.
- **Network:** DHCP on primary NIC (e.g. eno1); static IP may be configured later.
- **Security baseline:** SSH enabled; password login temporarily allowed for setup; key-only SSH, firewall, and optional fail2ban deferred to later tasks.

---

## 5. References

- **Context:** `context/project-context.json` (storage_architecture, current_storage_layout, boot_raid1_partition_plan).
- **Build report:** `docs/build-reports/TASK-0001.md`.
- **Review:** `docs/build-reports/reviews/REVIEW-TASK-0001.md`.
