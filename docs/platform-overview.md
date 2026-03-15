# Platform Overview

**Purpose:** High-level view of the Alethos Platform storage and boot layout, tier separation, and operational model. Enables future agents and operators to understand platform structure and failure domains without reading implementation detail.

**Last updated:** 2026-03-15 (TASK-0001 documentation sync)

---

## 1. What the platform is

The Alethos Platform runs on a **single-node homelab** (Lenovo ThinkStation P510) but is designed like a **mini production cluster**: storage and IO are separated by tier so that failure domains are clear and blast radius is minimized.

- **OS and boot** live on a dedicated RAID1 SATA SSD tier (two Intel 180GB SSDs).
- **Platform workloads** (k3s runtime, observability) will use NVMe (`/platform`).
- **Databases** (PostgreSQL, Redis) will use a separate NVMe tier (`/data`).
- **Backups** will use a 1TB HDD (`/backups`).

This separation ensures that high-churn workload IO does not compete with the OS or database, and that a full disk or runaway logs in one tier do not take down the whole system.

---

## 2. Storage and boot layout (current state)

### Implemented (TASK-0001)

- **Boot tier:** Two Intel SATA SSDs in mdadm RAID1. Mountpoints: `/boot/efi`, `/boot`, `/`, `/var`. Dual EFI and GRUB on both disks for boot resilience. No use of NVMe or HDD for OS.
- **Tier separation:** Boot tier is isolated; NVMe and HDD are unpartitioned and reserved for the next task.

### Planned (TASK-0002)

- **Platform tier:** NVMe → `/platform` and subdirs (rancher, containerd, kubelet, prometheus, loki); bind mounts from `/var/lib/*` for k3s and observability.
- **Database tier:** NVMe → `/data` (postgres, postgres_wal, redis).
- **Backup tier:** HDD → `/backups`.

Only **nvme0n1**, **nvme1n1**, and the **1TB HDD (sdb)** are in scope for TASK-0002. The second boot SSD (**sdc**) must not be used for platform, data, or backups.

---

## 3. Failure domains and operational expectations

| Tier       | Failure domain        | Operational expectation                          |
|-----------|------------------------|--------------------------------------------------|
| Boot      | Single node; RAID1    | One SSD failure: system stays up; boot from other SSD. |
| Platform  | Single node; one NVMe | If full or degraded: workloads affected; OS and DB intact. |
| Database  | Single node; one NVMe | Isolated from platform churn; recovery via backups/WAL. |
| Backup    | Single node; one HDD  | If disk fails: restore from off-host copy (future). |

Observability (disk usage, SMART, RAID status) is in place for the boot tier; metrics and alerting are planned for later phases.

---

## 4. References

- **Architecture detail:** `docs/system-blueprint.md`
- **Structured context:** `context/project-context.json`
- **Task and verification:** `docs/build-reports/TASK-0001.md`, `docs/build-reports/reviews/REVIEW-TASK-0001.md`
