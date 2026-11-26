# Phase 1.1: Arsitektur & Konsep Dasar MariaDB

> **Target:** Minggu 1 (Hari 1-3)  
> **Status:** `[ ]` Belum mulai · `[ ]` Sedang dikerjakan · `[x]` Selesai

---

## 1. Arsitektur High-Level MariaDB

```
┌─────────────────────────────────────────────────────────────────┐
│                        CLIENT LAYER                             │
│  mysql CLI  │  Connectors (JDBC, ODBC, Python, PHP, Go, Node)  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                     CONNECTION HANDLING                         │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐  │
│  │ Thread Pool │  │  Auth       │  │  Connection Thread      │  │
│  │ (optional)  │  │  Plugins    │  │  (per connection)       │  │
│  └─────────────┘  └─────────────┘  └─────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      SQL LAYER (Server)                         │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐  │
│  │  Parser     │──▶│  Optimizer  │──▶│  Executor               │  │
│  │  (SQL →     │   │  (Cost-based│   │  (Execution plan      │  │
│  │   AST)      │   │   + RBO)    │   │   → Storage Engine)   │  │
│  └─────────────┘   └─────────────┘   └─────────────────────────┘  │
│         │                │                    │                   │
│         ▼                ▼                    ▼                   │
│  ┌─────────────────────────────────────────────────────────────┐  │
│  │                    QUERY CACHE (Deprecated 10.6+)           │  │
│  └─────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                   STORAGE ENGINE LAYER                          │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐           │
│  │ InnoDB   │ │ MyISAM   │ │ Aria     │ │ ColumnStore│ ...    │
│  │ (Default)│ │ (Legacy) │ │ (Crash-  │ │ (Analytics)│        │
│  │          │ │          │ │  safe)   │ │           │        │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘           │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                      PHYSICAL STORAGE                           │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐  │
│  │ Data Files  │  │ Redo Logs   │  │ Undo Logs               │  │
│  │ (.ibd,      │  │ (ib_logfile │  │ (undo tablespaces)      │  │
│  │  .MYD)      │  │  0/1)       │  │                         │  │
│  └─────────────┘  └─────────────┘  └─────────────────────────┘  │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐  │
│  │ Binary Logs │  │ Slow/Error  │  │ Temp Files              │  │
│  │ (binlog.xxx)│  │ Logs        │  │ (tmpdir)                │  │
│  └─────────────┘  └─────────────┘  └─────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Komponen Utama & Fungsinya

| Komponen | Fungsi | Catatan |
|----------|--------|---------|
| **Parser** | Validasi syntax, bangun AST (Abstract Syntax Tree) | `sql/sql_parse.cc` |
| **Optimizer** | Pilih execution plan terbaik (cost-based + rule-based) | `sql/opt_*.cc`, `EXPLAIN` output |
| **Executor** | Jalankan plan, panggil storage engine API | Handler interface (`handler.h`) |
| **Storage Engine** | Baca/tulis data fisik, indexing, locking, MVCC | Plugin architecture |
| **Connection Thread** | Satu thread per koneksi (thread-per-connection) | Atau thread pool |
| **Background Threads** | Lihat tabel di bawah | Non-blocking |

### Background Threads InnoDB (Penting!)

| Thread | Fungsi | Konfigurasi |
|--------|--------|-------------|
| **Master Thread** | Flush dirty pages, merge change buffer, purge undo, checkpoint | Utama |
| **IO Read Threads** | Async read dari disk ke buffer pool | `innodb_read_io_threads` (default 4) |
| **IO Write Threads** | Async write buffer pool ke disk | `innodb_write_io_threads` (default 4) |
| **Log Writer** | Tulis redo log ke disk (fsync) | `innodb_flush_log_at_trx_commit` |
| **Log Flusher** | Flush log buffer ke OS cache | |
| **Purge Thread(s)** | Hapus undo log lama (MVCC cleanup) | `innodb_purge_threads` (default 4) |
| **Page Cleaner** | Flush dirty pages background | `innodb_page_cleaners` (default 4) |

> **Latihan:** `SHOW ENGINE INNODB STATUS \G` → cari `BACKGROUND THREAD` section

---

## 3. Storage Engine Comparison

| Fitur | **InnoDB** (Default) | **MyISAM** | **Aria** | **ColumnStore** | **Spider** |
|-------|---------------------|------------|----------|-----------------|------------|
| **Transaksi (ACID)** | ✅ Full | ❌ | ✅ (Crash-safe) | ✅ | ✅ |
| **MVCC** | ✅ | ❌ | ✅ | ✅ | ✅ |
| **Row-level Lock** | ✅ | ❌ (Table) | ✅ | ✅ | ✅ |
| **Foreign Key** | ✅ | ❌ | ✅ | ❌ | ✅ |
| **Fulltext Search** | ✅ (10.0+) | ✅ | ✅ | ❌ | ❌ |
| **Compression** | ✅ (page/row) | ✅ (myisampack) | ✅ | ✅ (columnar) | Via remote |
| **Replication** | ✅ Native | ⚠️ Limited | ✅ | ✅ | ✅ (Sharding) |
| **Use Case** | **OLTP Umum** | Read-only, legacy | Tmp tables, crash-safe | **OLAP/Analytics** | **Federated/Sharding** |

> **Best Practice:** Gunakan **InnoDB** untuk 95%+ use case. MyISAM/Aria hanya untuk tabel internal (`mysql.*`) atau read-only archive.

---

## 4. Konsep Fundamental: ACID & MVCC

### ACID di InnoDB

| Property | Implementasi InnoDB |
|----------|---------------------|
| **Atomicity** | Undo log (rollback) + Redo log (crash recovery) |
| **Consistency** | Foreign key, CHECK constraint, triggers |
| **Isolation** | MVCC + Row locks (Next-key lock) |
| **Durability** | Redo log (WAL) + `innodb_flush_log_at_trx_commit=1` |

### MVCC (Multi-Version Concurrency Control)

```
TRANSACTION A (BEGIN)          TRANSACTION B (BEGIN)
       │                             │
       ▼                             │
  UPDATE row SET x=10           SELECT * FROM table
  (membuat version baru)           │
       │                         (melihat version LAMA - snapshot)
       ▼                             │
  COMMIT                            ▼
       │                         SELECT * FROM table
       ▼                          (melihat version BARU)
```

- **Read View**: Snapshot konsisten saat `SELECT` pertama (REPEATABLE READ) atau per statement (READ COMMITTED)
- **Undo Log**: Menyimpan version lama untuk rollback & MVCC read
- **Purge**: Hapus undo log yang tidak dibutuhkan lagi (history list length)

> **Latihan:** 
> ```sql
> -- Session 1
> BEGIN; UPDATE accounts SET balance=100 WHERE id=1;
> 
> -- Session 2 (REPEATABLE READ default)
> SELECT * FROM accounts WHERE id=1; -- masih balance lama
> 
> -- Session 1
> COMMIT;
> 
> -- Session 2
> SELECT * FROM accounts WHERE id=1; -- masih balance lama (snapshot)
> SELECT * FROM accounts WHERE id=1; -- balance baru (read view refresh di 10.2+)
> ```

### Isolation Levels

| Level | Dirty Read | Non-Repeatable Read | Phantom Read | Locking |
|-------|------------|---------------------|--------------|---------|
| **READ UNCOMMITTED** | ✅ | ✅ | ✅ | Minimal |
| **READ COMMITTED** | ❌ | ✅ | ✅ | Row locks only |
| **REPEATABLE READ** (Default) | ❌ | ❌ | ✅* | Next-key locks |
| **SERIALIZABLE** | ❌ | ❌ | ❌ | Gap locks + range |

> *InnoDB mencegah phantom read dengan **next-key locks** (record + gap) di REPEATABLE READ

---

## 5. Perbedaan MariaDB vs MySQL (Key Points)

| Area | MariaDB | MySQL (Oracle) |
|------|---------|----------------|
| **License** | GPLv2 (fully open) | GPLv2 + Commercial |
| **Storage Engines** | InnoDB, MyISAM, Aria, ColumnStore, Spider, MyRocks, S3, Connect, SphinxSE | InnoDB, MyISAM, NDB Cluster |
| **Optimizer** | Cost-based + histogram, window functions (10.2+), CTE | Cost-based, CTE (8.0+), histograms (8.0+) |
| **Replication** | GTID (10.0+), Multi-source (10.0+), Parallel (10.0+) | GTID (5.6+), Multi-source (8.0+), Parallel (5.7+) |
| **Galera Cluster** | Built-in (MariaDB Galera) | Separate (MySQL InnoDB Cluster) |
| **MaxScale** | Included (MariaDB MaxScale) | ProxySQL/Router separate |
| **JSON** | Functions + indexing (10.2+) | Native type + functions (5.7+) |
| **Window Functions** | 10.2+ | 8.0+ |
| **Instant ADD COLUMN** | 10.3+ (ALGORITHM=INSTANT) | 8.0.12+ |
| **System Versioning** | 10.3+ (Temporal Tables) | Not native |
| **Oracle Compatibility** | PL/SQL parser (10.3+), `||` concat | Limited |

> **Migrasi MySQL → MariaDB:** `mysql_upgrade` cukup untuk minor. Major butuh logical dump/restore.

---

## 6. File & Direktori Penting

| Path (Linux) | Fungsi |
|--------------|--------|
| `/etc/my.cnf` / `/etc/mysql/my.cnf` | Konfigurasi utama |
| `/etc/mysql/mariadb.conf.d/` | Drop-in configs (preferred) |
| `/var/lib/mysql/` | Data directory (datadir) |
| `/var/log/mysql/` | Error log, slow log, binary log |
| `/run/mysqld/mysqld.sock` | Unix socket |
| `/usr/lib/mysql/plugin/` | Storage engine & plugin .so files |
| `/usr/share/mysql/` | SQL scripts, charsets, timezone |

### Struktur Data Directory

```
/var/lib/mysql/
├── ibdata1              # System tablespace (data dictionary, undo log, change buffer)
├── ib_logfile0          # Redo log file 0
├── ib_logfile1          # Redo log file 1
├── ibtmp1               # Temporary tablespace
├── undo_001, undo_002   # Undo tablespaces (jika innodb_undo_tablespaces > 0)
├── mysql/               # System database (user, db, tables_priv, etc)
├── performance_schema/  # Performance schema tables
├── information_schema/  # Virtual (di memory)
├── db_name/             # User database
│   ├── table_name.ibd   # Per-table tablespace (innodb_file_per_table=ON)
│   └── table_name.frm   # Table definition (legacy, 10.4+ di SDI)
└── #innodb_temp/        # Internal temp tables
```

---

## 7. Praktikum: Eksplorasi Arsitektur

### 7.1 Cek Versi & Storage Engine
```sql
SELECT VERSION();                    -- versi server
SHOW ENGINES;                        -- list engine & support
SHOW VARIABLES LIKE 'default_storage_engine';
```

### 7.2 Lihat Background Threads
```sql
SHOW ENGINE INNODB STATUS \G
-- Cari section: BACKGROUND THREAD
-- Cari: SEMAPHORES, TRANSACTIONS, FILE I/O, INSERT BUFFER, LOG, BUFFER POOL
```

### 7.3 Cek Konfigurasi Data Directory
```sql
SHOW VARIABLES LIKE 'datadir';
SHOW VARIABLES LIKE 'innodb_data_file_path';
SHOW VARIABLES LIKE 'innodb_log_file%';
SHOW VARIABLES LIKE 'innodb_undo%';
```

### 7.4 Eksperimen MVCC
```sql
-- Terminal 1
CREATE TABLE test_mvcc (id INT PRIMARY KEY, val INT);
INSERT INTO test_mvcc VALUES (1, 100);
BEGIN;
UPDATE test_mvcc SET val = 200 WHERE id = 1;
-- JANGAN COMMIT

-- Terminal 2 (REPEATABLE READ default)
SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SELECT * FROM test_mvcc WHERE id = 1;  -- Expected: 100

-- Terminal 1
COMMIT;

-- Terminal 2
SELECT * FROM test_mvcc WHERE id = 1;  -- Expected: 100 (snapshot)
-- Di MariaDB 10.2+: SELECT lagi akan lihat 200 (read view refresh)
```

### 7.5 Cek History List Length (Purge Status)
```sql
SHOW ENGINE INNODB STATUS \G
-- Cari: "History list length"
-- Normal: < 1000. Tinggi = long-running tx atau purge lambat
```

---

## 8. Checklist Phase 1.1

- [ ] Bisa menjelaskan arsitektur 3-layer (Client, SQL, Storage Engine)
- [ ] Bisa menyebut minimal 5 background thread InnoDB & fungsinya
- [ ] Bisa membandingkan InnoDB vs MyISAM vs Aria vs ColumnStore
- [ ] Paham ACID di InnoDB: redo log, undo log, MVCC
- [ ] Bisa jelaskan 4 isolation level & perbedaan locking
- [ ] Tahu perbedaan utama MariaDB vs MySQL (minimal 5 poin)
- [ ] Bisa locate file penting: datadir, config, logs, socket
- [ ] Praktikum MVCC berhasil dijalankan & dipahami
- [ ] Bisa baca `SHOW ENGINE INNODB STATUS` section BACKGROUND THREAD & HISTORY LIST

---

## 9. Referensi Lanjutan

| Topik | Resource |
|-------|----------|
| InnoDB Internals | https://dev.mysql.com/doc/internals/en/ |
| MariaDB Architecture | https://mariadb.com/kb/en/mariadb-architecture/ |
| MVCC Deep Dive | https://mariadb.com/kb/en/innodb-multi-version-concurrency-control/ |
| Isolation Levels | https://mariadb.com/kb/en/transaction-isolation-levels/ |
| Storage Engine API | https://mariadb.com/kb/en/storage-engine-api/ |

---

> **Next:** [Phase 1.2: Instalasi & Setup Awal](../1.2-instalasi/instalasi-setup-awal.md)