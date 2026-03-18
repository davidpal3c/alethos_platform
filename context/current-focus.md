# Current Focus

Project: Alethos Platform Homelab  
Phase: Phase 0 — Platform Foundation  
**Sub-phase (active): Phase 0B — Storage Architecture** ([roadmap](roadmap.md))

Phase 0A (Hardware + OS baseline) is **complete**: Ubuntu Server headless install, UEFI, mdadm RAID1 boot tier, dual ESP, SSH. **TASK-0001 is closed.**

The project is now focused on **implementing the remaining storage tiers** (platform, database, backup) and bind mounts per design—before k3s or workloads. That work is **TASK-0002**.

Roadmap alignment:

• **0A** — done (boot/OS)  
• **0B** — **now** (physical/logical mounts: `/platform`, `/data`, `/backups`, UUID `fstab`, bind mounts)  
• **0C** — next (k3s bootstrap after TASK-0002)

---

# Active Milestone

**Storage tier implementation (Phase 0B)**

Establish the four-tier storage architecture on the live node. Boot tier is already in place; remaining tiers:

BOOT TIER  
Done (TASK-0001): RAID1 SATA SSD mirror, `/boot` / `/` / `/var` on md devices.

PLATFORM TIER  
240GB NVMe → `/platform` + bind mounts to `/var/lib/*` (rancher, kubelet, containerd, prometheus, loki).

DATABASE TIER  
1TB NVMe → `/data` (`postgres`, `postgres_wal`, `redis`).

BACKUP TIER  
1TB HDD → `/backups` (isolated from live IO).

---

# Current Workstream

Platform Infrastructure

Focus areas (updated):

1. ~~Hardware preparation~~ / ~~Ubuntu install~~ — complete (0A)  
2. **Storage tier configuration (TASK-0002)** — immediate  
3. Kubernetes bootstrap (TASK-0003) — after TASK-0002  
4. Observability foundation — later phases  
5. Ingress — later phases  

---

# Active Tasks

**Completed**

TASK-0001 — Ubuntu Server installation + RAID1 boot mirror  

**Immediate focus**

TASK-0002 — Storage tier mount configuration ([tasks/TASK_0002.md](../tasks/TASK_0002.md), [tasks/todo.md](../tasks/todo.md))

**Queued (Phase 0C)**

TASK-0003 — k3s bootstrap  

**Later phases**

TASK-0004 — Observability stack baseline  
TASK-0005 — Ingress controller installation  

---

# Blocked Items

None currently.

Future items may depend on:

• filesystem strategy decision (ext4 vs ZFS)
• domain name acquisition for TLS

---

# Explicitly Deferred Work

The following are not allowed during Phase 0:

• Alethos API implementation
• Celery ingestion pipelines
• AI enrichment services
• dataset ingestion
• API feature work

These belong to later roadmap phases.

---

# Success Condition for Current Phase

The phase completes when the platform node can:

• run k3s reliably  
• expose HTTPS ingress  
• host observability stack  
• maintain tiered storage layout  
• support deployment of a test container workload


At that point the system transitions to:

Phase 1 — Observability & Deployment Discipline