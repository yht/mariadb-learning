# Phase 1.2: Instalasi & Setup Awal MariaDB

> **Target:** Minggu 1 (Hari 4-5)

---

## 1. Metode Instalasi: Pilih Satu

| Metode | Cocok Untuk | Kelebihan | Kekurangan |
|--------|-------------|-----------|------------|
| **Repository (apt/yum/dnf)** | Production Linux | Auto-update security, dependency management, standard path | Versi mungkin tidak latest |
| **Binary Tarball** | Custom path, non-root, testing | Versi tepat, portable, multiple instances | Manual update, manual systemd |
| **Docker/Podman** | Dev, CI/CD, microservices | Isolated, reproducible, easy version switch | Persistence setup, performance overhead |
| **Source Compile** | Development, custom build | Full control, debug symbols | Slow, complex, maintenance burden |

> **Rekomendasi Production:** Repository OS (Ubuntu/Debian: `mariadb-server`, RHEL/Rocky/Alma: `mariadb-server`) atau MariaDB Official Repository untuk versi terbaru.

---

## 2. Instalasi via Repository (Recommended)

### 2.1 Ubuntu / Debian (MariaDB Official Repo)

```bash
# 1. Install prerequisites
sudo apt update
sudo apt install -y curl gnupg2 software-properties-common

# 2. Add MariaDB Official Repository (pilih versi)
# LTS: 11.4 (supported until 2028), 10.11 (until 2028), 10.6 (until 2026)
# https://mariadb.org/download/?t=repo

# Ubuntu 24.04 (Noble) - MariaDB 11.4 LTS
curl -fsSL https://mariadb.org/mariadb_release_signing_key.asc | sudo gpg --dearmor -o /usr/share/keyrings/mariadb-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/mariadb-archive-keyring.gpg] https://mariadb.mirror.liquidtelecom.com/repo/11.4/ubuntu noble main" | sudo tee /etc/apt/sources.list.d/mariadb.list

# Ubuntu 22.04 (Jammy) - MariaDB 11.4 LTS
echo "deb [signed-by=/usr/share/keyrings/mariadb-archive-keyring.gpg] https://mariadb.mirror.liquidtelecom.com/repo/11.4/ubuntu jammy main" | sudo tee /etc/apt/sources.list.d/mariadb.list

# 3. Update & Install
sudo apt update
sudo apt install -y mariadb-server mariadb-client mariadb-backup

# 4. Verify
mariadb --version
systemctl status mariadb
```

### 2.2 RHEL / Rocky / AlmaLinux / Fedora

```bash
# 1. Add MariaDB Official Repo (via dnf config-manager)
# https://mariadb.org/download/?t=repo

# RHEL 9 / Rocky 9 / AlmaLinux 9 - MariaDB 11.4 LTS
sudo dnf install -y https://rpms.mariadb.com/mariadb/11.4/rhel9/x86_64/mariadb-release-11.4-1.el9.noarch.rpm

# RHEL 8 / Rocky 8 / AlmaLinux 8
sudo dnf install -y https://rpms.mariadb.com/mariadb/11.4/rhel8/x86_64/mariadb-release-11.4-1.el8.noarch.rpm

# Fedora
sudo dnf install -y https://rpms.mariadb.com/mariadb/11.4/fedora/$(rpm -E %fedora)/x86_64/mariadb-release-11.4-1.fc$(rpm -E %fedora).noarch.rpm

# 2. Install
sudo dnf install -y MariaDB-server MariaDB-client MariaDB-backup

# 3. Enable & Start
sudo systemctl enable --now mariadb
sudo systemctl status mariadb
```

### 2.3 openSUSE

```bash
# Via official repo
sudo zypper addrepo --refresh https://mariadb.mirror.liquidtelecom.com/repo/11.4/opensuse/15/x86_64/mariadb.repo
sudo zypper install MariaDB-server MariaDB-client MariaDB-backup
sudo systemctl enable --now mariadb
```

---

## 3. Instalasi via Binary Tarball (Multiple Instances / Non-root)

```bash
# 1. Download
cd /opt
VERSION=11.4.4
wget https://mariadb.mirror.liquidtelecom.com/mariadb-11.4.4/bintar-linux-x86_64/mariadb-11.4.4-linux-x86_64.tar.gz
tar xzf mariadb-11.4.4-linux-x86_64.tar.gz
ln -s mariadb-11.4.4-linux-x86_64 mariadb

# 2. Create mysql user (non-root)
sudo useradd -r -s /bin/false -d /opt/mariadb mysql
sudo chown -R mysql:mysql /opt/mariadb-11.4.4-linux-x86_64

# 3. Initialize data directory
sudo -u mysql /opt/mariadb/scripts/mariadb-install-db \
  --datadir=/var/lib/mysql3306 \
  --user=mysql \
  --basedir=/opt/mariadb

# 4. Create config file (/etc/my3306.cnf)
cat > /etc/my3306.cnf <<'EOF'
[mysqld]
basedir=/opt/mariadb
datadir=/var/lib/mysql3306
socket=/run/mysqld/mysqld3306.sock
port=3306
pid-file=/run/mysqld/mysqld3306.pid
log-error=/var/log/mysql/mariadb3306.err
user=mysql

# Basic tuning
innodb_buffer_pool_size=1G
innodb_log_file_size=256M
max_connections=200
EOF

# 5. Create systemd service (optional)
cat > /etc/systemd/system/mariadb3306.service <<'EOF'
[Unit]
Description=MariaDB 11.4 (Port 3306)
After=network.target

[Service]
Type=notify
User=mysql
Group=mysql
ExecStart=/opt/mariadb/bin/mariadbd --defaults-file=/etc/my3306.cnf
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
Restart=on-failure
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF

# 6. Start
sudo systemctl daemon-reload
sudo systemctl enable --now mariadb3306
sudo systemctl status mariadb3306

# 7. Connect
/opt/mariadb/bin/mariadb -S /run/mysqld/mysqld3306.sock -u root
```

---

## 4. Instalasi via Docker (Development / Testing)

```bash
# Single instance
docker run -d \
  --name mariadb-11.4 \
  -e MARIADB_ROOT_PASSWORD=secret123 \
  -e MARIADB_DATABASE=myapp \
  -e MARIADB_USER=appuser \
  -e MARIADB_PASSWORD=apppass \
  -p 3306:3306 \
  -v mariadb-data:/var/lib/mysql \
  -v ./my.cnf:/etc/mysql/my.cnf:ro \
  mariadb:11.4

# With custom config
cat > my.cnf <<'EOF'
[mysqld]
innodb_buffer_pool_size=512M
max_connections=100
slow_query_log=1
long_query_time=2
EOF

# Connect
docker exec -it mariadb-11.4 mariadb -u root -p

# Multiple instances (different ports)
docker run -d --name mariadb-3307 -p 3307:3306 -e MARIADB_ROOT_PASSWORD=secret mariadb:11.4
docker run -d --name mariadb-3308 -p 3308:3306 -e MARIADB_ROOT_PASSWORD=secret mariadb:11.4
```

> **Catatan Docker:** Data persist di volume `mariadb-data`. Untuk production, gunakan `--tmpfs /run/mysqld --tmpfs /var/lib/mysql/tmp` untuk performa.

---

## 5. Post-Installation: Verifikasi & Secure

### 5.1 Verifikasi Instalasi

```bash
# Cek service
systemctl status mariadb
systemctl is-enabled mariadb

# Cek version & build
mariadb --version
# Contoh output: mariadb  Ver 15.1 Distrib 11.4.4-MariaDB, for debian-linux-gnu (x86_64) using readline 8.2

# Cek proses
ps aux | grep mariadb
# Harus ada: mariadbd (main), background threads

# Cek port & socket
ss -tlnp | grep 3306
ls -la /run/mysqld/

# Test koneksi
mariadb -u root -e "SELECT VERSION(); SHOW VARIABLES LIKE 'datadir';"
```

### 5.2 `mariadb-secure-installation` (WAJIB Production)

```bash
sudo mariadb-secure-installation
```

**Interaktif - jawaban recommended:**

```
Enter current password for root (enter for none): [ENTER - default kosong]
Switch to unix_socket authentication? [Y/n] Y
Change the root password? [Y/n] Y
New password: [MASUKKAN PASSWORD KUAT]
Re-enter new password: [ULANGI]
Remove anonymous users? [Y/n] Y
Disallow root login remotely? [Y/n] Y
Remove test database and access to it? [Y/n] Y
Reload privilege tables now? [Y/n] Y
```

> **Catatan:** Di MariaDB 10.4+, `unix_socket` plugin default untuk root@localhost. Password root tidak dipakai untuk login lokal.

### 5.3 Manual Secure (Alternative / Automation)

```sql
-- Login sebagai root (unix_socket)
sudo mariadb -u root

-- Hapus anonymous user
DELETE FROM mysql.user WHERE User='';

-- Hapus test database
DROP DATABASE IF EXISTS test;
DELETE FROM mysql.db WHERE Db='test' OR Db='test\_%';

-- Set root password (jika tidak pakai unix_socket)
ALTER USER 'root'@'localhost' IDENTIFIED BY 'StrongP@ssw0rd!';

-- Batasi root hanya localhost
UPDATE mysql.user SET Host='localhost' WHERE User='root' AND Host='%';

-- Flush privileges
FLUSH PRIVILEGES;
```

---

## 6. Konfigurasi my.cnf: Lokasi & Prioritas

### 6.1 Urutan Baca Config (Prioritas Tinggi → Rendah)

```bash
# 1. Command line arguments (highest)
mariadbd --innodb_buffer_pool_size=2G

# 2. /etc/my.cnf (global)
# 3. /etc/mysql/my.cnf (Debian/Ubuntu)
# 4. /etc/mysql/mariadb.conf.d/*.cnf (drop-in, recommended)
# 5. ~/.my.cnf (user-specific)
# 6. DATADIR/.my.cnf (instance-specific, lowest)
```

### 6.2 Struktur Directory Config Modern (Debian/Ubuntu)

```
/etc/mysql/
├── my.cnf                    # Symlink ke mariadb.conf.d/50-mysql-clients.cnf
├── mariadb.conf.d/
│   ├── 50-client.cnf         # [client] section
│   ├── 50-mysql-clients.cnf  # [mysql] section
│   ├── 50-mysqld_safe.cnf    # [mysqld_safe]
│   ├── 50-server.cnf         # [mysqld] - MAIN CONFIG
│   ├── 60-galera.cnf         # Galera (jika install)
│   └── 99-custom.cnf         # BUAT FILE INI UNTUK OVERRIDE
└── conf.d/                   # Legacy (MySQL)
```

> **Best Practice:** Jangan edit `50-server.cnf` langsung. Buat `99-custom.cnf` di `mariadb.conf.d/`.

### 6.3 Template `99-custom.cnf` (Starter Production)

```ini
# /etc/mysql/mariadb.conf.d/99-custom.cnf
# ============================================================
# BASIC SETTINGS
# ============================================================
[mysqld]
# Identity
server_id = 1                          # UNIQUE per server (replication)
# Port & Socket
port = 3306
socket = /run/mysqld/mysqld.sock
# User & Directory
user = mysql
datadir = /var/lib/mysql
tmpdir = /tmp
# Language & Charset
character_set_server = utf8mb4
collation_server = utf8mb4_unicode_ci
# Timezone
default_time_zone = '+07:00'           # WIB / Asia/Jakarta

# ============================================================
# CONNECTIONS
# ============================================================
max_connections = 200                  # Sesuaikan RAM & app
thread_cache_size = 50                 # Cache thread untuk reconnect cepat
max_connect_errors = 100000            # Hindari block host
connect_timeout = 10
wait_timeout = 600                     # 10 menit idle
interactive_timeout = 600
# Thread pool (optional, untuk high concurrency)
# thread_handling = pool-of-threads
# thread_pool_size = 16

# ============================================================
# INNODB - MEMORY (PALING KRITIS)
# ============================================================
# 70-80% RAM untuk dedicated DB server
innodb_buffer_pool_size = 4G           # SET SESUAI RAM (contoh 4GB dari 8GB)
innodb_buffer_pool_instances = 4       # 1 per 1-2GB buffer pool
innodb_buffer_pool_chunk_size = 256M   # Default 128M, naikkan untuk pool besar

# Redo Log - critical untuk durability & performance
innodb_log_file_size = 512M            # Total 2 file = 1GB (10.6+ auto resize)
innodb_log_files_in_group = 2
innodb_log_buffer_size = 64M           # Batch commit

# Flush behavior - DURABILITY vs PERFORMANCE
innodb_flush_log_at_trx_commit = 1     # 1 = full ACID, 2 = faster (risk 1s loss)
innodb_flush_method = O_DIRECT         # Bypass OS cache (direct I/O)
innodb_flush_neighbors = 0             # 0 untuk SSD/NVMe, 1 untuk HDD

# File per table (default ON di 10.6+)
innodb_file_per_table = ON
innodb_file_per_table = 1

# Undo Log - separate tablespace untuk truncate
innodb_undo_tablespaces = 2            # Minimal 2 untuk online truncate
innodb_max_undo_log_size = 1G          # Auto truncate saat melebihi
innodb_undo_log_truncate = ON

# ============================================================
# INNODB - CONCURRENCY & PERFORMANCE
# ============================================================
innodb_read_io_threads = 8             # Default 4, naikkan untuk NVMe
innodb_write_io_threads = 8
innodb_purge_threads = 4               # Default 4
innodb_page_cleaners = 4               # Default 4
innodb_adaptive_hash_index = ON        # Auto enable untuk workload read-heavy
innodb_adaptive_hash_index_parts = 8   # Partition AHI (10.6+)

# Locking
innodb_lock_wait_timeout = 50          # Deteksi deadlock lebih cepat
innodb_deadlock_detect = ON

# ============================================================
# LOGGING & MONITORING
# ============================================================
# Error log
log_error = /var/log/mysql/mariadb.err
log_warnings = 2                       # Warning tambahan (aborted connects, dll)

# Slow Query Log (WAJIB)
slow_query_log = 1
slow_query_log_file = /var/log/mysql/slow.log
long_query_time = 2                    # Log query > 2 detik
log_queries_not_using_indexes = ON     # Log full table scan
log_slow_admin_statements = ON         # Log ALTER, ANALYZE, dll
log_slow_replica_statements = ON       # Log slow replication SQL

# General Log (debug only, OFF production)
# general_log = 0
# general_log_file = /var/log/mysql/general.log

# ============================================================
# BINARY LOG (Replication & PITR)
# ============================================================
log_bin = mariadb-bin
binlog_format = ROW                    # ROW = safest untuk replication
binlog_expire_logs_seconds = 604800    # 7 hari (10.6+), ganti expire_logs_days
max_binlog_size = 1G                   # Rotate size
binlog_checksum = CRC32                # Integrity check
sync_binlog = 1                        # 1 = safest, 0 = faster (OS cache)
# GTID
gtid_domain_id = 1                     # Unique per replication domain
gtid_strict_mode = ON                  # Strict GTID consistency

# ============================================================
# TEMPORARY TABLES & MEMORY
# ============================================================
tmp_table_size = 64M                   # Max in-memory tmp table
max_heap_table_size = 64M              # Max MEMORY table size
# Jika tmp table > ini → convert ke disk (Aria/MyISAM)

# Internal temp tables (InnoDB since 10.2)
innodb_temp_data_file_path = ibtmp1:12M:autoextend:max:5G

# ============================================================
# SECURITY
# ============================================================
local_infile = OFF                     # Disable LOAD DATA LOCAL
skip_show_database                     # Hide databases without privilege
# skip_name_resolve = 1                # Disable DNS lookup (faster connect)
# bind_address = 127.0.0.1             # Hanya localhost (ubah untuk remote)
require_secure_transport = ON          # Force SSL (setup cert dulu)

# ============================================================
# PERFORMANCE SCHEMA (Monitoring)
# ============================================================
performance_schema = ON
performance_schema_instrument = '%=ON'  # Enable all instruments

# ============================================================
# CLIENT DEFAULTS
# ============================================================
[client]
default-character-set = utf8mb4
socket = /run/mysqld/mysqld.sock

[mysql]
default-character-set = utf8mb4
no-auto-rehash                         # Faster startup (disable tab completion)

[mysqldump]
max_allowed_packet = 256M
default-character-set = utf8mb4
```

### 6.4 Validasi Config Sebelum Restart

```bash
# Test config syntax
mariadbd --defaults-file=/etc/mysql/mariadb.conf.d/99-custom.cnf --validate-config
# Atau
mariadb --help --defaults-file=/etc/mysql/mariadb.conf.d/99-custom.cnf | head -20

# Cek nilai efektif
mariadb -u root -e "SHOW VARIABLES LIKE 'innodb_buffer_pool_size';"
mariadb -u root -e "SHOW VARIABLES LIKE 'innodb_flush_log_at_trx_commit';"
```

---

## 7. Systemd Management

### 7.1 Command Dasar

```bash
# Start / Stop / Restart
sudo systemctl start mariadb
sudo systemctl stop mariadb
sudo systemctl restart mariadb
sudo systemctl reload mariadb          # Reload config (tidak semua variable)

# Enable / Disable auto-start
sudo systemctl enable mariadb
sudo systemctl disable mariadb

# Status
systemctl status mariadb
systemctl is-active mariadb
systemctl is-enabled mariadb

# Logs
journalctl -u mariadb -f               # Follow log
journalctl -u mariadb --since "1 hour ago"
journalctl -u mariadb -p err           # Error only
```

### 7.2 Override Systemd (Custom Limits)

```bash
# Edit override
sudo systemctl edit mariadb

# Masukkan:
[Service]
LimitNOFILE=65535
LimitNPROC=65535
LimitMEMLOCK=infinity
# Nice=-5                              # Priority lebih tinggi (optional)
# CPUSchedulingPolicy=rr               # Real-time (optional, butuh CAP_SYS_NICE)
```

```bash
# Reload & restart
sudo systemctl daemon-reload
sudo systemctl restart mariadb

# Verify limits
cat /proc/$(pgrep mariadbd)/limits | grep -E 'Max open files|Max locked memory'
```

### 7.3 Multiple Instances (mysqld_multi / systemd template)

**Option A: mysqld_multi (Legacy, still works)**

```ini
# /etc/my.cnf
[mysqld_multi]
mysqld     = /usr/bin/mariadbd
mysqladmin = /usr/bin/mariadb-admin
user       = mysql
log        = /var/log/mysql/mysqld_multi.log

[mysqld3306]
port       = 3306
datadir    = /var/lib/mysql3306
socket     = /run/mysqld/mysqld3306.sock
pid-file   = /run/mysqld/mysqld3306.pid
log-error  = /var/log/mysql/mariadb3306.err

[mysqld3307]
port       = 3307
datadir    = /var/lib/mysql3307
socket     = /run/mysqld/mysqld3307.sock
pid-file   = /run/mysqld/mysqld3307.pid
log-error  = /var/log/mysql/mariadb3307.err
```

```bash
# Initialize each datadir
sudo mariadb-install-db --datadir=/var/lib/mysql3306 --user=mysql
sudo mariadb-install-db --datadir=/var/lib/mysql3307 --user=mysql

# Start
mysqld_multi start 3306,3307
mysqld_multi stop 3306
mysqld_multi report
```

**Option B: systemd template (Modern)**

```bash
# Copy service template
sudo cp /usr/lib/systemd/system/mariadb.service /etc/systemd/system/mariadb@.service

# Edit /etc/systemd/system/mariadb@.service
# Ganti: 
# ExecStart=/usr/bin/mariadbd --defaults-file=/etc/my%I.cnf $MYSQLD_OPTS
# PIDFile=/run/mysqld/mysqld%I.pid

# Create config per instance: /etc/my3306.cnf, /etc/my3307.cnf
# Start
sudo systemctl start mariadb@3306 mariadb@3307
sudo systemctl enable mariadb@3306 mariadb@3307
```

---

## 8. Upgrade MariaDB (Minor Version)

### 8.1 In-Place Upgrade (Repository)

```bash
# Ubuntu/Debian
sudo apt update
sudo apt install --only-upgrade mariadb-server mariadb-client mariadb-backup

# RHEL/Rocky/Alma
sudo dnf update MariaDB-server MariaDB-client MariaDB-backup

# Restart & upgrade system tables
sudo systemctl restart mariadb
sudo mariadb-upgrade --force    # atau mysql_upgrade (deprecated)
```

### 8.2 Post-Upgrade Checklist

```bash
# 1. Cek error log
journalctl -u mariadb -p err --since "10 minutes ago"

# 2. Cek version
mariadb -u root -e "SELECT VERSION();"

# 3. Jalankan mysql_upgrade (mariadb-upgrade di 10.5+)
sudo mariadb-upgrade --force

# 4. Cek tabel system
mariadb -u root -e "CHECK TABLE mysql.user, mysql.db, mysql.tables_priv;"

# 5. Test aplikasi
# Run smoke tests / health checks
```

> **Major Upgrade (10.6 → 10.11 → 11.4):** Butuh logical dump/restore atau replication-based upgrade. Bisa `mariadb-dump --all-databases --single-transaction --master-data=2 --routines --triggers --events > full.sql` lalu restore ke version baru.

---

## 9. Praktikum: Install & Configure

### Lab 1: Fresh Install Ubuntu 24.04 + MariaDB 11.4
```bash
# 1. Add repo & install (ikuti 2.1)
# 2. Secure installation
sudo mariadb-secure-installation
# 3. Create custom config
sudo tee /etc/mysql/mariadb.conf.d/99-custom.cnf > /dev/null <<'EOF'
[mysqld]
innodb_buffer_pool_size = 1G
innodb_log_file_size = 256M
slow_query_log = 1
long_query_time = 1
log_bin = mariadb-bin
binlog_format = ROW
gtid_domain_id = 1
EOF
# 4. Restart & verify
sudo systemctl restart mariadb
mariadb -u root -p -e "SHOW VARIABLES LIKE 'innodb_buffer_pool_size'; SELECT VERSION();"
```

### Lab 2: Binary Tarball Second Instance (Port 3307)
```bash
# Ikuti section 3 di atas
# Verify dua instance jalan
ss -tlnp | grep -E '3306|3307'
mariadb -S /run/mysqld/mysqld3306.sock -u root -e "SELECT @@port;"
mariadb -S /run/mysqld/mysqld3307.sock -u root -e "SELECT @@port;"
```

### Lab 3: Config Validation & Reload
```bash
# Test config
mariadbd --defaults-file=/etc/mysql/mariadb.conf.d/99-custom.cnf --validate-config

# Runtime variable change (no restart)
mariadb -u root -p -e "SET GLOBAL long_query_time = 0.5; SHOW VARIABLES LIKE 'long_query_time';"

# Persist ke config
sudo sed -i 's/long_query_time = 1/long_query_time = 0.5/' /etc/mysql/mariadb.conf.d/99-custom.cnf
```

---

## 11. Troubleshooting Instalasi Umum

| Error | Penyebab | Solusi |
|-------|----------|--------|
| `Can't connect to local MySQL server through socket` | Socket path mismatch / server down | Cek `socket` di my.cnf & client, start service |
| `Access denied for user 'root'@'localhost'` | unix_socket plugin / password | `sudo mariadb -u root` atau reset password |
| `InnoDB: Error: log file ./ib_logfile0 is of different size` | `innodb_log_file_size` berubah | Stop server, hapus `ib_logfile*`, start (data aman) |
| `[ERROR] Could not increase number of max_open_files` | OS limit terlalu rendah | `systemctl edit mariadb` → `LimitNOFILE=65535` |
| `Table 'mysql.user' doesn't exist` | `mysql_upgrade` belum jalan | `sudo mariadb-upgrade --force` |
| `Unknown variable 'query_cache_size'` | Deprecated di 10.6+ | Hapus/comment baris tersebut |
| `InnoDB: Cannot allocate memory for the buffer pool` | `innodb_buffer_pool_size` > RAM | Kurangi nilai, set max 80% RAM |

---

## 12. Referensi Lanjutan

| Topik | Resource |
|-------|----------|
| MariaDB Repositories | https://mariadb.org/download/?t=repo |
| Configuration Options | https://mariadb.com/kb/en/server-system-variables/ |
| Systemd Management | https://mariadb.com/kb/en/systemd/ |
| mysqld_multi | https://mariadb.com/kb/en/mysqld_multi/ |
| Upgrade Guide | https://mariadb.com/kb/en/upgrading-between-major-mariadb-versions/ |
| Binary Tarball Install | https://mariadb.com/kb/en/installing-mariadb-binary-tarballs/ |

---

> **Next:** [Phase 1.3: Koneksi & Autentikasi](../1.3-autentikasi/koneksi-autentikasi.md)