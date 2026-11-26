# 07-Maintenance: Maintenance & Operations

> **Target:** Minggu 15-16

---

## 📋 Kurikulum

### Routine Maintenance
- `ANALYZE TABLE` - update statistics (kapan & seberapa sering)
- `OPTIMIZE TABLE` - reclaim space, defragment (online di 10.6+)
- `CHECK TABLE` / `REPAIR TABLE` (MyISAM/Aria)
- InnoDB: purge, merge change buffer, flush dirty pages
- Log rotation: binary log, slow log, error log, audit log (logrotate config)
- tmpdir cleanup, innodb_temp_tablespaces cleanup

### Upgrade & Patching
- Minor upgrade (10.11.x → 10.11.y): in-place, mysql_upgrade
- Major upgrade (10.6 → 10.11 → 11.4): logical dump/restore atau replication-based
- `mysql_upgrade` / `mariadb-upgrade` - system tables, timezone tables
- Testing upgrade di staging, rollback plan
- Deprecated/removed features check per version

### Troubleshooting Common Issues
- **Server won't start**: error log analysis, innodb_force_recovery (1-6), corrupted pages
- **High CPU**: runaway query, missing index, lock contention, too many threads
- **High Memory**: buffer pool oversized, memory leak (jemalloc), prepared statements not closed
- **Disk Full**: binary log purge, tmp tables, ibdata1 growth, undo tablespace
- **Replication Lag**: long-running DML on source, slow replica hardware, network
- **Corruption**: CHECKSUM TABLE, innochecksum, restore from backup

---

## 📁 Struktur Folder (Planned)

```
07-Maintenance/
├── README.md                 # This file
├── 01-routine-maintenance/   # ANALYZE, OPTIMIZE, CHECK, log rotation
│   └── README.md
├── 02-upgrade-patching/      # Minor/major upgrade procedures
│   └── README.md
├── 03-troubleshooting/       # Common issues & solutions
│   └── README.md
└── notes/                    # Personal notes
```

---

> **Previous:** [06-Performance-Tuning](../06-Performance-Tuning/)  
> **Next:** [08-Monitoring](../08-Monitoring/) - Minggu 17