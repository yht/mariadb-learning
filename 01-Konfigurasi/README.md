# 01-Konfigurasi: Konfigurasi & Tuning Dasar

> **Target:** Minggu 3-4

---

## 📋 Kurikulum

### File Konfigurasi (my.cnf) - Parameter Kritis
- **Memory**: innodb_buffer_pool_size (70-80% RAM), innodb_log_file_size, innodb_log_buffer_size
- **Connections**: max_connections, thread_cache_size, max_connect_errors
- **InnoDB**: innodb_flush_log_at_trx_commit, innodb_flush_method, innodb_file_per_table
- **Query Cache**: query_cache_type, query_cache_size (deprecated di 10.6+)
- **Logging**: slow_query_log, long_query_time, log_queries_not_using_indexes
- **Binary Log**: log_bin, binlog_format (ROW/STATEMENT/MIXED), expire_logs_days, max_binlog_size
- **Temp/Temp Tables**: tmp_table_size, max_heap_table_size, innodb_temp_data_file_path

### Monitoring & Status Server
- `SHOW GLOBAL STATUS`, `SHOW GLOBAL VARIABLES`
- `SHOW ENGINE INNODB STATUS` - interpretasi output
- `SHOW PROCESSLIST`, `SHOW FULL PROCESSLIST`
- Performance Schema: setup_instruments, setup_consumers, events_statements_summary
- Sys Schema: sys.schema_table_statistics, sys.user_summary
- Key metrics: QPS, TPS, Buffer Pool Hit Rate, Thread Running, Innodb_row_lock_waits

---

## 📁 Struktur Folder (Planned)

```
01-Konfigurasi/
├── README.md                 # This file
├── 01-mycnf-parameter/       # Parameter my.cnf deep dive
│   └── README.md
├── 02-monitoring-status/     # Monitoring & status server
│   └── README.md
└── notes/                    # Personal notes
```

---

> **Next:** [02-Manajemen-Database](../02-Manajemen-Database/) - Minggu 5-6