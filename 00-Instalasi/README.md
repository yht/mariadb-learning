# 00-Instalasi

> **Duration:** Minggu 1-2 (7 hari)

---

## 📁 Structure Created

```
00-Instalasi/
├── 00-arsitektur/
│   └── README.md      # Arsitektur, Storage Engine, ACID, MVCC, InnoDB internals
├── 01-instalasi/
│   └── README.md         # Install (repo, tarball, docker), secure, my.cnf, systemd, upgrade
└── 02-autentikasi/
    └── README.md          # User+Host, Auth plugins, Privileges, Roles, SSL/TLS, Monitoring
```

---

## 📋 Kurikulum

### Arsitektur & Konsep Dasar
- Arsitektur 3-layer (Client, SQL, Storage Engine) + background threads
- Storage Engine comparison: InnoDB vs MyISAM vs Aria vs ColumnStore vs Spider
- ACID di InnoDB: Redo log, Undo log, MVCC
- 4 Isolation Levels & locking behavior
- MariaDB vs MySQL perbedaan kunci (minimal 5 poin)
- File & direktori penting: datadir, config, logs, socket
- Praktikum: MVCC experiment, `SHOW ENGINE INNODB STATUS` reading

### Instalasi & Setup Awal
- Install via Repository (Ubuntu/Debian, RHEL/Rocky/Alma, openSUSE)
- Install via Binary Tarball (multiple instances, non-root)
- Install via Docker (development/testing)
- `mariadb-secure-installation` walkthrough
- Config file priority & modern structure (`mariadb.conf.d/99-custom.cnf`)
- Production-ready `my.cnf` template dengan parameter kritis
- Systemd management: start/stop/restart/enable/logs/override limits
- Multiple instances: `mysqld_multi` & systemd template
- Minor upgrade procedure & post-upgrade checklist
- Config validation dengan `--validate-config`
- Troubleshooting common install errors

### Koneksi & Autentikasi
- User = User + Host concept & host matching priority
- 8 Authentication plugins: native, caching_sha2, unix_socket, PAM, LDAP, GSSAPI, ed25519, clear_password
- User CRUD: CREATE/ALTER/RENAME/DROP USER dengan opsi lengkap
- Password policies: validate_password, expiration, history, reuse, failed login tracking
- Privilege hierarchy: Global → Database → Table → Column → Procedure
- Grant patterns: read-only, read-write, admin, replication, backup
- Roles (RBAC): create, grant to role, grant role to user, default role, nested roles
- SSL/TLS: Generate certs (OpenSSL), server/client config, REQUIRE SSL/X509
- Connection types: Unix socket vs TCP/IP vs Thread pool
- Connection pooling (app side: Python, Java/HikariCP)
- Monitoring: SHOW PROCESSLIST, performance_schema.threads, kill queries
- Troubleshooting auth errors

---

## 🎯 Key Commands Mastered

| Category | Commands |
|----------|----------|
| **Install** | `apt/dnf install mariadb-server`, `mariadb-install-db`, `mariadb-secure-installation` |
| **Config** | `mariadbd --validate-config`, `systemctl edit mariadb`, `mysqld_multi` |
| **User Mgmt** | `CREATE USER`, `ALTER USER`, `GRANT/REVOKE`, `CREATE ROLE`, `SET DEFAULT ROLE` |
| **Auth** | `INSTALL PLUGIN validate_password`, `INSTALL SONAME 'auth_pam'` |
| **SSL** | `openssl req/genrsa/x509`, `REQUIRE SSL`, `--ssl-mode=REQUIRED` |
| **Monitor** | `SHOW PROCESSLIST`, `performance_schema.threads`, `KILL QUERY/CONNECTION/USER` |
| **Status** | `SHOW ENGINE INNODB STATUS`, `SHOW VARIABLES`, `SHOW STATUS` |

---

## 📚 Reference Files Created

| File | Lines | Key Content |
|------|-------|-------------|
| `00-arsitektur/arsitektur-konsep-dasar.md` | ~550 | Architecture diagram, InnoDB threads, MVCC, isolation levels, MySQL vs MariaDB |
| `01-instalasi/instalasi-setup-awal.md` | ~750 | 3 install methods, secure install, my.cnf template, systemd, multi-instance, upgrade |
| `02-autentikasi/koneksi-autentikasi.md` | ~950 | Auth plugins, user mgmt, passwords, privileges, roles, SSL, pooling, monitoring |

**Total: ~2,250 lines of comprehensive learning material**
