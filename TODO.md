# MariaDB Administration - Kurikulum Beginner DBA

## 📋 Overview
Materi pembelajaran MariaDB Administration untuk **Beginner DBA** - dari instalasi hingga production-ready.

---

## 🎯 Phase 1: Fondasi & Instalasi (Minggu 1-2)

### 1.1 Arsitektur & Konsep Dasar
- [ ] Arsitektur MariaDB (Storage Engine, Connection Pool, Query Cache)
- [ ] Perbedaan MariaDB vs MySQL
- [ ] Storage Engine: InnoDB (default), MyISAM, Aria, ColumnStore, Spider
- [ ] ACID, MVCC, Locking mechanisms
- [ ] Proses: mysqld, background threads (redo log, checkpoint, purge)

### 1.2 Instalasi & Setup Awal
- [ ] Install via Repository (apt/yum/dnf) vs Binary Tarball vs Docker
- [ ] Verifikasi instalasi: `mysql_upgrade`, `mysql_secure_installation`
- [ ] Konfigurasi `my.cnf` / `my.ini` - lokasi & prioritas
- [ ] Systemd service management: start, stop, enable, status
- [ ] Multiple instances pada satu server (mysqld_multi)

### 1.3 Koneksi & Autentikasi
- [ ] User management: CREATE USER, DROP USER, RENAME USER
- [ ] Authentication plugins: mysql_native_password, caching_sha2_password, unix_socket, PAM, LDAP
- [ ] Host-based access control (%, localhost, IP specific)
- [ ] Password policies & expiration (validate_password plugin)
- [ ] SSL/TLS connections: generate certs, require SSL per user

---

## 🎯 Phase 2: Konfigurasi & Tuning Dasar (Minggu 3-4)

### 2.1 File Konfigurasi (my.cnf) - Parameter Kritis
- [ ] **Memory**: innodb_buffer_pool_size (70-80% RAM), innodb_log_file_size, innodb_log_buffer_size
- [ ] **Connections**: max_connections, thread_cache_size, max_connect_errors
- [ ] **InnoDB**: innodb_flush_log_at_trx_commit, innodb_flush_method, innodb_file_per_table
- [ ] **Query Cache**: query_cache_type, query_cache_size (deprecated di 10.6+)
- [ ] **Logging**: slow_query_log, long_query_time, log_queries_not_using_indexes
- [ ] **Binary Log**: log_bin, binlog_format (ROW/STATEMENT/MIXED), expire_logs_days, max_binlog_size
- [ ] **Temp/Temp Tables**: tmp_table_size, max_heap_table_size, innodb_temp_data_file_path

### 2.2 Monitoring & Status Server
- [ ] `SHOW GLOBAL STATUS`, `SHOW GLOBAL VARIABLES`
- [ ] `SHOW ENGINE INNODB STATUS` - interpretasi output
- [ ] `SHOW PROCESSLIST`, `SHOW FULL PROCESSLIST`
- [ ] Performance Schema: setup_instruments, setup_consumers, events_statements_summary
- [ ] Sys Schema: sys.schema_table_statistics, sys.user_summary
- [ ] Key metrics: QPS, TPS, Buffer Pool Hit Rate, Thread Running, Innodb_row_lock_waits

---

## 🎯 Phase 3: Manajemen Database & Objek (Minggu 5-6)

### 3.1 Database & Table Management
- [ ] CREATE/ALTER/DROP DATABASE, CHARACTER SET & COLLATION (utf8mb4, utf8mb4_unicode_ci)
- [ ] Table: CREATE TABLE, column types, constraints (PK, FK, UNIQUE, CHECK)
- [ ] Index: PRIMARY, UNIQUE, INDEX, FULLTEXT, SPATIAL - kapan gunakan yang mana
- [ ] Partitioning: RANGE, LIST, HASH, KEY - maintenance (ADD/DROP/REORGANIZE PARTITION)
- [ ] Generated Columns (VIRTUAL/STORED), JSON columns

### 3.2 DDL Online & Metadata Lock
- [ ] ALGORITHM=INPLACE vs COPY, LOCK=NONE/SHARED/EXCLUSIVE
- [ ] Online DDL di MariaDB 10.6+ (Instant ADD COLUMN)
- [ ] Metadata locking & deadlock prevention
- [ ] `pt-online-schema-change` (Percona Toolkit) untuk tabel besar

---

## 🎯 Phase 4: Backup & Recovery (Minggu 7-8)

### 4.1 Logical Backup
- [ ] `mariadb-dump` / `mysqldump`: single-transaction, master-data, routines, triggers, events
- [ ] Parallel dump: `mariadb-dump --parallel` (MariaDB 10.6+)
- [ ] Restore: `mysql < dump.sql`, pv untuk progress, parallel restore
- [ ] Selective restore: database, table, where clause

### 4.2 Physical Backup
- [ ] **MariaDB Backup** (Mariabackup) - fork dari Percona XtraBackup
- [ ] Full backup: `mariadb-backup --backup --target-dir=...`
- [ ] Incremental backup: `--incremental-basedir`, `--incremental-dir`
- [ ] Prepare: `--prepare` (replay redo log), `--prepare --apply-log-only`
- [ ] Restore: `--copy-back` / `--move-back`, permission fix (chown mysql:mysql)
- [ ] Streaming backup: `--stream=xbstream` ke remote/storage

### 4.3 Point-in-Time Recovery (PITR)
- [ ] Binary log retention & purge policy
- [ ] `mariadb-binlog` / `mysqlbinlog`: read, filter by position/GTID/time
- [ ] Recovery workflow: Full backup → Prepare → Apply binlogs → Start server
- [ ] GTID-based replication & recovery (MariaDB 10.0+)

### 4.4 Validasi & Testing Backup
- [ ] Automated restore test (staging server)
- [ ] Checksum validation: `CHECKSUM TABLE`, `pt-table-checksum`
- [ ] Backup monitoring & alerting

---

## 🎯 Phase 5: Replikasi & High Availability (Minggu 9-11)

### 5.1 Replikasi Dasar
- [ ] Master-Slave (Source-Replica) setup: server_id, log_bin, gtid_domain_id
- [ ] GTID Replication: `@@gtid_current_pos`, `@@gtid_binlog_pos`, `@@gtid_slave_pos`
- [ ] CHANGE MASTER TO / CHANGE REPLICATION SOURCE TO (MariaDB 10.5+)
- [ ] Replication filters: replicate_do_db, replicate_ignore_db, replicate_wild_*
- [ ] Multi-source replication (MariaDB 10.0+)

### 5.2 Replikasi Lanjutan
- [ ] Semi-synchronous replication: rpl_semi_sync_master_wait_point, timeout
- [ ] Delayed replication: MASTER_DELAY / SOURCE_DELAY
- [ ] Parallel replication: slave_parallel_threads / replica_parallel_threads
- [ ] Out-of-order parallel replication (MariaDB 10.1+)
- [ ] GTID strict mode, binlog_checksum

### 5.3 Failover & HA Solutions
- [ ] **MaxScale** (MariaDB MaxScale): Read/Write Split, Automatic Failover (MariaDB Monitor)
- [ ] **MHA** (Master High Availability) - legacy
- [ ] **Orchestrator** - topology management, automated failover
- [ ] **ProxySQL** - connection pooling, query routing, read/write split
- [ ] **Galera Cluster** (MariaDB Galera): Multi-master synchronous replication
  - [ ] wsrep_provider, wsrep_cluster_address, wsrep_sst_method (mariabackup)
  - [ ] Flow control, SST vs IST, split-brain prevention
  - [ ] Weighted quorum, primary component

### 5.4 Monitoring Replikasi
- [ ] Seconds_Behind_Master / Replica_Lag
- [ ] Relay log space, Slave_IO_Running, Slave_SQL_Running
- [ ] GTID consistency check: `SELECT GTID_SUBTRACT(...)`
- [ ] Alerting: lag > threshold, replication stopped, error 1236/1062

---

## 🎯 Phase 6: Security & Hardening (Minggu 12)

### 6.1 Access Control & Auditing
- [ ] Roles (MariaDB 10.0+): CREATE ROLE, GRANT ROLE, SET DEFAULT ROLE
- [ ] Column-level privileges, Table-level, Procedure-level
- [ ] **MariaDB Audit Plugin**: server_audit_events, server_audit_logging, output (file/syslog)
- [ ] **pam_authentication**, **ldap_authentication** untuk enterprise
- [ ] Data-at-rest encryption: innodb_encrypt_tables, key_management (file/keyring_vault/AWS KMS)
- [ ] In-transit encryption: SSL/TLS, require_secure_transport=ON

### 6.2 Hardening Checklist
- [ ] Remove anonymous users, test database
- [ ] Disable remote root login, bind-address = 127.0.0.1 (or specific IP)
- [ ] local_infile=OFF, secure_file_priv
- [ ] skip_show_database, skip_symbolic_links
- [ ] Run as non-root user, chroot/jail jika memungkinkan
- [ ] AppArmor/SELinux profiles

---

## 🎯 Phase 7: Performance Tuning Lanjutan (Minggu 13-14)

### 7.1 Query Optimization
- [ ] `EXPLAIN` / `EXPLAIN ANALYZE` (MariaDB 10.1+): type, key, rows, filtered, Extra
- [ ] Index strategies: covering index, prefix index, composite index order
- [ ] Optimizer hints: `/*+ MAX_EXECUTION_TIME(1000) */`, `/*+ JOIN_ORDER(t1 t2) */`
- [ ] Histogram statistics: ANALYZE TABLE, persistent statistics (innodb_stats_persistent)
- [ ] Slow query log analysis: `pt-query-digest`, `mysqldumpslow`

### 7.2 InnoDB Tuning Lanjutan
- [ ] Buffer Pool: instances, chunk_size, preload (innodb_buffer_pool_dump_at_shutdown)
- [ ] Redo log: capacity, sizing, checkpoint age monitoring
- [ ] Undo log: innodb_undo_tablespaces, innodb_max_undo_log_size, truncate
- [ ] Adaptive Hash Index: innodb_adaptive_hash_index, innodb_adaptive_hash_index_parts
- [ ] Change Buffer: innodb_change_buffering, innodb_change_buffer_max_size
- [ ] Purge threads: innodb_purge_threads, innodb_purge_batch_size

### 7.3 Concurrency & Locking
- [ ] Lock wait timeout: innodb_lock_wait_timeout
- [ ] Deadlock detection: innodb_deadlock_detect, SHOW ENGINE INNODB STATUS → LATEST DETECTED DEADLOCK
- [ ] Transaction isolation: READ COMMITTED vs REPEATABLE READ (default), binlog_format impact
- [ ] Long-running transactions: monitoring, killing (KILL QUERY vs KILL CONNECTION)

### 7.4 Capacity Planning
- [ ] Baseline metrics collection (Prometheus + Grafana / PMM)
- [ ] Growth forecasting: data size, QPS, connection count
- [ ] Hardware sizing: CPU, RAM, Disk (NVMe vs SSD vs HDD), Network
- [ ] Cloud vs On-premise considerations

---

## 🎯 Phase 8: Maintenance & Operations (Minggu 15-16)

### 8.1 Routine Maintenance
- [ ] `ANALYZE TABLE` - update statistics (kapan & seberapa sering)
- [ ] `OPTIMIZE TABLE` - reclaim space, defragment (online di 10.6+)
- [ ] `CHECK TABLE` / `REPAIR TABLE` (MyISAM/Aria)
- [ ] InnoDB: purge, merge change buffer, flush dirty pages
- [ ] Log rotation: binary log, slow log, error log, audit log (logrotate config)
- [ ] tmpdir cleanup, innodb_temp_tablespaces cleanup

### 8.2 Upgrade & Patching
- [ ] Minor upgrade (10.11.x → 10.11.y): in-place, mysql_upgrade
- [ ] Major upgrade (10.6 → 10.11 → 11.4): logical dump/restore atau replication-based
- [ ] `mysql_upgrade` / `mariadb-upgrade` - system tables, timezone tables
- [ ] Testing upgrade di staging, rollback plan
- [ ] Deprecated/removed features check per version

### 8.3 Troubleshooting Common Issues
- [ ] **Server won't start**: error log analysis, innodb_force_recovery (1-6), corrupted pages
- [ ] **High CPU**: runaway query, missing index, lock contention, too many threads
- [ ] **High Memory**: buffer pool oversized, memory leak (jemalloc), prepared statements not closed
- [ ] **Disk Full**: binary log purge, tmp tables, ibdata1 growth, undo tablespace
- [ ] **Replication Lag**: long-running DML on source, slow replica hardware, network
- [ ] **Corruption**: CHECKSUM TABLE, innochecksum, restore from backup

---

## 🎯 Phase 9: Monitoring & Observability (Minggu 17)

### 9.1 Tooling Stack
- [ ] **PMM (Percona Monitoring and Management)** - recommended all-in-one
- [ ] **Prometheus + Grafana** - custom dashboards, node_exporter, mysqld_exporter
- [ ] **Datadog / New Relic / CloudWatch** - managed observability
- [ ] **sysbench** - benchmarking: oltp_read_write, oltp_read_only, oltp_write_only
- [ ] **mysqlslap** - built-in load testing

### 9.2 Key Dashboards & Alerts
- [ ] **Overview**: QPS, TPS, Connections, Buffer Pool Hit%, Replication Lag
- [ ] **InnoDB**: Row operations, Log writes, Checkpoint age, History list length
- [ ] **Query Performance**: Slow queries, Top queries by latency/rows examined
- [ ] **System**: CPU, Memory, Disk I/O (IOPS, latency, throughput), Network
- [ ] **Alerts**: Disk > 80%, Replication lag > 30s, Connections > 80% max, Long query > 10s

---

## 🎯 Phase 10: Project Capstone (Minggu 18)

### 10.1 End-to-End Scenario
- [ ] Design production topology: Primary + 2 Replicas + MaxScale + Backup server
- [ ] Provision servers (Vagrant/Docker/Terraform/Ansible)
- [ ] Install MariaDB 11.4 LTS, configure my.cnf per best practices
- [ ] Setup GTID replication, verify failover dengan MaxScale
- [ ] Implement backup strategy: Daily full + Hourly incremental + PITR
- [ ] Load test dengan sysbench, tune berdasarkan hasil
- [ ] Document runbook: common operations, incident response, contacts
- [ ] Present architecture & decisions

---

## 📚 Referensi Utama

| Resource | Link |
|----------|------|
| MariaDB Knowledge Base | https://mariadb.com/kb/ |
| MariaDB Documentation (PDF) | https://mariadb.com/docs/ |
| InnoDB Internals Manual | https://dev.mysql.com/doc/internals/en/ |
| Percona Toolkit Docs | https://docs.percona.com/percona-toolkit/ |
| PMM Documentation | https://docs.percona.com/percona-monitoring-and-management/ |
| Galera Cluster Docs | https://galeracluster.com/library/documentation/ |
| MaxScale Documentation | https://mariadb.com/kb/en/mariadb-maxscale/ |

---

## ✅ Progress Tracking

```
Phase 1: ██████████ 100%  (Fondasi & Instalasi) ✓
Phase 2: ██████░░░░  60%  (Konfigurasi & Tuning)
Phase 3: ████░░░░░░  40%  (Manajemen Database)
Phase 4: ███░░░░░░░  30%  (Backup & Recovery)
Phase 5: ██░░░░░░░░  20%  (Replikasi & HA)
Phase 6: █░░░░░░░░░  10%  (Security)
Phase 7: ░░░░░░░░░░   0%  (Performance Tuning)
Phase 8: ░░░░░░░░░░   0%  (Maintenance)
Phase 9: ░░░░░░░░░░   0%  (Monitoring)
Phase 10: ░░░░░░░░░░  0%  (Capstone)
```

---

## 📝 Catatan Penggunaan

- **Checklist** (`[ ]`) = belum dikerjakan
- **Checklist** (`[x]`) = selesai
- **Bold** = topik prioritas/wajib
- Update progress tracking mingguan
- Setiap phase target 1-2 minggu (part-time) atau 3-5 hari (full-time)
- Buat note pribadi di folder `notes/` per phase