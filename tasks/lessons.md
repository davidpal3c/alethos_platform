# Lessons

This file records durable operational rules for the Alethos homelab platform.
It is the primary memory surface for the Lessons Agent described in `context/AGENTS.md`.

## Phase 0 — Platform Foundation

- Infrastructure tasks must include rollback procedure.  
- Never mix OS disk with high-churn workloads.  
- Do not expose internal services publicly by default.  
- Verify mountpoints after installation (for example: `lsblk`, `df -h`, `mount`).  
- Always confirm RAID rebuild procedures before relying on the array.  

