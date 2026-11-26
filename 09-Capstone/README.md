# 09-Capstone: Project Capstone

> **Target:** Minggu 18

---

## 📋 Kurikulum

### End-to-End Scenario
- Design production topology: Primary + 2 Replicas + MaxScale + Backup server
- Provision servers (Vagrant/Docker/Terraform/Ansible)
- Install MariaDB 11.4 LTS, configure my.cnf per best practices
- Setup GTID replication, verify failover dengan MaxScale
- Implement backup strategy: Daily full + Hourly incremental + PITR
- Load test dengan sysbench, tune berdasarkan hasil
- Document runbook: common operations, incident response, contacts
- Present architecture & decisions

---

## 📁 Struktur Folder (Planned)

```
09-Capstone/
├── README.md                 # This file
├── 01-topology-design/       # Production topology design
│   └── README.md
├── 02-provisioning/          # Infrastructure as Code
│   └── README.md
├── 03-install-config/        # MariaDB install & configuration
│   └── README.md
├── 04-replication-failover/  # GTID replication & MaxScale failover
│   └── README.md
├── 05-backup-strategy/       # Backup implementation
│   └── README.md
├── 06-load-test-tuning/      # Sysbench load test & tuning
│   └── README.md
├── 07-runbook/               # Operations runbook
│   └── README.md
├── 08-presentation/          # Final presentation
│   └── README.md
└── notes/                    # Personal notes
```

---

> **Previous:** [08-Monitoring](../08-Monitoring/)  
> **🎓 Curriculum Complete!**