# 03-Backup-Recovery: Backup & Recovery

> **Target:** Minggu 7-8

---

## 📋 Kurikulum

### Logical Backup
- `mariadb-dump` / `mysqldump`: single-transaction, master-data, routines, triggers, events
- Parallel dump: `mariadb-dump --parallel` (MariaDB 10.6+)
- Restore: `mysql < dump.sql`, pv untuk progress, parallel restore
- Selective restore: database, table, where clause

### Physical Backup
- **MariaDB Backup** (Mariabackup) - fork dari Percona XtraBackup
- Full backup: `mariadb-backup --backup --target-dir=...`
- Incremental backup: `--incremental-basedir`, `--incremental-dir`
- Prepare: `--prepare` (replay redo log), `--prepare --apply-log-only`
- Restore: `--copy-back` / `--move-back`, permission fix (chown mysql:mysql)
- Streaming backup: `--stream=xbstream` ke remote/storage

### Point-in-Time Recovery (PITR)
- Binary log retention & purge policy
- `mariadb-binlog` / `mysqlbinlog`: read, filter by position/GTID/time
- Recovery workflow: Full backup → Prepare → Apply binlogs → Start server
- GTID-based replication & recovery (MariaDB 10.0+)

### Validasi & Testing Backup
- Automated restore test (staging server)
- Checksum validation: `CHECKSUM TABLE`, `pt-table-checksum`
- Backup monitoring & alerting

---

## 📁 Struktur Folder (Planned)

```
03-Backup-Recovery/
├── README.md                 # This file
├── 01-logical-backup/        # mariadb-dump/mysqldump
│   └── README.md
├── 02-physical-backup/       # mariabackup
│   └── README.md
├── 03-pitr/                  # Point-in-Time Recovery
│   └── README.md
├── 04-validation/            # Backup validation & testing
│   └── README.md
└── notes/                    # Personal notes
```

---

> **Previous:** [02-Manajemen-Database](../02-Manajemen-Database/)  
> **Next:** [04-Replikasi-HA](../04-Replikasi-HA/) - Minggu 9-11