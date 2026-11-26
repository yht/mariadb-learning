# 05-Security: Security & Hardening

> **Target:** Minggu 12

---

## 📋 Kurikulum

### Access Control & Auditing
- Roles (MariaDB 10.0+): CREATE ROLE, GRANT ROLE, SET DEFAULT ROLE
- Column-level privileges, Table-level, Procedure-level
- **MariaDB Audit Plugin**: server_audit_events, server_audit_logging, output (file/syslog)
- **pam_authentication**, **ldap_authentication** untuk enterprise
- Data-at-rest encryption: innodb_encrypt_tables, key_management (file/keyring_vault/AWS KMS)
- In-transit encryption: SSL/TLS, require_secure_transport=ON

### Hardening Checklist
- Remove anonymous users, test database
- Disable remote root login, bind-address = 127.0.0.1 (or specific IP)
- local_infile=OFF, secure_file_priv
- skip_show_database, skip_symbolic_links
- Run as non-root user, chroot/jail jika memungkinkan
- AppArmor/SELinux profiles

---

## 📁 Struktur Folder (Planned)

```
05-Security/
├── README.md                 # This file
├── 01-access-control/        # Roles, privileges, audit
│   └── README.md
├── 02-hardening/             # Hardening checklist
│   └── README.md
└── notes/                    # Personal notes
```

---

> **Previous:** [04-Replikasi-HA](../04-Replikasi-HA/)  
> **Next:** [06-Performance-Tuning](../06-Performance-Tuning/) - Minggu 13-14