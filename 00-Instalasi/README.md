# 00-Instalasi

> **Duration:** Minggu 1-2 (7 hari)  
> **Status:** ✅ **COMPLETED**

---

## 📁 Structure Created

```
00-Instalasi/
├── 00-arsitektur/
│   └── arsitektur-konsep-dasar.md      # Arsitektur, Storage Engine, ACID, MVCC, InnoDB internals
├── 01-instalasi/
│   └── instalasi-setup-awal.md         # Install (repo, tarball, docker), secure, my.cnf, systemd, upgrade
└── 02-autentikasi/
    └── koneksi-autentikasi.md          # User+Host, Auth plugins, Privileges, Roles, SSL/TLS, Monitoring
```

---

## ✅ Checklist

### Arsitektur & Konsep Dasar
- [x] Arsitektur 3-layer (Client, SQL, Storage Engine) + background threads
- [x] Storage Engine comparison: InnoDB vs MyISAM vs Aria vs ColumnStore vs Spider
- [x] ACID di InnoDB: Redo log, Undo log, MVCC
- [x] 4 Isolation Levels & locking behavior
- [x] MariaDB vs MySQL perbedaan kunci (minimal 5 poin)
- [x] File & direktori penting: datadir, config, logs, socket
- [x] Praktikum: MVCC experiment, `SHOW ENGINE INNODB STATUS` reading

### Instalasi & Setup Awal
- [x] Install via Repository (Ubuntu/Debian, RHEL/Rocky/Alma, openSUSE)
- [x] Install via Binary Tarball (multiple instances, non-root)
- [x] Install via Docker (development/testing)
- [x] `mariadb-secure-installation` walkthrough
- [x] Config file priority & modern structure (`mariadb.conf.d/99-custom.cnf`)
- [x] Production-ready `my.cnf` template dengan parameter kritis
- [x] Systemd management: start/stop/restart/enable/logs/override limits
- [x] Multiple instances: `mysqld_multi` & systemd template
- [x] Minor upgrade procedure & post-upgrade checklist
- [x] Config validation dengan `--validate-config`
- [x] Troubleshooting common install errors

### Koneksi & Autentikasi
- [x] User = User + Host concept & host matching priority
- [x] 8 Authentication plugins: native, caching_sha2, unix_socket, PAM, LDAP, GSSAPI, ed25519, clear_password
- [x] User CRUD: CREATE/ALTER/RENAME/DROP USER dengan opsi lengkap
- [x] Password policies: validate_password, expiration, history, reuse, failed login tracking
- [x] Privilege hierarchy: Global → Database → Table → Column → Procedure
- [x] Grant patterns: read-only, read-write, admin, replication, backup
- [x] Roles (RBAC): create, grant to role, grant role to user, default role, nested roles
- [x] SSL/TLS: Generate certs (OpenSSL), server/client config, REQUIRE SSL/X509
- [x] Connection types: Unix socket vs TCP/IP vs Thread pool
- [x] Connection pooling (app side: Python, Java/HikariCP)
- [x] Monitoring: SHOW PROCESSLIST, performance_schema.threads, kill queries
- [x] Troubleshooting auth errors

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
