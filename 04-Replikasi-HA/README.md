# 04-Replikasi-HA: Replikasi & High Availability

> **Target:** Minggu 9-11

---

## 📋 Kurikulum

### Replikasi Dasar
- Master-Slave (Source-Replica) setup: server_id, log_bin, gtid_domain_id
- GTID Replication: `@@gtid_current_pos`, `@@gtid_binlog_pos`, `@@gtid_slave_pos`
- CHANGE MASTER TO / CHANGE REPLICATION SOURCE TO (MariaDB 10.5+)
- Replication filters: replicate_do_db, replicate_ignore_db, replicate_wild_*
- Multi-source replication (MariaDB 10.0+)

### Replikasi Lanjutan
- Semi-synchronous replication: rpl_semi_sync_master_wait_point, timeout
- Delayed replication: MASTER_DELAY / SOURCE_DELAY
- Parallel replication: slave_parallel_threads / replica_parallel_threads
- Out-of-order parallel replication (MariaDB 10.1+)
- GTID strict mode, binlog_checksum

### Failover & HA Solutions
- **MaxScale** (MariaDB MaxScale): Read/Write Split, Automatic Failover (MariaDB Monitor)
- **MHA** (Master High Availability) - legacy
- **Orchestrator** - topology management, automated failover
- **ProxySQL** - connection pooling, query routing, read/write split
- **Galera Cluster** (MariaDB Galera): Multi-master synchronous replication
  - wsrep_provider, wsrep_cluster_address, wsrep_sst_method (mariabackup)
  - Flow control, SST vs IST, split-brain prevention
  - Weighted quorum, primary component

### Monitoring Replikasi
- Seconds_Behind_Master / Replica_Lag
- Relay log space, Slave_IO_Running, Slave_SQL_Running
- GTID consistency check: `SELECT GTID_SUBTRACT(...)`
- Alerting: lag > threshold, replication stopped, error 1236/1062

---

## 📁 Struktur Folder (Planned)

```
04-Replikasi-HA/
├── README.md                 # This file
├── 01-replikasi-dasar/       # Basic replication setup
│   └── README.md
├── 02-replikasi-lanjutan/    # Advanced replication
│   └── README.md
├── 03-failover-ha/           # Failover & HA solutions
│   └── README.md
├── 04-monitoring/            # Replication monitoring
│   └── README.md
└── notes/                    # Personal notes
```

---

> **Previous:** [03-Backup-Recovery](../03-Backup-Recovery/)  
> **Next:** [05-Security](../05-Security/) - Minggu 12