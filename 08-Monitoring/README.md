# 08-Monitoring: Monitoring & Observability

> **Target:** Minggu 17

---

## 📋 Kurikulum

### Tooling Stack
- **PMM (Percona Monitoring and Management)** - recommended all-in-one
- **Prometheus + Grafana** - custom dashboards, node_exporter, mysqld_exporter
- **Datadog / New Relic / CloudWatch** - managed observability
- **sysbench** - benchmarking: oltp_read_write, oltp_read_only, oltp_write_only
- **mysqlslap** - built-in load testing

### Key Dashboards & Alerts
- **Overview**: QPS, TPS, Connections, Buffer Pool Hit%, Replication Lag
- **InnoDB**: Row operations, Log writes, Checkpoint age, History list length
- **Query Performance**: Slow queries, Top queries by latency/rows examined
- **System**: CPU, Memory, Disk I/O (IOPS, latency, throughput), Network
- **Alerts**: Disk > 80%, Replication lag > 30s, Connections > 80% max, Long query > 10s

---

## 📁 Struktur Folder (Planned)

```
08-Monitoring/
├── README.md                 # This file
├── 01-tooling-stack/         # PMM, Prometheus/Grafana, sysbench
│   └── README.md
├── 02-dashboards-alerts/     # Key dashboards & alerting rules
│   └── README.md
└── notes/                    # Personal notes
```

---

> **Previous:** [07-Maintenance](../07-Maintenance/)  
> **Next:** [09-Capstone](../09-Capstone/) - Minggu 18