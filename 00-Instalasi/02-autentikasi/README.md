# Phase 1.3: Koneksi & Autentikasi MariaDB

> **Target:** Minggu 1 (Hari 6-7)

---

## 1. Arsitektur Autentikasi MariaDB

```
CLIENT                                    SERVER (mariadbd)
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  1. TCP/IP / Unix Socket                                         │
│     └─▶ Connection Handler (per thread / thread pool)           │
│                                                                 │
│  2. Authentication Plugin Negotiation                            │
│     Client: "Saya mau pakai plugin X"                           │
│     Server: "OK, gunakan plugin X" / "Plugin tidak tersedia"    │
│                                                                 │
│  3. Plugin Execution                                             │
│     ┌─────────────────────────────────────────────────────┐     │
│     │  mysql_native_password  │  caching_sha2_password   │     │
│     │  unix_socket            │  pam / ldap / kerberos   │     │
│     │  auth_gssapi            │  auth_ed25519            │     │
│     └─────────────────────────────────────────────────────┘     │
│                                                                 │
│  4. Account Validation (mysql.user table)                        │
│     - User + Host match                                          │
│     - Plugin match                                               │
│     - Password valid / expired                                   │
│     - Account locked?                                            │
│     - SSL required?                                              │
│                                                                 │
│  5. Privilege Assignment                                         │
│     - Global (mysql.user)                                        │
│     - Database (mysql.db)                                        │
│     - Table (mysql.tables_priv)                                  │
│     - Column (mysql.columns_priv)                                │
│     - Procedure (mysql.procs_priv)                               │
│     - Proxy (mysql.proxies_priv)                                 │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. User & Host: Konsep Fundamental

### 2.1 User = User + Host (Bukan cuma username!)

```sql
-- User 'app' dari localhost ≠ 'app' dari 10.0.0.5 ≠ 'app' dari %
CREATE USER 'app'@'localhost' IDENTIFIED BY 'pass1';
CREATE USER 'app'@'10.0.0.5' IDENTIFIED BY 'pass2';
CREATE USER 'app'@'%' IDENTIFIED BY 'pass3';        -- Wildcard: semua host
CREATE USER 'app'@'10.0.0.%' IDENTIFIED BY 'pass4'; -- Subnet wildcard
CREATE USER 'app'@'%.example.com' IDENTIFIED BY 'pass5'; -- Hostname wildcard
```

### 2.2 Prioritas Host Matching (Spesifik → Umum)

| Urutan | Host Pattern | Contoh |
|--------|--------------|--------|
| 1 | Exact IP | `192.168.1.100` |
| 2 | Exact hostname | `app-server.example.com` |
| 3 | IP dengan wildcard | `192.168.1.%` |
| 4 | Hostname dengan wildcard | `%.example.com` |
| 5 | Wildcard penuh | `%` |
| 6 | `localhost` (special, Unix socket) | `localhost` |

> **Best Practice:** Hindari `'user'@'%'`. Gunakan subnet `'user'@'10.0.%'` atau hostname pattern.

### 2.3 Lihat User yang Ada

```sql
-- Semua user
SELECT User, Host, plugin, authentication_string, 
       password_expired, account_locked
FROM mysql.user
ORDER BY User, Host;

-- Format rapi
SELECT User, Host, 
       plugin AS AuthPlugin,
       IF(password_expired='Y','EXPIRED','OK') AS PwdStatus,
       IF(account_locked='Y','LOCKED','OPEN') AS AcctStatus,
       max_connections, max_user_connections
FROM mysql.user;
```

---

## 3. Authentication Plugins

### 3.1 Built-in Plugins Comparison

| Plugin | Since | Mechanism | Use Case | Keamanan |
|--------|-------|-----------|----------|----------|
| **mysql_native_password** | 5.5+ | SHA1(password) double-hash | Legacy apps, compatibility | ⚠️ Weak (SHA1) |
| **caching_sha2_password** | 10.4+ (default 10.5+) | SHA-256 + caching | **Default modern**, secure | ✅ Strong |
| **unix_socket** | 10.4+ | SO_PEERCRED (UID/GID) | Local root, system users | ✅ Very strong (OS auth) |
| **pam** | 10.0+ | PAM stack (ldap, kerberos, etc) | Enterprise AD/LDAP | ✅ Centralized |
| **ldap** | 10.6+ | Direct LDAP bind | AD/OpenLDAP integration | ✅ Centralized |
| **auth_gssapi** | 10.4+ | Kerberos/GSSAPI | Kerberos environments | ✅ Enterprise |
| **auth_ed25519** | 10.6+ | Ed25519 public key | Passwordless, automation | ✅ Very strong |
| **mysql_clear_password** | 5.5+ | Plaintext (via TLS only!) | PAM/LDAP client side | ⚠️ Butuh TLS |

### 3.2 Default Plugin per Version

```sql
-- Cek default authentication plugin
SHOW VARIABLES LIKE 'default_authentication_plugin';
-- 10.4: mysql_native_password
-- 10.5+: caching_sha2_password
```

### 3.3 Create User dengan Plugin Tertentu

```sql
-- caching_sha2_password (recommended modern)
CREATE USER 'app'@'%' IDENTIFIED WITH caching_sha2_password BY 'StrongPass123!';

-- mysql_native_password (legacy app compatibility)
CREATE USER 'legacy_app'@'%' IDENTIFIED WITH mysql_native_password BY 'LegacyPass123!';

-- unix_socket (local OS user, no password)
CREATE USER 'backup'@'localhost' IDENTIFIED WITH unix_socket;
-- Login: sudo -u backup mariadb (no password needed)

-- PAM (butuh plugin load & pam config)
INSTALL SONAME 'auth_pam';
CREATE USER 'ldap_user'@'%' IDENTIFIED WITH pam AS 'mariadb';

-- LDAP Direct (MariaDB 10.6+)
INSTALL SONAME 'auth_ldap';
CREATE USER 'ldap_user2'@'%' IDENTIFIED WITH ldap AS 'cn=users,dc=example,dc=com';

-- Ed25519 Public Key (passwordless)
INSTALL SONAME 'auth_ed25519';
CREATE USER 'automation'@'%' IDENTIFIED WITH auth_ed25519 AS 'PUBLIC_KEY_HERE';
```

---

## 4. User Management: CRUD Operations

### 4.1 Create User

```sql
-- Basic
CREATE USER 'app'@'10.0.0.%' IDENTIFIED BY 'StrongP@ssw0rd!';

-- With resource limits
CREATE USER 'reporting'@'%' IDENTIFIED BY 'ReadOnly123!'
  WITH MAX_QUERIES_PER_HOUR 1000
       MAX_UPDATES_PER_HOUR 0
       MAX_CONNECTIONS_PER_HOUR 50
       MAX_USER_CONNECTIONS 5;

-- With password policy (validate_password)
CREATE USER 'strict'@'%' IDENTIFIED BY 'ComplexP@ssw0rd123!'
  PASSWORD EXPIRE INTERVAL 90 DAY
  FAILED_LOGIN_ATTEMPTS 3
  PASSWORD_LOCK_TIME 2;

-- With TLS requirement
CREATE USER 'secure_app'@'%' IDENTIFIED BY 'SecurePass123!'
  REQUIRE SSL;                          -- Butuh SSL
  -- REQUIRE X509;                      -- Butuh client cert
  -- REQUIRE ISSUER '/CN=CA' SUBJECT '/CN=client';
```

### 4.2 Alter User

```sql
-- Ganti password
ALTER USER 'app'@'10.0.0.%' IDENTIFIED BY 'NewStrongP@ssw0rd!';

-- Ganti plugin
ALTER USER 'app'@'10.0.0.%' IDENTIFIED WITH caching_sha2_password BY 'NewPass123!';

-- Expire password (force change next login)
ALTER USER 'app'@'10.0.0.%' PASSWORD EXPIRE;

-- Lock/Unlock account
ALTER USER 'app'@'10.0.0.%' ACCOUNT LOCK;
ALTER USER 'app'@'10.0.0.%' ACCOUNT UNLOCK;

-- Resource limits
ALTER USER 'reporting'@'%' WITH MAX_USER_CONNECTIONS 10;

-- TLS requirements
ALTER USER 'secure_app'@'%' REQUIRE SSL;
ALTER USER 'secure_app'@'%' REQUIRE NONE;  -- Hapus requirement
```

### 4.3 Rename & Drop User

```sql
-- Rename (MariaDB 10.0+)
RENAME USER 'old_name'@'%' TO 'new_name'@'%';

-- Drop (hapus user & privileges)
DROP USER 'app'@'10.0.0.%';
DROP USER IF EXISTS 'old_app'@'%';  -- Safe drop
```

### 4.4 Show Create User (Audit)

```sql
-- Lihat definisi user (termasuk password hash)
SHOW CREATE USER 'app'@'10.0.0.%';
/* Output:
CREATE USER 'app'@'10.0.0.%' IDENTIFIED WITH 'caching_sha2_password' 
AS '$A$005$...' 
REQUIRE NONE PASSWORD EXPIRE DEFAULT ACCOUNT UNLOCK 
PASSWORD HISTORY DEFAULT PASSWORD REUSE INTERVAL DEFAULT 
FAILED_LOGIN_ATTEMPTS 0 PASSWORD_LOCK_TIME 0
*/
```

---

## 5. Password Policies & Validation

### 5.1 validate_password Plugin (Built-in)

```sql
-- Install (usually pre-installed)
INSTALL PLUGIN validate_password SONAME 'validate_password';

-- Cek status
SHOW PLUGINS WHERE Name LIKE 'validate%';

-- Konfigurasi (dynamic, no restart)
SET GLOBAL validate_password_length = 12;           -- Min length (default 8)
SET GLOBAL validate_password_mixed_case_count = 1;  -- Min upper+lower
SET GLOBAL validate_password_number_count = 1;      -- Min digit
SET GLOBAL validate_password_special_char_count = 1;-- Min special char
SET GLOBAL validate_password_policy = 'MEDIUM';     -- LOW/MEDIUM/STRONG
SET GLOBAL validate_password_dictionary_file = '/usr/share/dict/words'; -- Custom dict
```

| Policy | Checks |
|--------|--------|
| **LOW** | Length only |
| **MEDIUM** | Length + uppercase + lowercase + number + special char |
| **STRONG** | MEDIUM + dictionary check (no common words) |

### 5.2 Password Expiration & History (MariaDB 10.4+)

```sql
-- Global defaults
SET GLOBAL default_password_lifetime = 90;        -- Expire after 90 days (0 = never)
SET GLOBAL password_history = 5;                  -- Remember last 5 passwords
SET GLOBAL password_reuse_interval = 365;         -- Can't reuse within 365 days

-- Per user (override global)
ALTER USER 'app'@'%' PASSWORD EXPIRE INTERVAL 60 DAY;
ALTER USER 'app'@'%' PASSWORD HISTORY 10;
ALTER USER 'app'@'%' PASSWORD REUSE INTERVAL 180 DAY;

-- Disable expiration for service accounts
ALTER USER 'backup'@'localhost' PASSWORD EXPIRE NEVER;
```

### 5.3 Failed Login Tracking (MariaDB 10.4+)

```sql
-- Global
SET GLOBAL max_failed_login_attempts = 3;
SET GLOBAL password_lock_time = 4;  -- Hours to lock (0 = forever until admin unlock)

-- Per user
ALTER USER 'app'@'%' FAILED_LOGIN_ATTEMPTS 5 PASSWORD_LOCK_TIME 2;

-- Unlock manually
ALTER USER 'app'@'%' ACCOUNT UNLOCK;

-- Check locked accounts
SELECT User, Host, account_locked 
FROM mysql.user 
WHERE account_locked = 'Y';
```

---

## 6. Privileges System (GRANT/REVOKE)

### 6.1 Privilege Hierarchy

```
GLOBAL (mysql.user)
  │
  ├── SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, ALTER, INDEX
  ├── CREATE USER, DROP USER, GRANT OPTION
  ├── SUPER, RELOAD, SHUTDOWN, PROCESS, FILE, REPLICATION *
  └── ALL PRIVILEGES

DATABASE (mysql.db)
  │
  ├── SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, ALTER, INDEX
  ├── CREATE TEMPORARY TABLES, LOCK TABLES, EXECUTE, CREATE VIEW
  └── ALL PRIVILEGES

TABLE (mysql.tables_priv)
  │
  ├── SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, ALTER, INDEX
  ├── TRIGGER, REFERENCES
  └── ALL PRIVILEGES

COLUMN (mysql.columns_priv)
  │
  ├── SELECT, INSERT, UPDATE, REFERENCES (per column)

PROCEDURE/FUNCTION (mysql.procs_priv)
  │
  ├── EXECUTE, ALTER ROUTINE, GRANT OPTION
```

### 6.2 Common Grant Patterns

```sql
-- Read-only user (reporting)
CREATE USER 'reporting'@'%' IDENTIFIED BY 'ReadOnly123!';
GRANT SELECT ON production_db.* TO 'reporting'@'%';
GRANT SELECT ON analytics_db.* TO 'reporting'@'%';
-- Atau: GRANT SELECT ON *.* TO 'reporting'@'%' WITH MAX_USER_CONNECTIONS 5;

-- Application user (read/write, no DDL)
CREATE USER 'app'@'10.0.%' IDENTIFIED BY 'AppPass123!';
GRANT SELECT, INSERT, UPDATE, DELETE, EXECUTE ON app_db.* TO 'app'@'10.0.%';

-- Developer (DDL + DML, no admin)
CREATE USER 'dev'@'%' IDENTIFIED BY 'DevPass123!';
GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, ALTER, INDEX, 
      CREATE VIEW, CREATE ROUTINE, ALTER ROUTINE, EXECUTE, TRIGGER
   ON dev_db.* TO 'dev'@'%';

-- Admin (full access, but not SUPER)
CREATE USER 'dba'@'localhost' IDENTIFIED BY 'DbaPass123!';
GRANT ALL PRIVILEGES ON *.* TO 'dba'@'localhost' WITH GRANT OPTION;

-- Replication user
CREATE USER 'repl'@'%' IDENTIFIED BY 'ReplPass123!';
GRANT REPLICATION REPLICA ON *.* TO 'repl'@'%';  -- MariaDB 10.5.2+ (REPLICA)
-- GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%';  -- Legacy alias

-- Backup user (mariabackup needs)
CREATE USER 'backup'@'localhost' IDENTIFIED BY 'BackupPass123!';
GRANT RELOAD, PROCESS, LOCK TABLES, REPLICATION REPLICA ON *.* TO 'backup'@'localhost';
GRANT SELECT ON *.* TO 'backup'@'localhost';
```

### 6.3 Revoke & Show Grants

```sql
-- Revoke specific
REVOKE INSERT, UPDATE ON app_db.* FROM 'app'@'10.0.%';

-- Revoke all
REVOKE ALL PRIVILEGES ON *.* FROM 'old_dev'@'%';

-- Show grants
SHOW GRANTS FOR 'app'@'10.0.%';
SHOW GRANTS FOR CURRENT_USER;  -- Current session
SHOW GRANTS;                    -- Same as above
```

---

## 7. Roles (MariaDB 10.0+) - RBAC

### 7.1 Create & Assign Roles

```sql
-- 1. Create roles
CREATE ROLE 'read_only', 'read_write', 'db_admin', 'replication';

-- 2. Grant privileges to roles
GRANT SELECT ON *.* TO 'read_only';
GRANT SELECT, INSERT, UPDATE, DELETE ON app_db.* TO 'read_write';
GRANT ALL PRIVILEGES ON *.* TO 'db_admin' WITH GRANT OPTION;
GRANT REPLICATION REPLICA ON *.* TO 'replication';

-- 3. Grant roles to users
CREATE USER 'analyst'@'%' IDENTIFIED BY 'Analyst123!';
CREATE USER 'app_user'@'%' IDENTIFIED BY 'AppUser123!';
CREATE USER 'dba_john'@'localhost' IDENTIFIED BY 'DbaJohn123!';

GRANT 'read_only' TO 'analyst'@'%';
GRANT 'read_write' TO 'app_user'@'%';
GRANT 'db_admin', 'replication' TO 'dba_john'@'localhost';

-- 4. Set default role (auto-activate on login)
SET DEFAULT ROLE 'read_only' TO 'analyst'@'%';
SET DEFAULT ROLE 'read_write' TO 'app_user'@'%';
SET DEFAULT ROLE ALL TO 'dba_john'@'localhost';

-- 5. User activates role in session
SET ROLE 'read_write';              -- Activate specific
SET ROLE ALL;                       -- Activate all granted
SET ROLE NONE;                      -- Deactivate all
SELECT CURRENT_ROLE();              -- Check active role
```

### 7.2 Role Hierarchy (Nested Roles)

```sql
-- Role bisa grant ke role lain
CREATE ROLE 'junior_dba', 'senior_dba';
GRANT 'read_only' TO 'junior_dba';
GRANT 'read_write', 'junior_dba' TO 'senior_dba';  -- Inherits junior_dba
GRANT 'senior_dba' TO 'dba_john'@'localhost';
```

### 7.3 View Role Info

```sql
-- List roles
SELECT * FROM mysql.roles_mapping;

-- Show role grants
SHOW GRANTS FOR 'read_write' USING 'read_write';

-- User's effective privileges (including roles)
SHOW GRANTS FOR 'dba_john'@'localhost' USING 'db_admin', 'replication';
```

---

## 8. SSL/TLS Connections

### 8.1 Generate Certificates (OpenSSL)

```bash
# Directory
mkdir -p /etc/mysql/ssl
cd /etc/mysql/ssl

# 1. CA Private Key
openssl genrsa -out ca-key.pem 4096

# 2. CA Certificate
openssl req -new -x509 -nodes -days 3650 \
  -key ca-key.pem -out ca.pem \
  -subj "/CN=MariaDB CA"

# 3. Server Private Key & CSR
openssl req -newkey rsa:4096 -nodes -days 3650 \
  -keyout server-key.pem -out server-req.pem \
  -subj "/CN=db-server.example.com"

# 4. Server Certificate (signed by CA)
openssl x509 -req -in server-req.pem -days 3650 \
  -CA ca.pem -CAkey ca-key.pem -CAcreateserial \
  -out server-cert.pem

# 5. Client Private Key & CSR
openssl req -newkey rsa:4096 -nodes -days 3650 \
  -keyout client-key.pem -out client-req.pem \
  -subj "/CN=app-client"

# 6. Client Certificate (signed by CA)
openssl x509 -req -in client-req.pem -days 3650 \
  -CA ca.pem -CAkey ca-key.pem -CAcreateserial \
  -out client-cert.pem

# 7. Verify
openssl verify -CAfile ca.pem server-cert.pem client-cert.pem

# 8. Permissions
chown mysql:mysql *.pem
chmod 640 *.pem
chmod 600 *-key.pem
```

### 8.2 Server Config (my.cnf)

```ini
[mysqld]
# SSL Files
ssl_ca = /etc/mysql/ssl/ca.pem
ssl_cert = /etc/mysql/ssl/server-cert.pem
ssl_key = /etc/mysql/ssl/server-key.pem

# Cipher suite (modern)
ssl_cipher = TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256:TLS_AES_128_GCM_SHA256
tls_version = TLSv1.2,TLSv1.3

# Force SSL for specific users (or global)
# require_secure_transport = ON  -- Global force SSL
```

### 8.3 Client Config

```ini
# /etc/mysql/mariadb.conf.d/99-ssl-client.cnf
[client]
ssl_ca = /etc/mysql/ssl/ca.pem
ssl_cert = /etc/mysql/ssl/client-cert.pem
ssl_key = /etc/mysql/ssl/client-key.pem
# ssl_verify_server_cert = ON  -- Verify server cert (default ON in 10.5+)
```

### 8.4 Require SSL per User

```sql
-- Require SSL (encrypted connection)
CREATE USER 'secure_app'@'%' IDENTIFIED BY 'Pass123!' REQUIRE SSL;

-- Require Client Certificate (mutual TLS)
CREATE USER 'mutual_app'@'%' IDENTIFIED BY 'Pass123!' REQUIRE X509;

-- Require Specific Issuer/Subject
CREATE USER 'strict_app'@'%' IDENTIFIED BY 'Pass123!' 
  REQUIRE ISSUER '/CN=MariaDB CA' SUBJECT '/CN=app-client';

-- Check SSL status
SHOW VARIABLES LIKE '%ssl%';
STATUS;  -- Cek "SSL: Cipher in use is TLS_AES_256_GCM_SHA384"
```

### 8.5 Verify SSL Connection

```bash
# Test connection with SSL
mariadb -h db-server.example.com -u secure_app -p --ssl-ca=/etc/mysql/ssl/ca.pem

# Check cipher
mariadb -u secure_app -p -e "STATUS; SHOW VARIABLES LIKE 'ssl_cipher';"

# Force SSL (fail if not available)
mariadb -h db-server -u secure_app -p --ssl-mode=REQUIRED
```

| ssl-mode | Behavior |
|----------|----------|
| `DISABLED` | No SSL |
| `PREFERRED` | Try SSL, fallback plain (default) |
| `REQUIRED` | Fail if no SSL |
| `VERIFY_CA` | REQUIRED + verify CA |
| `VERIFY_IDENTITY` | VERIFY_CA + verify hostname |

---

## 9. Connection Methods & Performance

### 9.1 Connection Types

| Method | Protocol | Use Case | Performance |
|--------|----------|----------|-------------|
| **Unix Socket** | Local (file) | Localhost, root, backup | **Fastest** (no TCP overhead) |
| **TCP/IP (localhost)** | 127.0.0.1:3306 | Apps on same host | Good |
| **TCP/IP (remote)** | IP:3306 | Remote apps | Network latency |
| **Named Pipe** | Windows only | Windows local | Similar to socket |

### 9.2 Connection Pooling (Application Side)

```python
# Python (mysql-connector-python / pymysql) - Pool
import mysql.connector.pooling

pool = mysql.connector.pooling.MySQLConnectionPool(
    pool_name="app_pool",
    pool_size=20,                    # Max connections
    pool_reset_session=True,         # Reset session vars
    host="db-server",
    user="app",
    password="AppPass123!",
    database="app_db",
    autocommit=True,
    charset="utf8mb4"
)

# Get connection
conn = pool.get_connection()
cursor = conn.cursor()
cursor.execute("SELECT 1")
conn.close()  # Return to pool
```

```java
// Java (HikariCP) - Recommended
HikariConfig config = new HikariConfig();
config.setJdbcUrl("jdbc:mariadb://db-server:3306/app_db");
config.setUsername("app");
config.setPassword("AppPass123!");
config.setMaximumPoolSize(20);
config.setMinimumIdle(5);
config.setConnectionTimeout(5000);
config.setIdleTimeout(300000);
config.setMaxLifetime(1800000);
config.addDataSourceProperty("cachePrepStmts", "true");
config.addDataSourceProperty("prepStmtCacheSize", "250");
config.addDataSourceProperty("prepStmtCacheSqlLimit", "2048");
HikariDataSource ds = new HikariDataSource(config);
```

### 9.3 Server-Side Thread Pool (High Concurrency)

```ini
# my.cnf - Enable thread pool (MariaDB 10.1+)
[mysqld]
thread_handling = pool-of-threads
thread_pool_size = 16              # Number of thread groups (CPU cores)
thread_pool_max_threads = 500      # Max threads total
thread_pool_idle_timeout = 60      # Seconds before killing idle
thread_pool_stall_limit = 500      # ms before considering stalled
```

> **When to use:** > 500 concurrent connections, high contention. Not needed for typical web apps with connection pooling.

---

## 10. Monitoring Connections & Users

### 10.1 Current Connections

```sql
-- All connections
SHOW PROCESSLIST;
SHOW FULL PROCESSLIST;  -- Full query text

-- Performance Schema (detailed)
SELECT * FROM performance_schema.threads 
WHERE TYPE = 'FOREGROUND' 
ORDER BY PROCESSLIST_TIME DESC;

-- Connection count per user
SELECT USER, HOST, COUNT(*) as connections
FROM performance_schema.threads
WHERE TYPE = 'FOREGROUND'
GROUP BY USER, HOST;

-- Connection count per host
SELECT SUBSTRING_INDEX(HOST, ':', 1) as client_ip, COUNT(*) as connections
FROM performance_schema.threads
WHERE TYPE = 'FOREGROUND'
GROUP BY client_ip
ORDER BY connections DESC;
```

### 10.2 Connection Limits & Errors

```sql
-- Global max connections
SHOW VARIABLES LIKE 'max_connections';
SHOW VARIABLES LIKE 'max_connect_errors';

-- Current usage
SHOW STATUS LIKE 'Threads_connected';
SHOW STATUS LIKE 'Max_used_connections';
SHOW STATUS LIKE 'Aborted_connects';
SHOW STATUS LIKE 'Connection_errors%';

-- Per user limit
SELECT User, Host, max_user_connections 
FROM mysql.user 
WHERE max_user_connections > 0;
```

### 10.3 Kill Connections

```sql
-- Kill query only (keeps connection)
KILL QUERY 12345;

-- Kill connection (full disconnect)
KILL CONNECTION 12345;

-- Kill all connections for user (MariaDB 10.6+)
KILL USER 'app'@'%';

-- Script: kill long-running queries (> 60s)
SELECT CONCAT('KILL ', ID, ';') 
FROM INFORMATION_SCHEMA.PROCESSLIST 
WHERE TIME > 60 AND USER != 'root' AND COMMAND = 'Query';
```

---

## 11. Praktikum: User & Auth Management

### Lab 1: Create Users dengan Berbagai Plugin
```sql
-- 1. Modern app user (caching_sha2_password)
CREATE USER 'webapp'@'10.0.%' IDENTIFIED BY 'WebAppPass123!';
GRANT SELECT, INSERT, UPDATE, DELETE ON app_db.* TO 'webapp'@'10.0.%';

-- 2. Legacy app (mysql_native_password)
CREATE USER 'legacy'@'10.0.%' IDENTIFIED WITH mysql_native_password BY 'LegacyPass123!';
GRANT SELECT, INSERT, UPDATE, DELETE ON legacy_db.* TO 'legacy'@'10.0.%';

-- 3. Backup user (unix_socket)
CREATE USER 'backup'@'localhost' IDENTIFIED WITH unix_socket;
GRANT RELOAD, PROCESS, LOCK TABLES, REPLICATION REPLICA, SELECT ON *.* TO 'backup'@'localhost';

-- 4. Read-only analyst (role-based)
CREATE ROLE 'read_only_analyst';
GRANT SELECT ON *.* TO 'read_only_analyst';
CREATE USER 'analyst'@'%' IDENTIFIED BY 'Analyst123!';
GRANT 'read_only_analyst' TO 'analyst'@'%';
SET DEFAULT ROLE 'read_only_analyst' TO 'analyst'@'%';

-- 5. Verify
SELECT User, Host, plugin, authentication_string FROM mysql.user WHERE User IN ('webapp','legacy','backup','analyst');
SHOW GRANTS FOR 'webapp'@'10.0.%';
SHOW GRANTS FOR 'analyst'@'%' USING 'read_only_analyst';
```

### Lab 2: Password Policy & Expiration
```sql
-- Enable validate_password
INSTALL PLUGIN validate_password SONAME 'validate_password';
SET GLOBAL validate_password_length = 12;
SET GLOBAL validate_password_policy = 'MEDIUM';

-- Test weak password (should fail)
CREATE USER 'weak'@'%' IDENTIFIED BY '123';  -- ERROR 1819 (HY000)

-- Test strong password
CREATE USER 'strong'@'%' IDENTIFIED BY 'StrongP@ssw0rd123!';  -- OK

-- Set expiration
ALTER USER 'strong'@'%' PASSWORD EXPIRE INTERVAL 90 DAY;
ALTER USER 'strong'@'%' FAILED_LOGIN_ATTEMPTS 3 PASSWORD_LOCK_TIME 2;

-- Simulate failed login (3x wrong password) → account locked
-- Then unlock
ALTER USER 'strong'@'%' ACCOUNT UNLOCK;
```

### Lab 3: SSL Setup & Test
```bash
# 1. Generate certs (ikuti section 8.1)
# 2. Configure server (section 8.2)
# 3. Restart
sudo systemctl restart mariadb

# 4. Create SSL-required user
CREATE USER 'ssl_user'@'%' IDENTIFIED BY 'SslPass123!' REQUIRE SSL;

# 5. Test connection
mariadb -h localhost -u ssl_user -p --ssl-mode=REQUIRED --ssl-ca=/etc/mysql/ssl/ca.pem
# Should work

# 6. Test without SSL (should fail)
mariadb -h localhost -u ssl_user -p --ssl-mode=DISABLED
# ERROR 1045 (28000): Access denied (SSL required)
```

### Lab 4: Connection Monitoring
```sql
-- Simulate connections
-- Terminal 1: mariadb -u webapp -p -e "DO SLEEP(100);"
-- Terminal 2: mariadb -u analyst -p -e "SELECT SLEEP(50);"

-- Monitor
SHOW PROCESSLIST;
SELECT * FROM performance_schema.threads WHERE TYPE='FOREGROUND'\G

-- Kill long query
KILL QUERY <thread_id>;
```

---

## 13. Troubleshooting Autentikasi

| Error | Penyebab | Solusi |
|-------|----------|--------|
| `Access denied for user 'x'@'localhost'` | unix_socket plugin, password salah | `sudo mariadb -u root` atau reset password |
| `Authentication plugin 'caching_sha2_password' not loaded` | Client library lama (Connector/J < 8.0.13, PHP mysqlnd lama) | Upgrade client / use `mysql_native_password` |
| `SSL connection error: certificate verify failed` | CA mismatch / self-signed not trusted | Verify `ssl_ca` path, use `--ssl-mode=VERIFY_CA` |
| `Host 'x.x.x.x' is blocked because of many connection errors` | `max_connect_errors` tercapai | `FLUSH HOSTS;` atau increase `max_connect_errors` |
| `Too many connections` | `max_connections` tercapai | Increase `max_connections`, check connection pool leak |
| `User 'x' has exceeded 'max_user_connections'` | Per-user limit | `ALTER USER 'x'@'%' WITH MAX_USER_CONNECTIONS 50;` |
| `Password expired` | `password_expired = 'Y'` | Login & set new password, atau `ALTER USER ... PASSWORD EXPIRE NEVER` |

---

## 14. Referensi Lanjutan

| Topik | Resource |
|-------|----------|
| Authentication Plugins | https://mariadb.com/kb/en/authentication-plugins/ |
| caching_sha2_password | https://mariadb.com/kb/en/caching_sha2_password-authentication-plugin/ |
| unix_socket Plugin | https://mariadb.com/kb/en/unix_socket-authentication-plugin/ |
| PAM Authentication | https://mariadb.com/kb/en/pam-authentication-plugin/ |
| LDAP Authentication | https://mariadb.com/kb/en/ldap-authentication-plugin/ |
| Roles (RBAC) | https://mariadb.com/kb/en/roles/ |
| Password Validation | https://mariadb.com/kb/en/password-validation-plugin/ |
| SSL/TLS Configuration | https://mariadb.com/kb/en/securing-connections-for-client-and-server/ |
| Connection Handling | https://mariadb.com/kb/en/connection-handling/ |
| Thread Pool | https://mariadb.com/kb/en/thread-pool/ |

---

> **Phase 1 Complete!** ✅
> 
> **Next:** [Phase 2: Konfigurasi & Tuning Dasar](../phase2/README.md) - Minggu 3-4
> 
> **Progress Update:**
> ```
> Phase 1: ██████████ 100%  (Fondasi & Instalasi) ✓
> Phase 2: ░░░░░░░░░░   0%  (Konfigurasi & Tuning)
> Phase 3: ░░░░░░░░░░   0%  (Manajemen Database)
> ...
> ```