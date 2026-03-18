# Task Queue

Execution-ready tasks derived from the roadmap.

---

# TASK-0001

Title: Ubuntu Server installation + RAID1 boot mirror

Phase: Phase 0 — Platform Foundation

Status: Complete

Dependencies:
None

Goal:

Install Ubuntu Server on the boot tier and implement the UEFI + mdadm RAID1 boot mirror partition plan.

Scope:

• Install Ubuntu Server (headless)
• Configure mdadm RAID1 for boot drives in UEFI mode
• Create identical GPT partitions on both SSDs for:

  /boot/efi  
  /boot  
  /  
  /var

• Create mdadm RAID1 arrays:

  /dev/md0 → /boot  
  /dev/md1 → /  
  /dev/md2 → /var

• Install GRUB on both disks
• Verify boot resilience

Non-scope:

• Kubernetes
• observability
• workload containers

Files / Docs affected:

context/project-context.json  
docs/platform-overview.md  
docs/system-blueprint.md  
docs/build-reports/TASK-0001.md

Acceptance Criteria:

• System boots from RAID1 mirror
• mountpoints match design
• SSH access working
• OS disk usage monitored

Verification Steps:

• lsblk
• cat /proc/mdstat
• df -h
• reboot test

---

# TASK-0002

Title: Storage tier mount configuration (Phase 0B)

Phase: Phase 0 — Platform Foundation (sub-phase **0B — Storage Architecture**)

Status: **Ready**

Why now:

0A is complete; k3s (0C) must not run container/runtime data on the boot RAID. Platform, database, and backup tiers must exist with UUID `fstab` and bind mounts before TASK-0003.

Dependencies:

TASK-0001 (complete)

Goal:

Implement physical/logical storage for platform, database, and backup tiers on **alethos-node-01**, aligned with `context/project-context.json`.

Scope:

• **Discover** block devices with `lsblk -f` / `nvme list` (names can vary); confirm before destructive steps.  
• **Platform tier:** single partition on 240GB WD SN530 NVMe → ext4 → `/platform`; create subdirs; bind-mount to `/var/lib/rancher`, `kubelet`, `containerd`, `prometheus`, `loki`.  
• **Database tier:** single partition on 1TB NVMe → ext4 → `/data`; create `/data/postgres`, `postgres_wal`, `redis`.  
• **Backup tier:** partition 1TB HDD → ext4 → `/backups`.  
  **Live mapping (TASK-0001):** RAID boot = **sda + sdc**; **backup HDD = sdb** — do **not** use sdc for backups (sdc is RAID member).

Non-scope:

• k3s install, ZFS/snapshot policy, backup automation, observability install.

Risks:

• Wrong disk selection can destroy RAID or OS — verify UUIDs and model/size before `mkfs`.  
• Bind mounts before k3s: empty dirs OK; ensure `/etc/fstab` ordering (base mount before bind).

Files / docs affected:

`context/project-context.json` (post-verify)  
`docs/system-blueprint.md`  
`docs/platform-overview.md`  
`docs/build-reports/TASK-0002.md`

Acceptance Criteria:

• `/platform`, `/data`, `/backups` mounted at boot via UUID in `/etc/fstab`.  
• All five bind targets present and mounted after reboot.  
• `df -h` shows expected capacities; boot RAID untouched.

Verification:

`lsblk -f` · `findmnt` · `mount | grep -E 'platform|/data|backups'` · reboot test · optional `sudo systemd-analyze blame` if boot delay

Builder output:

Operator runbook (partition, mkfs, fstab, bind mounts, verification); no execution on node by Builder unless operator delegates.

---

# TASK-0003

Title: k3s cluster bootstrap

Phase: Phase 0 — Platform Foundation

Status: Pending

Dependencies:

TASK-0002

Goal:

Deploy Kubernetes runtime using k3s.

Scope:

• install k3s
• relocate runtime directories to platform tier
• configure namespaces

Non-scope:

TLS  
observability stack

Files affected:

platform/k3s/cluster-setup.md  
docs/system-blueprint.md  
docs/build-reports/TASK-0003.md

Acceptance Criteria:

• kubectl get nodes returns Ready
• cluster networking functional
• persistent volumes usable

Verification:

kubectl get nodes  
kubectl get pods -A

---

# TASK-0004

Title: Observability stack baseline

Phase: Phase 1 — Observability Discipline

Status: Pending

Dependencies:

TASK-0003

Goal:

Deploy Prometheus + Grafana + Loki stack.

Scope:

• Prometheus deployment
• Grafana dashboards
• Loki logging
• node-exporter metrics

Acceptance Criteria:

• metrics visible in Grafana
• disk usage metrics visible
• logs ingested

Verification:

Grafana dashboards functional.

---

# TASK-0005

Title: Ingress controller + TLS

Phase: Phase 2 — Deployment Discipline

Status: Pending

Dependencies:

TASK-0003

Goal:

Establish public traffic routing.

Scope:

• ingress controller installation
• TLS certificate management
• domain routing

Acceptance Criteria:

• HTTPS endpoint accessible
• routing to internal services works

Verification:

curl https://domain/health