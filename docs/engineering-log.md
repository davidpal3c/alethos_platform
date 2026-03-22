# Engineering Log

Chronological record of significant platform changes for traceability and context reload.

Format: **Date | Task ID | Summary of change | Architectural impact**

---

2026-03-15 | TASK-0001 | Ubuntu Server installation and RAID1 boot configuration completed on alethos-node-01. Boot tier: two Intel SATA SSDs (sda + sdc) in mdadm RAID1 (md0→/boot, md1→/, md2→/var), dual ESP, GRUB on both disks. NVMe and HDD (sdb) reserved for TASK-0002. | Boot tier implemented and isolated from platform/DB/backup tiers; storage tier separation established; foundation for Phase 0 storage mount task.

2026-03-15 | TASK-0002 | Storage tiers mounted on alethos-node-01: /platform on nvme1n1 (~238G), /data on nvme0n1 (~477G), /backups on sdb (~931G); ext4; UUID-only fstab lines with defaults,nofail; five bind mounts from /platform to /var/lib/{rancher,kubelet,containerd,prometheus,loki}; post-reboot validation per BR-TASK-0002. | Platform, database, and backup failure domains are live; high-churn paths off boot RAID; k3s-ready bind layout; doc sizes corrected from nominal 1TB DB to 512G-class hardware.
