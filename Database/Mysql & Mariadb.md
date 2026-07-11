# 01. MySQL / MariaDB — Cài đặt, Tối ưu, Bảo mật

## Mục lục

- [I. MySQL vs MariaDB](#i-mysql-vs-mariadb)
- [II. Cài đặt trên Ubuntu 22.04](#ii-cài-đặt-trên-ubuntu-2204)
  - [2.1. Cài đặt MariaDB](#21-cài-đặt-mariadb)
  - [2.2. Cài đặt MySQL 8.0](#22-cài-đặt-mysql-80)
- [III. Tối ưu hiệu năng](#iii-tối-ưu-hiệu-năng)
  - [3.1. Kiểm tra Slow Query](#31-kiểm-tra-slow-query)
  - [3.2. EXPLAIN — phân tích execution plan](#32-explain--phân-tích-execution-plan)
  - [3.3. InnoDB Buffer Pool](#33-innodb-buffer-pool)
  - [3.4. Connection Pool](#34-connection-pool)
- [IV. Bảo mật MySQL/MariaDB](#iv-bảo-mật-mysqlmariadb)
  - [4.1. Dọn user nguy hiểm](#41-dọn-user-nguy-hiểm)
  - [4.2. Password Policy](#42-password-policy)
  - [4.3. Ngăn remote root login & bind address](#43-ngăn-remote-root-login--bind-address)
- [ A: Microsoft SQL Server](#a-microsoft-sql-server)
  - [A.1. Thông tin cơ bản](#a1-thông-tin-cơ-bản)
  - [A.2. Cài đặt và cấu hình trên Ubuntu 22.04](#a2-cài-đặt-và-cấu-hình-trên-ubuntu-2204)
  - [A.3. Cấu hình hiệu năng](#a3-cấu-hình-hiệu-năng)
  - [A.4. Bảo mật cơ bản](#a4-bảo-mật-cơ-bản)
- [ B: Redis](#b-redis)
  - [B.1. Tổng quan](#b1-tổng-quan)
  - [B.2. Cài đặt Redis 7.x trên Ubuntu 22.04](#b2-cài-đặt-redis-7x-trên-ubuntu-2204)
  - [B.3. Cấu hình `/etc/redis/redis.conf`](#b3-cấu-hình-etcredisredisconf)
  - [B.4. Bảo mật Redis](#b4-bảo-mật-redis)

---

## I. MySQL vs MariaDB

MariaDB là một fork của MySQL, ra đời năm 2009 sau khi Oracle mua lại Sun Microsystems. Cú pháp SQL của MariaDB gần như tương thích hoàn toàn với MySQL, nên người dùng MySQL có thể chuyển sang MariaDB rất dễ dàng.

| Tiêu chí | MySQL 8.0 | MariaDB 11.x |
|---|---|---|
| **Nguồn gốc** | Oracle Corporation phát triển | Fork từ MySQL bởi cộng đồng MariaDB Foundation |
| **License** | GPL + Oracle Commercial | GPL thuần — hoàn toàn mã nguồn mở |
| **Storage Engine** | InnoDB (mặc định) | InnoDB + Aria + MyRocks + nhiều engine khác |
| **JSON Support** | Native JSON type mạnh hơn | Hỗ trợ JSON đủ dùng, đang cải thiện dần |
| **Replication** | Replication tiêu chuẩn | Hỗ trợ Galera Cluster Multi-Master tốt hơn |
| **Hiệu năng** | Tối ưu tốt cho workload phổ biến | Nhiều benchmark cho thấy nhanh hơn MySQL |
| **Cloud Support** | Hỗ trợ mạnh trên AWS RDS, Azure | Ít managed service hơn |
| **Độ phổ biến** | Rất phổ biến trong enterprise | Phổ biến trong môi trường self-hosted |
| **Dùng khi nào** | Cloud managed, enterprise standard | Self-hosted, full open-source, cluster |

---

## II. Cài đặt trên Ubuntu 22.04

### 2.1. Cài đặt MariaDB

**Bước 1: Cài MariaDB từ official repo**
```bash
sudo apt update
sudo apt install -y mariadb-server mariadb-client
```

**Bước 2: Khởi động & bật auto-start**
```bash
sudo systemctl start mariadb
sudo systemctl enable mariadb
sudo systemctl status mariadb
```

**Bước 3: Chạy script bảo mật ban đầu**
```bash
sudo mysql_secure_installation
```
Trả lời theo hướng dẫn:
- `Enter current root password:` → Enter (để trống lần đầu)
- `Switch to unix_socket auth?` → n
- `Change root password?` → y → đặt password mạnh
- `Remove anonymous users?` → y
- `Disallow root login remotely?` → y
- `Remove test database?` → y
- `Reload privilege tables?` → y

**Bước 4: Đăng nhập kiểm tra**
```sql
mysql -u root -p
SELECT VERSION();
SHOW DATABASES;
EXIT;
```

### 2.2. Cài đặt MySQL 8.0

**Bước 1: Cài MySQL**
```bash
sudo apt update
sudo apt install mysql-server -y
```

**Bước 2: Khởi động & bảo mật**
```bash
sudo systemctl enable --now mysql
sudo mysql_secure_installation
```

**Bước 3: Đăng nhập**
```sql
sudo mysql -u root
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'StrongPass!';
FLUSH PRIVILEGES;
```

---

## III. Tối ưu hiệu năng

### 3.1. Kiểm tra Slow Query

Sau khi bật `slow_query_log`:
```sql
-- Top 10 query chậm nhất (chạy ở shell, không phải trong mysql)
-- sudo mysqldumpslow -s t -t 10 /var/log/mysql/mysql-slow.log

-- Xem query đang chạy
SHOW FULL PROCESSLIST;

-- Query chạy lâu hơn 5 giây
SELECT id, user, host, db, time, state, LEFT(info,80)
FROM information_schema.PROCESSLIST
WHERE time > 5
ORDER BY time DESC;
```

### 3.2. EXPLAIN — phân tích execution plan

```sql
EXPLAIN
SELECT *
FROM orders o
JOIN customers c ON o.customer_id = c.id
WHERE o.status = 'pending';
```

Xem cột `type` trong kết quả:
- `const` / `eq_ref` — tốt nhất (dùng index)
- `ref` / `range` — chấp nhận được
- `ALL` — đọc toàn bộ bảng → **cần thêm index!**

```sql
CREATE INDEX idx_orders_status ON orders(status);
CREATE INDEX idx_orders_customer ON orders(customer_id);
```

### 3.3. InnoDB Buffer Pool

```sql
SHOW STATUS LIKE 'Innodb_buffer_pool_%';

-- Tính hit rate (nên > 99%)
SELECT
  (1 - Innodb_buffer_pool_reads / Innodb_buffer_pool_read_requests) * 100
  AS buffer_pool_hit_rate;
```

### 3.4. Connection Pool

```sql
-- Xem max connections hiện tại
SHOW VARIABLES LIKE 'max_connections';

-- Xem peak connections từ khi khởi động
SHOW STATUS LIKE 'Max_used_connections';

-- Thay đổi không cần restart (tạm thời)
SET GLOBAL max_connections = 300;

-- Xem connections đang dùng
SHOW STATUS LIKE 'Threads_connected';
```

---

## IV. Bảo mật MySQL/MariaDB

### 4.1. Dọn user nguy hiểm

```sql
-- Xem tất cả user và host
SELECT user, host, plugin, password_expired
FROM mysql.user ORDER BY user;

-- Xóa anonymous user (nếu còn sót)
DELETE FROM mysql.user WHERE user = '';

-- Xóa user không cần thiết
DROP USER ''@'localhost';
DROP USER ''@'%';
FLUSH PRIVILEGES;
```

### 4.2. Password Policy

```sql
-- Bật plugin validate_password
INSTALL PLUGIN validate_password SONAME 'validate_password.so';

-- Cấu hình mức độ (MEDIUM yêu cầu chữ hoa, số, ký tự đặc biệt)
SET GLOBAL validate_password_policy = 'MEDIUM';
SET GLOBAL validate_password_length = 12;
```
Thêm vào `my.cnf` để cấu hình tồn tại sau restart:
```ini
[mysqld]
validate_password_policy = MEDIUM
validate_password_length = 12
```

### 4.3. Ngăn remote root login & bind address

```sql
-- Đảm bảo root chỉ login từ localhost
UPDATE mysql.user SET host = 'localhost'
WHERE user = 'root' AND host != 'localhost';
FLUSH PRIVILEGES;
```

Trong `/etc/mysql/mariadb.conf.d/50-server.cnf`:
```ini
# Chỉ lắng nghe localhost (không nhận kết nối từ ngoài)
bind-address = 127.0.0.1
# Nếu cần remote app kết nối: bind-address = 0.0.0.0
# nhưng phải kết hợp firewall ufw
```

---

##  A: Microsoft SQL Server

> Không nằm trong 3 DB trọng tâm của bộ tài liệu này, nhưng được giữ lại làm  vì có nội dung thực hành thật khá đầy đủ (cài đặt, tối ưu, bảo mật trên Ubuntu 22.04).

### A.1. Thông tin cơ bản

| Thuộc tính | Giá trị |
|---|---|
| Nhà phát triển | Microsoft |
| Phiên bản mới nhất | SQL Server 2022 |
| Chạy trên | Windows, Linux, Docker |
| Default port | 1433 |
| License | Trả phí |

**Luồng xử lý query:** App gửi query → SQL Server Engine nhận → Query Optimizer chọn execution plan tối ưu → đọc/ghi data qua Buffer Pool (RAM cache) → nếu data chưa có trong RAM thì mới đọc từ disk.

### A.2. Cài đặt và cấu hình trên Ubuntu 22.04

**Bước 1: Import GPG key & thêm repository**
```bash
curl -fsSL https://packages.microsoft.com/keys/microsoft.asc | sudo tee /etc/apt/trusted.gpg.d/microsoft.asc
curl -fsSL https://packages.microsoft.com/config/ubuntu/22.04/mssql-server-2022.list | sudo tee /etc/apt/sources.list.d/mssql-server-2022.list
```

**Bước 2: Cài SQL Server**
```bash
sudo apt-get update
sudo apt-get install -y mssql-server
```

**Bước 3: Chạy setup ban đầu**
```bash
sudo /opt/mssql/bin/mssql-conf setup
```
Wizard sẽ hỏi: (1) Choose edition — Developer (free) → nhập "2"; (2) Accept license — Yes; (3) SA password — đặt password phức tạp (≥8 ký tự).

> SA là tài khoản sysadmin cấp cao nhất — đặt password mạnh và không dùng SA cho app thông thường.

**Bước 4: Kiểm tra service**
```bash
sudo systemctl status mssql-server
sudo systemctl enable mssql-server   # auto-start khi reboot
```

**Bước 5: Cài `sqlcmd`** — công cụ CLI thao tác DB
```bash
curl -fsSL https://packages.microsoft.com/config/ubuntu/22.04/prod.list | sudo tee /etc/apt/sources.list.d/mssql-tools.list
sudo apt-get update
sudo apt-get install -y mssql-tools18 unixodbc-dev
echo 'export PATH="$PATH:/opt/mssql-tools18/bin"' >> ~/.bashrc
source ~/.bashrc
```

**Bước 6: Test kết nối lần đầu**
```bash
sqlcmd -S localhost -U SA -P 'YourStrongPassword' -C
```
Trong prompt `sqlcmd`:
```sql
SELECT @@VERSION
GO
```

### A.3. Cấu hình hiệu năng

File cấu hình: `/var/opt/mssql/mssql.conf`

**1. Giới hạn RAM — quan trọng nhất**
```ini
[memory]
memorylimitmb = 4096   # Giới hạn 4GB
```
Hoặc:
```bash
sudo /opt/mssql/bin/mssql-conf set memory.memorylimitmb 4096
```

**2. Tách data, log, backup ra ổ đĩa riêng**
```bash
sudo mkdir -p /mnt/data/mssql/{data,log,backup}
sudo chown mssql:mssql /mnt/data/mssql/{data,log,backup}
sudo /opt/mssql/bin/mssql-conf set filelocation.defaultdatadir /mnt/data/mssql/data
sudo /opt/mssql/bin/mssql-conf set filelocation.defaultlogdir /mnt/data/mssql/log
sudo /opt/mssql/bin/mssql-conf set filelocation.defaultbackupdir /mnt/data/mssql/backup
sudo systemctl restart mssql-server
```

**3. Kiểm tra hiệu năng thực tế**
```sql
-- Xem các query đang chạy chậm nhất
SELECT TOP 10 total_elapsed_time / execution_count AS avg_ms, execution_count,
       SUBSTRING(st.text, 1, 100) AS query_text
FROM sys.dm_exec_query_stats qs
CROSS APPLY sys.dm_exec_sql_text(qs.sql_handle) st
ORDER BY avg_ms DESC;

-- Xem memory usage hiện tại
SELECT physical_memory_in_use_kb / 1024 AS memory_mb, page_fault_count
FROM sys.dm_os_process_memory;
```

### A.4. Bảo mật cơ bản

**1. Vô hiệu hóa tài khoản SA**
```sql
CREATE LOGIN dba_admin WITH PASSWORD = 'Str0ngP@ssw0rd!';
ALTER SERVER ROLE sysadmin ADD MEMBER dba_admin;

-- Tạo login riêng cho từng app (quyền tối thiểu)
CREATE LOGIN app_myapp WITH PASSWORD = 'AppP@ss456!';

-- Disable SA
ALTER LOGIN SA DISABLE;
ALTER LOGIN SA WITH NAME = sa_disabled;
```

**2. Cấp quyền đúng mức**
```sql
USE MyAppDB;
CREATE USER app_myapp FOR LOGIN app_myapp;
GRANT SELECT, INSERT, UPDATE, DELETE ON SCHEMA::dbo TO app_myapp;
-- LƯU Ý: không bao giờ cấp các quyền phá hoại như DROP, ALTER, TRUNCATE cho app.
```

**3. Bật mã hóa kết nối TLS**
```bash
# Tạo chứng chỉ tự ký (self-signed) cho môi trường Dev
openssl req -x509 -nodes -newkey rsa:2048 \
  -subj "/CN=mssql-server" \
  -keyout /etc/ssl/private/mssql.key \
  -out /etc/ssl/certs/mssql.pem -days 365

sudo chown mssql:mssql /etc/ssl/private/mssql.key
sudo chmod 600 /etc/ssl/private/mssql.key

sudo /opt/mssql/bin/mssql-conf set network.tlscert /etc/ssl/certs/mssql.pem
sudo /opt/mssql/bin/mssql-conf set network.tlskey /etc/ssl/private/mssql.key
sudo /opt/mssql/bin/mssql-conf set network.tlsprotocols 1.2
sudo /opt/mssql/bin/mssql-conf set network.forceencryption 1

sudo systemctl restart mssql-server
```

**4. Firewall**
```bash
sudo ufw enable
sudo ufw allow from 10.0.1.50 to any port 1433
sudo ufw status verbose
```

**5. Bật audit log**
```sql
CREATE SERVER AUDIT SecurityAudit
TO FILE (FILEPATH = '/var/opt/mssql/audit/');

CREATE SERVER AUDIT SPECIFICATION AuditLogins
FOR SERVER AUDIT SecurityAudit
ADD (FAILED_LOGIN_GROUP);

ALTER SERVER AUDIT SecurityAudit WITH (STATE = ON);
ALTER SERVER AUDIT SPECIFICATION AuditLogins WITH (STATE = ON);
```

---

##  B: Redis

> Không nằm trong 3 DB trọng tâm, nhưng giữ lại làm  vì có nội dung thực hành thật (cài đặt, cấu hình, bảo mật). Redis thường đi kèm MySQL trong kiến trúc web (App → Redis cache → MySQL) nên đặt chung  ở đây hợp lý.

### B.1. Tổng quan

Redis là In-Memory Database — dữ liệu sống trong RAM. Redis không thay thế MySQL — nó là lớp đứng phía trước để giảm tải: kiến trúc phổ biến nhất là **App → Redis (cache hit?) → MySQL (cache miss)**.

- **Tốc độ siêu nhanh:** mọi thao tác đọc/ghi diễn ra trực tiếp trên RAM, độ trễ tính bằng micro giây.
- **Cấu trúc dữ liệu phong phú:** khác Memcached (chỉ lưu chuỗi đơn giản), Redis hỗ trợ Strings, Lists, Sets, Hashes, Sorted Sets.
- **Tính bền bỉ:** dù chạy trên RAM, Redis vẫn có cơ chế snapshot định kỳ + ghi log thao tác xuống ổ cứng — mất điện vẫn khôi phục được khi khởi động lại.

### B.2. Cài đặt Redis 7.x trên Ubuntu 22.04

```bash
# Thêm Redis official repo
curl -fsSL https://packages.redis.io/gpg \
  | sudo gpg --dearmor -o /usr/share/keyrings/redis-archive-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/redis-archive-keyring.gpg] \
https://packages.redis.io/deb $(lsb_release -cs) main" \
  | sudo tee /etc/apt/sources.list.d/redis.list

sudo apt update
sudo apt install -y redis
```

**Khởi động & kiểm tra:**
```bash
sudo systemctl start redis-server
sudo systemctl enable redis-server
sudo systemctl status redis-server
```

**Test bằng `redis-cli`:**
```
redis-cli
PING                    # → PONG 
SET hello "world"       # lưu key-value
GET hello                # → "world"
INFO server              # thông tin server
INFO memory               # dùng bao nhiêu RAM
DBSIZE                    # số key đang có
```

**Benchmark:**
```bash
redis-benchmark -n 100000 -q -P 16
# Kết quả thường thấy:
# SET:   ~800.000 requests/second
# GET:   ~900.000 requests/second
# LPUSH: ~700.000 requests/second
```

### B.3. Cấu hình `/etc/redis/redis.conf`

```ini
# ── Network ─────────────────────────────────────────
bind            127.0.0.1        # chỉ localhost. Thêm IP app nếu remote
port            6379
protected-mode yes               # từ chối kết nối nếu không có auth
tcp-backlog    511

# ── Auth ─────────────────────────────────────────────
requirepass    StrongRedisPass123!  # BẮT BUỘC đặt password

# ── Memory ───────────────────────────────────────────
maxmemory      2gb               # giới hạn RAM tối đa
maxmemory-policy allkeys-lru    # khi đầy: xóa key ít dùng nhất
#   allkeys-lru   = xóa bất kỳ key ít dùng (cache thuần)
#   volatile-lru  = chỉ xóa key có TTL (mix cache+persistent)
#   volatile-ttl  = xóa key sắp hết TTL nhất
#   noeviction    = từ chối write khi đầy (không dùng cho cache)

# ── Persistence ──────────────────────────────────────
# RDB Snapshot — backup nhanh, có thể mất data gần nhất
save            900 1            # sau 900s nếu có ≥1 thay đổi
save            300 10           # sau 300s nếu có ≥10 thay đổi
save            60  10000        # sau 60s nếu có ≥10000 thay đổi
dbfilename      dump.rdb
dir             /var/lib/redis

# AOF — log mọi lệnh, an toàn hơn nhưng tốn disk
appendonly      yes
appendfsync     everysec         # flush mỗi giây (cân bằng an toàn/hiệu năng)
#   always   = flush mỗi lệnh (an toàn nhất, chậm nhất)
#   everysec = flush mỗi giây (khuyên dùng)
#   no       = OS quyết định (nhanh nhất, nguy hiểm)

# ── Performance ──────────────────────────────────────
maxclients      1000             # số connection đồng thời tối đa
tcp-keepalive   300
lazyfree-lazy-eviction yes      # xóa key lớn không đồng bộ (tránh block)

# ── Logging ──────────────────────────────────────────
loglevel        notice           # verbose | notice | warning
logfile         /var/log/redis/redis-server.log

# ── TLS (production) ─────────────────────────────────
# tls-port 6380
# tls-cert-file /etc/ssl/redis/redis.crt
# tls-key-file  /etc/ssl/redis/redis.key
```

### B.4. Bảo mật Redis

> Redis mặc định không có password và lắng nghe `0.0.0.0` trên nhiều distro. Kết hợp với port 6379 hay bị scan liên tục từ internet → rất nguy hiểm. Làm phần này ngay sau khi cài.

**1. Đặt Password — bắt buộc**
```bash
# Trong redis.conf:
requirepass Hieucute123!

# Hoặc đặt ngay không cần restart (tạm thời):
redis-cli
CONFIG SET requirepass "Hieucute123!"

# Sau khi đặt password, đăng nhập:
redis-cli -a Hieucute123!
# hoặc sau khi kết nối: AUTH Hieucute123!

# Lưu vào file config để persistent:
redis-cli -a Hieucute123! CONFIG REWRITE
```

**2. Bind + Firewall — không để lộ port 6379**
```bash
# redis.conf — chỉ lắng nghe localhost
bind 127.0.0.1

# Nếu app server ở IP khác (ví dụ 10.0.1.50):
bind 127.0.0.1 10.0.1.10   # 10.0.1.10 = IP của Redis server
sudo ufw allow from 10.0.1.50 to any port 6379
sudo ufw enable

# Kiểm tra port đang listen ở đâu
ss -tlnp | grep 6379
# An toàn: 127.0.0.1:6379
# Nguy hiểm: 0.0.0.0:6379
```

**3. Rename hoặc disable lệnh nguy hiểm**
```ini
rename-command FLUSHALL ""                     # disable hoàn toàn
rename-command FLUSHDB  ""                     # disable hoàn toàn
rename-command CONFIG   "HIDDEN_CONFIG_9x2k"   # đổi tên khó đoán
rename-command DEBUG    ""                     # disable
rename-command KEYS     "SCAN_KEYS"            # khuyến khích dùng SCAN thay KEYS
```

**4. Dùng ACL (Redis 6+) — phân quyền chi tiết theo user**
```bash
redis-cli -a Hieucute123!

# Tạo user chỉ đọc được key "cache:*"
ACL SETUSER readonly_user on >ReadOnlyPass! ~cache:* +GET +MGET +EXISTS +TTL -@all

# Tạo user cho app — chỉ SET/GET, không FLUSH
ACL SETUSER app_user on >AppPass456! ~* +@read +@write -FLUSHDB -FLUSHALL -CONFIG -DEBUG

# Xem danh sách ACL
ACL LIST
```
