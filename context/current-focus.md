# Current Focus

Project: Alethos Platform Homelab  
Phase: Phase 0 — Platform Foundation

---

# Active Milestone

Build a **stable platform node** capable of:

• running Kubernetes (k3s)
• hosting observability stack
• providing ingress routing
• storing state across defined storage tiers
• supporting future workload deployment

The platform must reach a state where it behaves like a **single-node private cloud**.

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

TASK-0001 — Ubuntu Server installation and disk partition plan  
TASK-0002 — Storage tier mount layout  
TASK-0003 — k3s bootstrap  
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