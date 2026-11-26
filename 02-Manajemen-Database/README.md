# 02-Manajemen-Database: Manajemen Database & Objek

> **Target:** Minggu 5-6

---

## 📋 Kurikulum

### Database & Table Management
- CREATE/ALTER/DROP DATABASE, CHARACTER SET & COLLATION (utf8mb4, utf8mb4_unicode_ci)
- Table: CREATE TABLE, column types, constraints (PK, FK, UNIQUE, CHECK)
- Index: PRIMARY, UNIQUE, INDEX, FULLTEXT, SPATIAL - kapan gunakan yang mana
- Partitioning: RANGE, LIST, HASH, KEY - maintenance (ADD/DROP/REORGANIZE PARTITION)
- Generated Columns (VIRTUAL/STORED), JSON columns

### DDL Online & Metadata Lock
- ALGORITHM=INPLACE vs COPY, LOCK=NONE/SHARED/EXCLUSIVE
- Online DDL di MariaDB 10.6+ (Instant ADD COLUMN)
- Metadata locking & deadlock prevention
- `pt-online-schema-change` (Percona Toolkit) untuk tabel besar

---

## 📁 Struktur Folder (Planned)

```
02-Manajemen-Database/
├── README.md                 # This file
├── 01-database-table/        # Database & table management
│   └── README.md
├── 02-ddl-online/            # Online DDL & metadata lock
│   └── README.md
└── notes/                    # Personal notes
```

---

> **Previous:** [01-Konfigurasi](../01-Konfigurasi/)  
> **Next:** [03-Backup-Recovery](../03-Backup-Recovery/) - Minggu 7-8