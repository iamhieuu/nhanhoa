# 03. PostgreSQL — Cài đặt, Đặc điểm riêng, So sánh với MySQL


## Mục lục

- [I. Tổng quan PostgreSQL](#i-tổng-quan-postgresql)
- [II. Cài đặt trên Ubuntu 22.04](#ii-cài-đặt-trên-ubuntu-2204)
  - [2.1. Cách 1 — Cài từ repo mặc định của Ubuntu](#21-cách-1--cài-từ-repo-mặc-định-của-ubuntu)
  - [2.2. Cách 2 — Cài bản mới nhất từ PGDG repository (khuyến nghị)](#22-cách-2--cài-bản-mới-nhất-từ-pgdg-repository-khuyến-nghị)
  - [2.3. Kiểm tra sau khi cài](#23-kiểm-tra-sau-khi-cài)
- [III. Kiến trúc & khái niệm đặc trưng](#iii-kiến-trúc--khái-niệm-đặc-trưng)
- [IV. Cấu hình cơ bản](#iv-cấu-hình-cơ-bản)
  - [4.1. `postgresql.conf`](#41-postgresqlconf)
  - [4.2. `pg_hba.conf` — kiểm soát xác thực](#42-pg_hbaconf--kiểm-soát-xác-thực)
  - [4.3. Cho phép kết nối remote](#43-cho-phép-kết-nối-remote)
- [V. Quản lý Role, Database, thao tác cơ bản](#v-quản-lý-role-database-thao-tác-cơ-bản)
- [VI. Bảo mật](#vi-bảo-mật)
- [VII. Tối ưu hiệu năng cơ bản](#vii-tối-ưu-hiệu-năng-cơ-bản)
- [VIII. Backup & Restore](#viii-backup--restore)

---

## I. Tổng quan PostgreSQL

PostgreSQL (thường gọi tắt là **Postgres**) là hệ quản trị cơ sở dữ liệu quan hệ mã nguồn mở, khởi nguồn từ dự án POSTGRES tại Đại học California, Berkeley từ năm 1986. Đây là một trong những RDBMS lâu đời và được đánh giá cao nhất về tính toàn vẹn dữ liệu, khả năng mở rộng và mức độ tuân thủ chuẩn SQL.

**Đặc điểm nổi bật:**
- Tuân thủ chuẩn SQL rất nghiêm ngặt (ANSI SQL).
- Hỗ trợ kiểu dữ liệu phong phú: JSON/JSONB, Array, hstore (key-value), UUID, kiểu hình học (geometric), range types...
- Hỗ trợ **extension** mạnh — nổi bật nhất là **PostGIS** (dữ liệu không gian địa lý/GIS).
- Có thể định nghĩa hàm bằng nhiều ngôn ngữ (PL/pgSQL, PL/Python, PL/Perl...).
- Hỗ trợ **MVCC** (Multi-Version Concurrency Control) tương tự InnoDB, cho phép đọc/ghi đồng thời không block lẫn nhau.
- Mỗi kết nối được xử lý bằng 1 **process** riêng (khác MySQL dùng thread) — an toàn hơn nhưng tốn RAM hơn khi có nhiều kết nối đồng thời.

---

## II. Cài đặt trên Ubuntu 22.04

### 2.1. Cách 1 — Cài từ repo mặc định của Ubuntu

Đơn giản nhất, nhưng phiên bản PostgreSQL đi kèm Ubuntu 22.04 (jammy) thường **không phải bản mới nhất** (thường là PostgreSQL 14):

```bash
sudo apt update
sudo apt install -y postgresql postgresql-contrib
```

`postgresql-contrib` chứa các extension bổ sung hữu ích (pg_stat_statements, uuid-ossp...).

### 2.2. Cách 2 — Cài bản mới nhất từ PGDG repository (khuyến nghị)

PGDG (PostgreSQL Global Development Group) duy trì repo APT riêng, luôn cập nhật bản mới nhất cho các phiên bản Ubuntu LTS, kể cả 22.04 (jammy):

```bash
# Cài công cụ cần thiết
sudo apt install -y curl ca-certificates

# Import GPG signing key
sudo install -d /usr/share/postgresql-common/pgdg
sudo curl -o /usr/share/postgresql-common/pgdg/apt.postgresql.org.asc \
  --fail https://www.postgresql.org/media/keys/ACCC4CF8.asc

# Thêm repository (thay "jammy" nếu dùng bản Ubuntu khác)
echo "deb [signed-by=/usr/share/postgresql-common/pgdg/apt.postgresql.org.asc] \
https://apt.postgresql.org/pub/repos/apt jammy-pgdg main" | \
  sudo tee /etc/apt/sources.list.d/pgdg.list

# Cập nhật & cài PostgreSQL 16 (phiên bản ổn định phổ biến hiện tại)
sudo apt update
sudo apt install -y postgresql-16 postgresql-contrib-16
```

### 2.3. Kiểm tra sau khi cài

```bash
sudo systemctl status postgresql
sudo systemctl enable postgresql

# Xem cluster (instance) PostgreSQL đang chạy
pg_lsclusters

# Đăng nhập lần đầu — mặc định dùng peer authentication qua user hệ thống "postgres"
sudo -u postgres psql
```
Trong prompt `psql`:
```sql
SELECT version();
\du       -- danh sách role/user
\l        -- danh sách database
\q        -- thoát
```

---

## III. Kiến trúc & khái niệm đặc trưng

| Khái niệm | Ý nghĩa |
|---|---|
| **Cluster** | Một instance PostgreSQL quản lý nhiều database, tương ứng 1 thư mục data directory (`/var/lib/postgresql/<version>/main`). |
| **Role** | PostgreSQL không phân biệt "user" và "group" — tất cả đều là **Role**. Role có thể có quyền LOGIN (như user) hoặc không (dùng như group). |
| **Schema** | Không gian tên trong 1 database, dùng để nhóm bảng/view (mặc định là `public`). Khác MySQL — nơi mỗi "database" gần như tương đương 1 schema riêng biệt. |
| **Tablespace** | Vị trí vật lý trên đĩa để lưu dữ liệu — cho phép đặt các bảng lớn/ít dùng ra ổ đĩa khác. |
| **WAL (Write-Ahead Log)** | Tương đương Binary Log/Redo Log của MySQL — ghi log thay đổi trước khi ghi vào data file, phục vụ crash recovery và replication. |
| **VACUUM** | Cơ chế dọn dẹp các dòng dữ liệu "chết" (dead tuples) sinh ra do MVCC khi UPDATE/DELETE — cần chạy định kỳ (autovacuum mặc định đã bật) để tránh phình bảng và index bị chậm. |

---

## IV. Cấu hình cơ bản

### 4.1. `postgresql.conf`

Đường dẫn: `/etc/postgresql/16/main/postgresql.conf`

Các tham số hay chỉnh nhất:
```ini
listen_addresses = 'localhost'      # đổi thành '*' nếu cần remote (kết hợp pg_hba.conf)
port = 5432
max_connections = 100

shared_buffers = 256MB              # ~25% RAM server (RAM 1GB → 256MB là hợp lý)
work_mem = 4MB                      # bộ nhớ cho mỗi thao tác sort/hash
maintenance_work_mem = 64MB         # bộ nhớ cho VACUUM, CREATE INDEX

wal_level = replica                 # cần "replica" hoặc "logical" nếu dùng replication
logging_collector = on
log_directory = 'log'
log_min_duration_statement = 1000   # log query chạy > 1000ms (tương đương slow query log MySQL)
```
Sau khi sửa, áp dụng bằng:
```bash
sudo systemctl restart postgresql
```

### 4.2. `pg_hba.conf` — kiểm soát xác thực

Đường dẫn: `/etc/postgresql/16/main/pg_hba.conf`. Đây là điểm khác biệt lớn nhất so với MySQL — PostgreSQL kiểm soát **ai được kết nối từ đâu, bằng phương thức xác thực nào** hoàn toàn qua file riêng này, không gộp chung vào bảng user như MySQL:

```
# TYPE  DATABASE  USER  ADDRESS         METHOD
local   all       all                   peer
host    all       all   127.0.0.1/32    scram-sha-256
host    all       all   10.0.1.0/24     scram-sha-256
```

- `local` — kết nối qua Unix socket (từ chính server), thường dùng `peer` (khớp user hệ điều hành).
- `host` — kết nối qua TCP/IP.
- `METHOD` phổ biến: `trust` (không cần password — chỉ dùng nội bộ/dev), `peer` (khớp user OS), `scram-sha-256` (xác thực password, khuyến nghị cho production), `reject` (từ chối).

### 4.3. Cho phép kết nối remote

```bash
# 1. Sửa postgresql.conf
sudo sed -i "s/^listen_addresses.*/listen_addresses = '*'/" /etc/postgresql/16/main/postgresql.conf

# 2. Thêm dòng cho phép IP app server trong pg_hba.conf
echo "host    all    all    10.0.1.50/32    scram-sha-256" | sudo tee -a /etc/postgresql/16/main/pg_hba.conf

# 3. Restart
sudo systemctl restart postgresql

# 4. Mở firewall
sudo ufw allow from 10.0.1.50 to any port 5432
```

---

## V. Quản lý Role, Database, thao tác cơ bản

```sql
-- Tạo role có thể login (tương đương "user")
CREATE ROLE app_user WITH LOGIN PASSWORD 'StrongPass123!';

-- Tạo database
CREATE DATABASE my_app_db OWNER app_user;

-- Cấp quyền cụ thể (thay vì cấp toàn bộ như superuser)
GRANT CONNECT ON DATABASE my_app_db TO app_user;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO app_user;

-- Đảm bảo bảng tạo SAU này cũng tự động cấp quyền tương tự
ALTER DEFAULT PRIVILEGES IN SCHEMA public
  GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO app_user;
```

**CRUD cơ bản** (cú pháp gần giống chuẩn SQL/MySQL):
```sql
CREATE TABLE customers (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    email TEXT UNIQUE,
    created_at TIMESTAMP DEFAULT now()
);

INSERT INTO customers (name, email) VALUES ('Nguyen A', 'a@example.com');
SELECT * FROM customers WHERE email = 'a@example.com';
UPDATE customers SET name = 'Nguyen B' WHERE id = 1;
DELETE FROM customers WHERE id = 1;
```

> **Khác biệt cần lưu ý:** PostgreSQL dùng `SERIAL`/`IDENTITY` thay cho `AUTO_INCREMENT` của MySQL; dùng `TEXT` thoải mái (không giới hạn độ dài như `VARCHAR` kiểu MySQL cũ); phân biệt hoa-thường tên bảng/cột nếu đặt trong dấu ngoặc kép `"TenBang"`.

---

## VI. Bảo mật

```sql
-- Đổi password superuser mặc định
ALTER USER postgres WITH PASSWORD 'NewStrongPassword!';

-- Không cấp quyền SUPERUSER cho user ứng dụng
CREATE ROLE readonly_user WITH LOGIN PASSWORD 'ReadOnlyPass!';
GRANT CONNECT ON DATABASE my_app_db TO readonly_user;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO readonly_user;
```

- **Bind địa chỉ:** giữ `listen_addresses = 'localhost'` nếu không cần remote; nếu cần, luôn giới hạn IP cụ thể trong `pg_hba.conf` thay vì mở `0.0.0.0/0`.
- **SSL/TLS:** bật bằng `ssl = on` trong `postgresql.conf`, kèm `ssl_cert_file`/`ssl_key_file`, rồi ép buộc kết nối SSL bằng method `hostssl` thay vì `host` trong `pg_hba.conf`.
- **Row-Level Security (RLS):** tính năng đặc trưng của PostgreSQL — cho phép giới hạn quyền truy cập theo từng dòng dữ liệu (không chỉ theo bảng), hữu ích cho hệ thống multi-tenant.
- **Audit:** dùng extension `pgaudit` để ghi log chi tiết theo chuẩn audit (tương tự Audit Log của MSSQL/MySQL Enterprise).

---

## VII. Tối ưu hiệu năng cơ bản

```sql
-- Tương đương EXPLAIN của MySQL, kèm ANALYZE để chạy thật và đo thời gian
EXPLAIN ANALYZE
SELECT * FROM orders WHERE status = 'pending';

-- Tạo index
CREATE INDEX idx_orders_status ON orders(status);

-- Xem các query đang chạy
SELECT pid, now() - query_start AS duration, query
FROM pg_stat_activity
WHERE state = 'active'
ORDER BY duration DESC;

-- Kiểm tra cache hit ratio (nên > 99%, tương đương buffer pool hit rate MySQL)
SELECT
  sum(heap_blks_hit) / nullif(sum(heap_blks_hit) + sum(heap_blks_read), 0) AS cache_hit_ratio
FROM pg_statio_user_tables;

-- Chạy VACUUM thủ công nếu nghi ngờ bảng bị phình (autovacuum thường tự lo việc này)
VACUUM ANALYZE orders;
```

---

## VIII. Backup & Restore

```bash
# Backup 1 database ra file SQL (tương đương mysqldump)
pg_dump -U postgres -d my_app_db -F c -f my_app_db.backup

# Backup toàn bộ cluster (mọi database + role)
pg_dumpall -U postgres -f full_cluster.sql

# Restore từ file backup dạng custom (-F c)
pg_restore -U postgres -d my_app_db -c my_app_db.backup

# Restore từ file SQL thuần
psql -U postgres -d my_app_db -f my_app_db.sql
```

> Với hệ thống production cần **Point-in-Time Recovery (PITR)**, PostgreSQL dùng cơ chế WAL archiving kết hợp base backup (`pg_basebackup`) — tương tự Binary Log + full backup của MySQL.

---
