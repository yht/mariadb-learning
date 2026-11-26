# 06-Performance-Tuning: Performance Tuning Lanjutan

> **Target:** Minggu 13-14

---

## 📋 Kurikulum

### Query Optimization
- `EXPLAIN` / `EXPLAIN ANALYZE` (MariaDB 10.1+): type, key, rows, filtered, Extra
- Index strategies: covering index, prefix index, composite index order
- Optimizer hints: `/*+ MAX_EXECUTION_TIME(1000) */`, `/*+ JOIN_ORDER(t1 t2) */`
- Histogram statistics: ANALYZE TABLE, persistent statistics (innodb_stats_persistent)
- Slow query log analysis: `pt-query-digest`, `mysqldumpslow`

### InnoDB Tuning Lanjutan
- Buffer Pool: instances, chunk_size, preload (innodb_buffer_pool_dump_at_shutdown)
- Redo log: capacity, sizing, checkpoint age monitoring
- Undo log: innodb_undo_tablespaces, innodb_max_undo_log_size, truncate
- Adaptive Hash Index: innodb_adaptive_hash_index, innodb_adaptive_hash_index_parts
- Change Buffer: innodb_change_buffering, innodb_change_buffer_max_size
- Purge threads: innodb_purge_threads, innodb_purge_batch_size

### Concurrency & Locking
- Lock wait timeout: innodb_lock_wait_timeout
- Deadlock detection: innodb_deadlock_detect, SHOW ENGINE INNODB STATUS → LATEST DETECTED DEADLOCK
- Transaction isolation: READ COMMITTED vs REPEATABLE READ (default), binlog_format impact
- Long-running transactions: monitoring, killing (KILL QUERY vs KILL CONNECTION)

### Capacity Planning
- Baseline metrics collection (Prometheus + Grafana / PMM)
- Growth forecasting: data size, QPS, connection count
- Hardware sizing: CPU, RAM, Disk (NVMe vs SSD vs HDD), Network
- Cloud vs On-premise considerations

---

## 📁 Struktur Folder (Planned)

```
06-Performance-Tuning/
├── README.md                 # This file
├── 01-query-optimization/    # EXPLAIN, indexes, hints
│   └── README.md
├── 02-innodb-tuning/         # InnoDB advanced tuning
│   └── README.md
├── 03-concurrency-locking/   # Locking, deadlocks, isolation
│   └── README.md
├── 04-capacity-planning/     # Hardware & growth planning
│   └── README.md
└── notes/                    # Personal notes
```

---

> **Previous:** [05-Security](../05-Security/)  
> **Next:** [07-Maintenance](../07-Maintenance/) - Minggu 15-16