# Task Queue

Execution-ready tasks derived from the roadmap.

---

# TASK-0001

Title: Ubuntu Server installation and partition strategy

Phase: Phase 0 — Platform Foundation

Status: Pending

Dependencies:
None

Goal:

Install Ubuntu Server on the boot tier and implement the RAID1 partition plan.

Scope:

• Install Ubuntu Server (headless)
• Configure mdadm RAID1 for boot drives
• Create partitions for:

  /boot/efi  
  /boot  
  /  
  /var

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

Title: Storage tier mount configuration

Phase: Phase 0 — Platform Foundation

Status: Pending

Dependencies:

TASK-0001

Goal:

Implement mountpoints for platform, database, and backup tiers.

Scope:

Platform tier mounts:

/var/lib/rancher  
/var/lib/containerd  
/var/lib/kubelet  
/var/lib/prometheus  
/var/lib/loki  

Database tier mounts:

/data/postgres  
/data/postgres_wal  
/data/redis  

Backup tier mount:

/backups

Non-scope:

filesystem snapshot strategy

Files affected:

docs/system-blueprint.md  
docs/platform-overview.md  
docs/build-reports/TASK-0002.md

Acceptance Criteria:

• mounts persist via fstab
• NVMe tiers isolated from OS
• backup tier accessible

Verification:

lsblk  
df -h  
mount

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