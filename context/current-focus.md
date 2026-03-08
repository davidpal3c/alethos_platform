# Current Focus

Project: Alethos Platform Homelab  
Phase: Phase 0 — Platform Foundation

The project is currently focused on provisioning the base infrastructure node
for the homelab platform before any workloads or APIs are deployed.

This includes:

• Ubuntu Server installation  
• RAID1 boot tier configuration  
• Storage tier mounting  
• preparation for k3s platform workloads

---

# Active Milestone

Base Node Provisioning

The first objective is preparing the single-node infrastructure that will
host the Alethos platform.

This includes establishing the four-tier storage architecture:

BOOT TIER
RAID1 SATA SSD mirror

PLATFORM TIER
240GB NVMe mounted at /platform

DATABASE TIER
1TB NVMe mounted at /data

BACKUP TIER
1TB HDD mounted at /backups


---

# Current Workstream

Platform Infrastructure

Focus areas:

1. Hardware preparation
2. Storage tier configuration
3. Ubuntu Server installation
4. Kubernetes bootstrap
5. Observability foundation

---

# Active Tasks

Immediate focus:

TASK-0001 — Ubuntu Server installation and disk partition plan  
TASK-0002 — Storage tier mount layout  

Queued within current phase:

TASK-0003 — k3s bootstrap  

Later phases:

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