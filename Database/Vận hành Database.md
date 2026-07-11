# 05. Vận hành Database — Backup, Replication, HA, Giám sát, Bottleneck, Deadlock, Log

## Mục lục

- [I. User và Quản lý quyền truy cập](#i-user-và-quản-lý-quyền-truy-cập)
- [II. Sao lưu và Phục hồi (Backup & Recovery)](#ii-sao-lưu-và-phục-hồi-backup--recovery)
- [III. Giám sát (Monitoring)](#iii-giám-sát-monitoring)
  - [3.1. Theo dõi hiệu năng nhanh theo từng DB](#31-theo-dõi-hiệu-năng-nhanh-theo-từng-db)
  - [3.2. Bảng so sánh công cụ giám sát](#32-bảng-so-sánh-công-cụ-giám-sát)
  - [3.3. Metric quan trọng — MySQL](#33-metric-quan-trọng--mysql)
  - [3.4. Cài Prometheus + mysqld_exporter + Grafana](#34-cài-prometheus--mysqld_exporter--grafana)
- [IV. Tối ưu hóa truy vấn (Query Optimization)](#iv-tối-ưu-hóa-truy-vấn-query-optimization)
- [V. Quản lý Transaction và Lock](#v-quản-lý-transaction-và-lock)
  - [5.1. Transaction & Lock cơ bản](#51-transaction--lock-cơ-bản)
  - [5.2. Lock Types trong MySQL InnoDB](#52-lock-types-trong-mysql-innodb)
  - [5.3. Deadlock](#53-deadlock)
- [VI. Nhận diện và xử lý Bottleneck](#vi-nhận-diện-và-xử-lý-bottleneck)
- [VII. Log và Phân tích lỗi](#vii-log-và-phân-tích-lỗi)
- [VIII. Bảo mật Database Server](#viii-bảo-mật-database-server)
- [IX. High Availability và Scalability](#ix-high-availability-và-scalability)
  - [9.1. Tổng quan — Tại sao cần Replication?](#91-tổng-quan--tại-sao-cần-replication)
  - [9.2. MySQL Replication](#92-mysql-replication)
  - [9.3. SQL Server Replication](#93-sql-server-replication)
  - [9.4. MongoDB Replication — Replica Set](#94-mongodb-replication--replica-set)
  - [9.5. So sánh Replication giữa 3 hệ CSDL](#95-so-sánh-replication-giữa-3-hệ-csdl)
  - [9.6. Cluster và Failover](#96-cluster-và-failover)
  - [9.7. Sharding](#97-sharding)
  - [9.8. Load Balancing](#98-load-balancing)
- [X. Tích hợp & Kết nối](#x-tích-hợp--kết-nối)
  - [10.1. Giao thức kết nối Database](#101-giao-thức-kết-nối-database)
  - [10.2. ODBC / JDBC và chuẩn kết nối](#102-odbc--jdbc-và-chuẩn-kết-nối)
  - [10.3. API và Web Services cho Database](#103-api-và-web-services-cho-database)
  - [10.4. ETL — Extract, Transform, Load](#104-etl--extract-transform-load)

---

## I. User và Quản lý quyền truy cập

Trách nhiệm cốt lõi của DBA về access control:
- Giảm thiểu thiệt hại khi bị compromise.
- Phân quyền tối thiểu (Principle of Least Privilege).
- Audit được ai làm gì.
- Tách role rõ ràng.
- Không để app có quyền phá DB.
- Chuẩn hóa access để scale team.

**Nguyên tắc tối thượng:** một ứng dụng web chỉ nên có quyền `SELECT, INSERT, UPDATE, DELETE` trên đúng database của nó, tuyệt đối không có quyền `DROP TABLE` hay `SHUTDOWN` server. Các hệ thống thường áp dụng **Role-Based Access Control (RBAC):** tạo các Role (nhóm quyền) như `read_only`, `db_admin`, `app_user` rồi gán Role đó cho User.

**MySQL/MariaDB:**
```sql
-- Tạo user chỉ được truy cập từ một IP cụ thể (ví dụ: web server)
CREATE USER 'app_user'@'192.168.1.100' IDENTIFIED BY 'Iamhieu123!';

-- Gán quyền DML trên một database cụ thể
GRANT SELECT, INSERT, UPDATE, DELETE ON my_app_db.* TO 'app_user'@'192.168.1.100';

FLUSH PRIVILEGES;
```

**MongoDB:** quản lý user theo cơ chế Role dựa trên từng database cụ thể:
```javascript
use sales_db
db.createUser({
  user: "report_user",
  pwd: "SecurePassword123",
  roles: [ { role: "read", db: "sales_db" } ]  // Chỉ được quyền đọc data
})
```

---

## II. Sao lưu và Phục hồi (Backup & Recovery)

| Loại backup | Mô tả |
|---|---|
| **Full Backup** | Sao lưu toàn bộ dữ liệu tại một thời điểm — an toàn nhất, tốn dung lượng/thời gian nhất. |
| **Differential/Incremental Backup** | Chỉ sao lưu phần thay đổi kể từ lần Full Backup gần nhất — tiết kiệm không gian. |
| **Point-in-Time Recovery (PITR)** | Dùng file log lưu vết (Binary Log ở MySQL, Oplog ở MongoDB, Transaction Log ở MSSQL) để phục hồi database về chính xác một mốc thời gian cụ thể. |

**MySQL** (dùng `mysqldump`):
```bash
# Sao lưu toàn bộ database sales_db ra file script SQL
mysqldump -u root -p sales_db > /backup/sales_db_backup.sql

# Phục hồi dữ liệu từ file backup vào một database trống
mysql -u root -p new_sales_db < /backup/sales_db_backup.sql
```

**Redis** (lưu trên RAM nhưng có 2 cơ chế ghi xuống đĩa: RDB — chụp snapshot, và AOF — ghi log từng lệnh):
```bash
# Ép Redis tạo ngay một file snapshot rdb (chạy ngầm background, không nghẽn hệ thống)
redis-cli BGSAVE
```

---

## III. Giám sát (Monitoring)

Để hệ thống luôn "khỏe mạnh", cần giám sát đồng thời cả tài nguyên phần cứng lẫn chỉ số nội tại của Database:
- **System Metrics:** CPU, RAM, Network throughput, và quan trọng nhất là **Disk IOPS**.
- **Database Metrics:** số connection đang mở, số Slow Queries, Buffer Pool/Cache Hit Ratio (tỷ lệ dữ liệu tìm thấy trên RAM — thấp nghĩa là DB đang phải quét ổ cứng quá nhiều).
- **Xu hướng vận hành:** thường cấu hình các Exporter (như `mysqld_exporter`) đẩy dữ liệu về Prometheus và trực quan hóa qua Grafana.

**Vì sao cần giám sát chủ động:**
```
Không có giám sát:  Sự cố xảy ra → Người dùng báo → DBA mới biết
                    Response time: 30-60 phút

Có giám sát:        Metric vượt ngưỡng → Alert → DBA biết trước khi user thấy
                    Response time: 1-5 phút
```

### 3.1. Theo dõi hiệu năng nhanh theo từng DB

**MongoDB** (xem hiệu năng real-time nhanh qua CLI):
```bash
# Hiển thị số lượng lệnh đọc/ghi, số connection và lượng RAM tiêu thụ mỗi giây
mongostat --rowcount 5
```

**MSSQL** (truy vấn Dynamic Management Views — DMVs):
```sql
-- Tìm top 5 câu lệnh đang chiếm dụng nhiều CPU nhất trong bộ nhớ đệm
SELECT TOP 5 text, total_worker_time/execution_count AS [Avg CPU Time]
FROM sys.dm_exec_query_stats
CROSS APPLY sys.dm_exec_sql_text(plan_handle)
ORDER BY total_worker_time DESC;
```

### 3.2. Bảng so sánh công cụ giám sát

| Công cụ | Loại | Hỗ trợ DB | Điểm mạnh | Hạn chế |
|---|---|---|---|---|
| **PMM** (Percona) | Full-stack | MySQL, PostgreSQL, MongoDB | Query Analytics, đầy đủ | Nặng hơn, cần server riêng |
| **Prometheus + mysqld_exporter** | Metrics | MySQL | Nhẹ, tích hợp Grafana | Chỉ metrics, không query analytics |
| **MySQL Workbench** | GUI | MySQL | Visual, dễ dùng | Không real-time alert |
| **pgAdmin** | GUI | PostgreSQL | Đầy đủ tính năng PG | Chỉ cho PostgreSQL |
| **Datadog** | Cloud SaaS | Đa DB | APM tích hợp, ít setup | Đắt tiền |
| **New Relic** | Cloud SaaS | Đa DB | End-to-end visibility | Đắt tiền |

### 3.3. Metric quan trọng — MySQL

| Metric | Ý nghĩa | Ngưỡng cảnh báo | Lệnh kiểm tra |
|---|---|---|---|
| **Threads_connected** | Số kết nối đang mở | > 80% max_connections | `SHOW STATUS LIKE 'Threads_connected'` |
| **Innodb_buffer_pool_hit_rate** | % đọc từ RAM | < 95% | `SHOW STATUS LIKE 'Innodb_buffer_pool%'` |
| **Slow_queries** | Query chạy chậm | Tăng đột biến | `SHOW STATUS LIKE 'Slow_queries'` |
| **Questions** | Tổng query/giây | Baseline × 3 | `SHOW STATUS LIKE 'Questions'` |
| **Seconds_Behind_Source** | Replica lag | > 30 giây | `SHOW REPLICA STATUS\G` |
| **Innodb_row_lock_waits** | Số lần chờ lock | Tăng liên tục | `SHOW STATUS LIKE 'Innodb_row_lock%'` |

### 3.4. Cài Prometheus + mysqld_exporter + Grafana

```bash
# Cài mysqld_exporter trên Ubuntu 22.04
wget https://github.com/prometheus/mysqld_exporter/releases/download/v0.15.1/mysqld_exporter-0.15.1.linux-amd64.tar.gz
tar xf mysqld_exporter-*.tar.gz
sudo mv mysqld_exporter-*/mysqld_exporter /usr/local/bin/

# Tạo MySQL user riêng cho exporter (quyền tối thiểu — chỉ đọc metric)
# CREATE USER 'exporter'@'localhost' IDENTIFIED BY 'Exp@2026!';
# GRANT PROCESS, REPLICATION CLIENT, SELECT ON *.* TO 'exporter'@'localhost';

mysqld_exporter --config.my-cnf=/etc/.mysqld_exporter.cnf &
# Metrics tại: http://localhost:9104/metrics
```

**Grafana Dashboard ID hữu ích cho MySQL:**
```
7362  → MySQL Overview (mysqld_exporter)
14057 → MySQL InnoDB Metrics
17320 → MySQL Replication
763   → MySQL Overview (PMM-style)
```

---

## IV. Tối ưu hóa truy vấn (Query Optimization)

Khi bảng dữ liệu tăng từ vài nghìn lên vài triệu dòng, một câu lệnh tìm kiếm không tối ưu sẽ bắt Database thực hiện **Full Table Scan** (quét toàn bộ), đẩy CPU lên 100%.

- **Index:** giống mục lục sách, giúp tìm dữ liệu mà không cần lật từng trang — Database thường dùng cấu trúc **B-Tree** để phân nhánh tìm kiếm cực nhanh.
- **Execution Plan:** kế hoạch thực thi mà Database tự tính toán để lấy dữ liệu bằng con đường ngắn nhất.

**MySQL/MariaDB** (dùng `EXPLAIN`):
```sql
-- Kiểm tra xem câu lệnh đang chạy như thế nào
EXPLAIN SELECT * FROM customers WHERE email = 'hieu@example.com';
```
Nếu cột `type` trả về `ALL` nghĩa là đang quét toàn bộ bảng (rất tệ) → tạo Index:
```sql
CREATE INDEX idx_customers_email ON customers(email);
```
Chạy lại `EXPLAIN`, `type` sẽ chuyển thành `const` hoặc `ref` và số dòng cần quét (`rows`) giảm còn 1 dòng.

---

## V. Quản lý Transaction và Lock

### 5.1. Transaction & Lock cơ bản

**Transaction:** đảm bảo tính **ACID**. Điển hình là nghiệp vụ chuyển tiền ngân hàng: (1) Trừ tiền tài khoản A → (2) Cộng tiền tài khoản B. Hai hành động phải cùng thành công hoặc cùng thất bại và quay về trạng thái cũ — không có trạng thái lấp lửng ở giữa.

**Lock:** để tránh 2 user cùng sửa một dòng dữ liệu tại một thời điểm gây sai lệch:
- **Shared Lock:** cho phép nhiều người cùng đọc, nhưng không ai được sửa.
- **Exclusive Lock:** chỉ một người được giữ quyền sửa, block toàn bộ người khác.

**MySQL** (dùng InnoDB để quản lý Transaction):
```sql
START TRANSACTION;
-- Trừ tiền tài khoản A (dòng này bị khóa, session khác phải đợi)
UPDATE accounts SET balance = balance - 500 WHERE account_id = 'A';
-- Cộng tiền tài khoản B
UPDATE accounts SET balance = balance + 500 WHERE account_id = 'B';
-- Nếu cả 2 lệnh chạy mượt, lưu vĩnh viễn và nhả khóa
COMMIT;
-- Nếu lỗi hệ thống, khôi phục lại như chưa có gì xảy ra
-- ROLLBACK;
```

### 5.2. Lock Types trong MySQL InnoDB

| Lock Type | Mô tả | Gây bởi | Ảnh hưởng |
|---|---|---|---|
| **Row Lock** | Khóa 1 row cụ thể | UPDATE, DELETE, SELECT FOR UPDATE | Chỉ row đó |
| **Gap Lock** | Khóa khoảng trống giữa các rows | Range queries trong REPEATABLE READ | Row mới insert vào khoảng này |
| **Next-Key Lock** | Row Lock + Gap Lock | Mặc định của InnoDB | Row + khoảng phía trước |
| **Table Lock** | Khóa toàn bảng | ALTER TABLE, LOCK TABLES | Toàn bảng |
| **Intention Lock** | Báo ý định lock | Trước khi lock row | Tương tác với table lock |

### 5.3. Deadlock

**Deadlock** là hiện tượng tiến trình X giữ tài nguyên 1 và đợi tài nguyên 2, trong khi tiến trình Y giữ tài nguyên 2 và đợi tài nguyên 1 — cả hai đứng đợi nhau vô thời hạn:

```
Transaction A                    Transaction B
─────────────                    ─────────────
LOCK row 1 (A giữ)               LOCK row 2 (B giữ)
      │                                │
      ▼                                ▼
CHỜ lock row 2 ──────────────── CHỜ lock row 1
      (B đang giữ)                     (A đang giữ)

→ Vòng lặp chờ nhau → DEADLOCK!
MySQL tự phát hiện và kill transaction có ít work hơn
```

**Xử lý Deadlock:**
```sql
-- Xem deadlock gần nhất
SHOW ENGINE INNODB STATUS\G
-- Tìm phần "LATEST DETECTED DEADLOCK"

-- Xem transaction đang chạy
SELECT * FROM information_schema.INNODB_TRX ORDER BY trx_started;

-- Kill transaction gây deadlock
KILL 1234;  -- thread_id lấy từ SHOW PROCESSLIST
```

**Phòng tránh Deadlock — 4 nguyên tắc:**
1. **Lock theo cùng thứ tự:** lock A rồi B theo thứ tự cố định.
2. **Transaction ngắn nhất có thể:** BEGIN → UPDATE → COMMIT ngay lập tức, xử lý xong là commit luôn.
3. **Dùng `SELECT ... FOR UPDATE` đủ dùng:** `SELECT id FROM orders WHERE id=1 FOR UPDATE` (lock đúng row, không lock thừa).
4. **Retry logic trong app:**
```python
try:
    execute_transaction()
except DeadlockError:
    sleep(random * 0.1)
    execute_transaction()  # Retry
```

---

## VI. Nhận diện và xử lý Bottleneck

### Mô hình USE — Phương pháp debug chuẩn

```
USE Method cho mỗi tài nguyên:
  U = Utilization  (đang dùng bao nhiêu %)
  S = Saturation   (đang chờ queue không?)
  E = Errors       (có lỗi không?)

Áp dụng theo tầng:
  CPU → RAM → Disk I/O → Network → DB Lock
```

### Bảng nhận diện Bottleneck theo triệu chứng

| Triệu chứng | Tầng nghi ngờ | Lệnh kiểm tra | Hành động |
|---|---|---|---|
| Query chậm, CPU cao | Query/Index | `SHOW PROCESSLIST` | EXPLAIN query, thêm index |
| Query chậm, CPU thấp | Disk I/O | `iostat -x 1 5` | Tăng buffer pool, dùng SSD |
| Nhiều kết nối timeout | Connection pool | `SHOW STATUS LIKE 'Threads%'` | Tăng max_connections, dùng ProxySQL |
| Replication lag tăng | Network/Slow query | `SHOW REPLICA STATUS\G` | Tối ưu query, tăng slave_threads |
| OOM Killer giết MySQL | RAM | `free -h`, `dmesg \| grep oom` | Tăng RAM, giảm buffer_pool_size |

### Công cụ debug nhanh

```bash
# 1. Xem query đang chạy (real-time)
sudo mysql -u root -p -e "SHOW FULL PROCESSLIST\G" | grep -v "Sleep"

# 2. Query chậm nhất hiện tại
sudo mysql -u root -p << 'SQL'
SELECT query, exec_count,
       ROUND(avg_latency/1000000000, 3) AS avg_sec,
       ROUND(total_latency/1000000000, 3) AS total_sec
FROM sys.statement_analysis
ORDER BY avg_latency DESC LIMIT 10;
SQL

# 3. Table nào đang bị lock
sudo mysql -u root -p -e "
SELECT r.trx_id waiting_trx,
       r.trx_mysql_thread_id waiting_thread,
       b.trx_id blocking_trx,
       b.trx_mysql_thread_id blocking_thread,
       p.info blocking_query
FROM information_schema.innodb_lock_waits w
JOIN information_schema.innodb_trx b ON b.trx_id = w.blocking_trx_id
JOIN information_schema.innodb_trx r ON r.trx_id = w.requesting_trx_id
JOIN information_schema.processlist p ON p.id = b.trx_mysql_thread_id;"

# 4. Buffer Pool Hit Rate (phải > 95%)
sudo mysql -u root -p -e "
SELECT ROUND(
  (1 - Innodb_buffer_pool_reads / Innodb_buffer_pool_read_requests) * 100, 2
) AS buffer_pool_hit_rate
FROM (SELECT
  (SELECT VARIABLE_VALUE FROM information_schema.GLOBAL_STATUS WHERE VARIABLE_NAME='Innodb_buffer_pool_reads') AS Innodb_buffer_pool_reads,
  (SELECT VARIABLE_VALUE FROM information_schema.GLOBAL_STATUS WHERE VARIABLE_NAME='Innodb_buffer_pool_read_requests') AS Innodb_buffer_pool_read_requests
) t;"

# 5. Disk I/O
iostat -x 1 5
# %util > 80%  → Disk là bottleneck
# await > 20ms → Disk latency cao
```

### Slow Query Log — Công cụ số 1 tối ưu

```ini
# Bật trong /etc/mysql/mysql.conf.d/mysqld.cnf
slow_query_log      = 1
slow_query_log_file = /var/log/mysql/mysql-slow.log
long_query_time     = 1           # Log query chạy > 1 giây
log_queries_not_using_indexes = 1 # Log query không dùng index
```

```bash
# Phân tích slow query log
sudo mysqldumpslow -s t -t 10 /var/log/mysql/mysql-slow.log
# -s t  = sắp xếp theo tổng thời gian
# -t 10 = top 10 query

# Hoặc dùng pt-query-digest (Percona Toolkit)
sudo pt-query-digest /var/log/mysql/mysql-slow.log | head -100
```

---

## VII. Log và Phân tích lỗi

### Các loại Log trong MySQL

| Log | Vị trí | Ghi gì | Dùng khi |
|---|---|---|---|
| **Error Log** | `/var/log/mysql/error.log` | Startup, shutdown, critical errors | Luôn bật, debug sự cố |
| **General Query Log** | Tắt theo mặc định | Mọi query đến server | Debug ngắn hạn (tốn I/O) |
| **Slow Query Log** | `/var/log/mysql/mysql-slow.log` | Query chậm hơn `long_query_time` | Tối ưu performance |
| **Binary Log** | `/var/log/mysql/mysql-bin.*` | Mọi thay đổi data (INSERT/UPDATE/DELETE) | Replication, PITR backup |
| **Relay Log** | Replica only | Binlog từ Primary | Replication |
| **InnoDB Undo Log** | Trong data directory | Snapshot cũ cho MVCC | Internal, không đọc trực tiếp |

### Quy trình debug sự cố chuẩn

```bash
# BƯỚC 1: Xem Error Log (luôn bắt đầu từ đây)
sudo tail -100 /var/log/mysql/error.log | grep -E "ERROR|Warning|FATAL"

# BƯỚC 2: Xem MySQL status
sudo mysql -u root -p -e "SHOW GLOBAL STATUS;" | grep -E "Error|Abort|Timeout"

# BƯỚC 3: Xem process đang chạy
sudo mysql -u root -p -e "SHOW FULL PROCESSLIST\G" | grep -v "Sleep"

# BƯỚC 4: Xem lock và transaction
sudo mysql -u root -p -e "SELECT * FROM sys.innodb_lock_waits\G"

# BƯỚC 5: Xem InnoDB engine status
sudo mysql -u root -p -e "SHOW ENGINE INNODB STATUS\G" 2>/dev/null \
  | grep -A 50 "TRANSACTIONS"
```

### Phân tích Binary Log — Tìm ai xóa data

```bash
# Ví dụ: Ai đã xóa table lúc 10:30 sáng nay?
sudo mysqlbinlog \
  --start-datetime="2026-05-30 10:25:00" \
  --stop-datetime="2026-05-30 10:35:00" \
  --base64-output=DECODE-ROWS \
  -v /var/log/mysql/mysql-bin.000023 \
  | grep -E "DELETE|DROP|user@|timestamp"
```

### Log Rotation — Tự động để không đầy disk

```bash
# /etc/logrotate.d/mysql-server
/var/log/mysql/*.log {
    daily
    rotate 7            # Giữ 7 ngày
    compress            # Nén log cũ
    delaycompress
    missingok
    notifempty
    create 640 mysql adm
    postrotate
        MYADMIN="/usr/bin/mysqladmin --defaults-file=/etc/mysql/debian.cnf"
        if [ -f `$MYADMIN variables 2>/dev/null | awk '{ if ($2 == "pid_file") print $4}'` ]; then
            $MYADMIN flush-logs
        fi
    endscript
}
```

---

## VIII. Bảo mật Database Server

### 1. Các mối đe dọa bảo mật phổ biến

Trong môi trường database, các cuộc tấn công thường nhắm vào 4 con đường chính:

- **SQL/NoSQL Injection:** lỗ hổng kinh điển nhưng vẫn cực kỳ nguy hiểm — xảy ra khi ứng dụng không kiểm tra kỹ dữ liệu nhập vào từ người dùng, cho phép kẻ tấn công "chèn" câu lệnh database độc hại, bypass đăng nhập, lấy toàn bộ dữ liệu hoặc xóa sạch database.
- **Phơi nhiễm do cấu hình sai:** thường do quản trị viên chủ quan/thiếu kinh nghiệm — để database mở cổng ra Internet công cộng (`0.0.0.0/0`) mà không đổi port mặc định, không đặt password cho Admin, hoặc để lộ file backup công khai trên cloud storage.
- **Brute Force và Credential Stuffing:** dùng công cụ tự động dò quét password liên tục, hoặc dùng danh sách tài khoản/mật khẩu rò rỉ từ hệ thống khác để thử đăng nhập.
- **Mối đe dọa từ bên trong:** nhân viên cũ, quản trị viên có quá nhiều quyền vô tình/cố ý đánh cắp dữ liệu, hoặc tài khoản bị hacker chiếm quyền.

### 2. Mã hóa dữ liệu (Data Encryption)

Cần thực hiện mã hóa ở 2 trạng thái:

**Mã hóa trên đường truyền (Encryption in Transit):**
- Bảo vệ dữ liệu khi di chuyển giữa Application Server và Database Server qua mạng.
- Cơ chế: TLS/SSL thiết lập đường ống mã hóa an toàn — toàn bộ câu lệnh SQL gửi đi và kết quả trả về đều được mã hóa.
- Mục đích: ngăn chặn nghe lén, đánh cắp gói tin trên đường truyền (Man-in-the-Middle Attack).

**Mã hóa khi lưu trữ (Encryption at Rest):**
- Bảo vệ dữ liệu khi nằm yên trên ổ cứng, file backup, file log hệ thống.
- **Transparent Data Encryption (TDE):** mã hóa ở cấp database (phổ biến ở MSSQL, MySQL Enterprise, Oracle) — tự động mã hóa file dữ liệu (`.mdf`, `.ibd`) trước khi ghi xuống đĩa, giải mã khi nạp lên RAM, hoàn toàn "trong suốt" với ứng dụng.
- **Application-level Encryption:** mã hóa trực tiếp từ code ứng dụng trước khi insert vào database — database chỉ thấy chuỗi ký tự vô nghĩa (dùng cho trường cực nhạy cảm như số thẻ tín dụng, mật khẩu).
- Mục đích: dù kẻ trộm lấy được ổ cứng vật lý hoặc file backup `.sql`/`.bak`, chúng cũng không đọc được nội dung nếu không có khóa giải mã (Master Key).

### 3. Audit và theo dõi truy cập (Database Auditing)

Audit Log là hệ thống nhật ký độc lập, ghi lại chi tiết hành vi bảo mật trên Database Server — trả lời câu hỏi: **Ai** (user, IP) đã làm gì (SELECT, DROP, GRANT...), vào lúc nào, thành công hay thất bại.

| Loại log | Mục đích |
|---|---|
| Error Log | Chỉ ghi lỗi hệ thống để debug |
| Transaction Log | Phục hồi dữ liệu khi crash |
| **Audit Log** | Chỉ tập trung vào sự kiện an ninh thông tin |

**Giá trị cốt lõi:**
- **Phát hiện bất thường:** VD tài khoản kế toán bình thường đột nhiên SELECT hàng triệu dòng dữ liệu khách hàng lúc 2h sáng → cảnh báo ngay.
- **Điều tra số:** khi xảy ra rò rỉ dữ liệu, Audit log là bằng chứng duy nhất tìm nguyên nhân/thủ phạm.
- **Tuân thủ:** PCI-DSS (tài chính/thẻ), GDPR (bảo vệ dữ liệu châu Âu), ISO 27001 đều bắt buộc bật Database Auditing.

### 4. Patch Management và Cập nhật bảo mật

**Patch Management** là quy trình chiến lược nhằm kiểm tra, cài đặt bản vá bảo mật và cập nhật phiên bản cho cả OS lẫn Database Engine.

**Quy trình chuẩn trong doanh nghiệp:**
1. **Theo dõi & Đánh giá:** nhận thông tin lỗ hổng mới từ nhà phát hành, đánh giá mức độ ảnh hưởng (Nghiêm trọng/Cao/Trung bình).
2. **Thử nghiệm:** tuyệt đối không áp bản vá trực tiếp lên Production — phải chạy thử trên Staging/UAT trước.
3. **Sao lưu dự phòng:** thực hiện Full Backup toàn diện ngay trước khi update Production để có đường lùi.
4. **Triển khai trong Maintenance Window:** chọn khung giờ bảo trì để giảm ảnh hưởng người dùng.

---

## IX. High Availability và Scalability

### 9.1. Tổng quan — Tại sao cần Replication?

**Replication** là quá trình sao chép tự động mọi thay đổi dữ liệu từ server chính (Primary/Master) sang một hoặc nhiều server phụ (Replica/Slave) theo thời gian thực.

```
Hệ thống KHÔNG có Replication:
┌──────────────┐        ┌──────────────┐
│   APP SERVER │───────►│  DB SERVER   │  ← SPOF!
└──────────────┘        └──────────────┘
                         Server này chết → toàn bộ hệ thống chết

Hệ thống CÓ Replication:
┌──────────────┐  Write  ┌──────────────┐
│   APP SERVER │────────►│   PRIMARY    │──── sync ────►┌──────────┐
│              │  Read   └──────────────┘               │ REPLICA 1│
│              │─────────────────────────────────────►  └──────────┘
└──────────────┘                                         ┌──────────┐
                                                         │ REPLICA 2│
                                                         └──────────┘
Primary chết → Promote Replica → Hệ thống tiếp tục chạy
```

**Ba mục tiêu chính của Replication:**

| Mục tiêu | Giải thích | Ví dụ thực tế |
|---|---|---|
| **High Availability** | Hệ thống tiếp tục hoạt động khi 1 node chết | Promote Replica khi Primary crash |
| **Read Scalability** | Phân tải query SELECT sang nhiều Replica | App đọc từ 3 Replica, ghi vào 1 Primary |
| **Data Protection** | Có bản sao dữ liệu ở nhiều máy | Replica ở datacenter khác → Disaster Recovery |

### 9.2. MySQL Replication

> **Master/Slave (Source/Replica):** ba chế độ ghi log — Statement-based (log câu SQL gốc, log nhỏ nhưng hàm như `NOW()`/`RAND()` không nhất quán), Row-based (log từng row thay đổi, chính xác tuyệt đối nhưng log lớn hơn), Mixed (kết hợp tự động).

**Kiến trúc tổng quan:**
```
                    ┌─────────────────────────────┐
   App ghi ──────►  │        PRIMARY (Master)      │
                    │  - Nhận INSERT/UPDATE/DELETE │
                    │  - Ghi vào Binary Log (binlog)│
                    └────────────┬────────────────-┘
                                 │  Binary Log Events
                    ┌────────────▼──────────────────────────────┐
                    │           REPLICATION CHANNEL              │
                    │  IO Thread kéo binlog về Relay Log        │
                    │  SQL Thread đọc Relay Log → apply vào DB  │
                    └────────────┬──────────────────────────────┘
                                 │
              ┌──────────────────┼──────────────────┐
              ▼                  ▼                   ▼
      ┌──────────────┐  ┌──────────────┐   ┌──────────────┐
      │  REPLICA 1   │  │  REPLICA 2   │   │  REPLICA 3   │
      │  (Read only) │  │  (Read only) │   │  (Reporting) │
      └──────────────┘  └──────────────┘   └──────────────┘
              ▲                  ▲                   ▲
   App đọc ───┴──────────────────┘                   │
   (SELECT)                                     BI/Analytics
```

**Cơ chế hoạt động chi tiết (3 luồng):**
```
PHÍA PRIMARY:
1. Client thực thi INSERT INTO orders...
2. Storage Engine ghi data vào disk
3. Binlog Thread ghi event vào Binary Log (nhật ký mọi thay đổi)

PHÍA REPLICA — 2 thread song song:
IO Thread:   kết nối lên Primary qua TCP, liên tục kéo Binary Log events mới về,
             ghi vào Relay Log (file local của Replica)
SQL Thread:  đọc events từ Relay Log, thực thi lại các câu SQL đó trên Replica
             Replica lag = độ trễ giữa Primary và Replica
```

**Các lệnh quản lý thực tế:**
```sql
-- === TRÊN PRIMARY ===
-- Tạo user dành riêng cho replication (không dùng root!)
CREATE USER 'repl_user'@'%' IDENTIFIED BY 'StrongRepl@2026';
GRANT REPLICATION SLAVE ON *.* TO 'repl_user'@'%';
FLUSH PRIVILEGES;

-- Xem trạng thái binlog (lấy File và Position cho Replica)
SHOW BINARY LOG STATUS\G
-- *** File: mysql-bin.000003
-- *** Position: 157

-- === TRÊN REPLICA ===
CHANGE REPLICATION SOURCE TO
  SOURCE_HOST='192.168.1.10', SOURCE_PORT=3306,
  SOURCE_USER='repl_user', SOURCE_PASSWORD='StrongRepl@2026',
  SOURCE_LOG_FILE='mysql-bin.000003', SOURCE_LOG_POS=157;

START REPLICA;
SHOW REPLICA STATUS\G

-- Kiểm tra và thiết lập binlog format
SHOW VARIABLES LIKE 'binlog_format';
SET GLOBAL binlog_format = 'ROW';  -- Khuyến nghị 2026
```

**Đọc kết quả `SHOW REPLICA STATUS` — những dòng quan trọng nhất:**
```
Replica_IO_Running: Yes          ← IO Thread đang chạy, kết nối Primary OK
Replica_SQL_Running: Yes         ← SQL Thread đang chạy, apply events OK
Seconds_Behind_Source: 0         ← Lag = 0 giây → Replica đồng bộ hoàn toàn
Last_Error:                      ← Trống = không có lỗi

⚠️ Dấu hiệu có vấn đề:
Replica_IO_Running: No           → Mất kết nối với Primary (network, auth)
Replica_SQL_Running: No          → Lỗi khi apply SQL (thường do data conflict)
Seconds_Behind_Source: 3600      → Replica đang lag 1 tiếng — quá tải hoặc slow query
```

### 9.3. SQL Server Replication

SQL Server có **4 loại Replication** — điểm khác biệt lớn so với MySQL/MongoDB:

```
1. SNAPSHOT REPLICATION
   Chụp toàn bộ data tại một thời điểm → gửi sang Subscriber
   Dùng khi: data ít thay đổi, đồng bộ theo lịch
   Ví dụ: Catalog sản phẩm, danh mục, báo cáo cuối ngày

2. TRANSACTIONAL REPLICATION (Phổ biến nhất)
   Gần giống MySQL binlog replication
   Mỗi transaction được ghi vào Distribution DB → gửi đi
   Dùng khi: cần real-time, low latency
   Ví dụ: OLTP → Read Replica cho reporting

3. MERGE REPLICATION
   Cả Publisher và Subscriber đều có thể ghi
   Có cơ chế giải quyết conflict tự động
   Dùng khi: app offline, sync lại khi có mạng
   Ví dụ: App bán hàng trên tablet, đồng bộ về trung tâm

4. ALWAYS ON AVAILABILITY GROUPS (Enterprise — Hiện đại)
   Không phải "Replication" truyền thống
   Dùng log shipping + sync/async commit
   Tự động failover, readable secondaries
   Dùng khi: Production enterprise, HA yêu cầu cao
```

```sql
-- Kiểm tra trạng thái Availability Group
SELECT ag.name AS AGName,
       ar.replica_server_name,
       ar.availability_mode_desc,
       ars.role_desc,
       ars.synchronization_state_desc
FROM sys.availability_groups ag
JOIN sys.availability_replicas ar ON ag.group_id = ar.group_id
JOIN sys.dm_hadr_availability_replica_states ars ON ar.replica_id = ars.replica_id;

-- Xem lag của Secondary
SELECT database_name, log_send_queue_size, redo_queue_size, log_send_rate, redo_rate
FROM sys.dm_hadr_database_replica_states;
```

### 9.4. MongoDB Replication — Replica Set

> MongoDB không gọi là Master-Slave mà là **Replica Set** — nhóm các instance giữ cùng một bộ dữ liệu, có cơ chế **bầu cử tự động (election)** khi Primary gặp sự cố.

**Kiến trúc Replica Set:**
```
┌─────────────────────────────────────────────────────────────┐
│                    REPLICA SET (3 nodes)                     │
│   ┌──────────────┐    oplog sync    ┌──────────────────┐   │
│   │   PRIMARY    │ ───────────────► │   SECONDARY 1    │   │
│   │ Reads+Writes │ ◄─── Heartbeat ──│  Read (optional) │   │
│   │ oplog source │  (every 2 sec)   │  Votes in elect. │   │
│   └──────┬───────┘                  └──────────────────┘   │
│          │ oplog sync    ┌──────────────────┐               │
│          └─────────────► │   SECONDARY 2    │               │
│                          │  (or ARBITER)    │               │
│                          │  Votes in elect. │               │
│                          └──────────────────┘               │
└─────────────────────────────────────────────────────────────┘
ARBITER: Node đặc biệt — chỉ tham gia bầu cử, không lưu data
         Dùng khi muốn có số node lẻ mà không muốn tốn tài nguyên
```

```javascript
// Khởi tạo Replica Set
rs.initiate({
  _id: "rs0",
  members: [
    { _id: 0, host: "mongo1:27017", priority: 2 },  // Ưu tiên làm Primary
    { _id: 1, host: "mongo2:27017", priority: 1 },
    { _id: 2, host: "mongo3:27017", priority: 0, hidden: true, votes: 1 }
  ]
})

rs.status()
rs.printReplicationInfo()            // Xem oplog size và lag
rs.printSecondaryReplicationInfo()   // Xem lag từng Secondary

// Cấu hình ReadPreference — đọc từ Secondary
db.getMongo().setReadPref("secondaryPreferred")
// "primary" | "primaryPreferred" | "secondary" | "secondaryPreferred" | "nearest"
```

**Cơ chế Election — điểm khác biệt lớn nhất so với MySQL:**
```
Kịch bản: Primary bị crash → sau 10 giây không nhận heartbeat →

1. Secondary nhận ra Primary mất
2. Secondary tự đề cử (candidate)
3. Gửi RequestVote đến các node khác
4. Node có oplog mới nhất + priority cao nhất thắng
5. Cần đa số phiếu (majority): 3 nodes → cần 2
6. Winner trở thành PRIMARY mới
7. Toàn bộ quá trình: ~10-30 giây

✅ TỰ ĐỘNG — không cần can thiệp thủ công
❌ MySQL cần DBA can thiệp failover thủ công
```

**Oplog — tương đương Binary Log của MySQL:**
```javascript
use local
db.oplog.rs.find().sort({$natural:-1}).limit(3).pretty()

// Kết quả mẫu:
{
  "ts": Timestamp(1706000001, 1),   // Thời điểm operation
  "op": "i",                         // i=insert, u=update, d=delete
  "ns": "ecommerce.orders",          // namespace: db.collection
  "o": { "_id": ObjectId("..."), "total": 500000 }
}
// Secondary liên tục đọc oplog của Primary và apply vào local
```

### 9.5. So sánh Replication giữa 3 hệ CSDL

| Tiêu chí | MySQL 8.4 | SQL Server 2022 | MongoDB 7 |
|---|---|---|---|
| **Log mechanism** | Binary Log | Transaction Log | Oplog |
| **Failover** | Thủ công / Orchestrator | WSFC tự động | Election tự động |
| **Readable Secondaries** | Cần cấu hình | ✅ Always On | ✅ ReadPreference |
| **Số Replica** | Không giới hạn | Up to 8 secondaries | Tối đa 50 members |
| **Sync modes** | Async / Semi-sync | Sync / Async | Tunable WriteConcern |
| **Multi-master** | Group Replication | Không | Không |
| **Tự phát hiện topology** | Không | Qua Listener | ✅ Tự động |
| **Độ phức tạp cấu hình** | Trung bình | Cao | Thấp |

**So sánh chi tiết hơn — MySQL Replication vs MongoDB Replica Set:**

| Tiêu chí | MySQL Replication | MongoDB Replica Set |
|---|---|---|
| **Failover** | Thủ công (hoặc Orchestrator/MHA) | ✅ Tự động (Election ~10-30s) |
| **Log mechanism** | Binary Log (binlog) | Oplog (capped collection) |
| **Số replica tối đa** | Không giới hạn | 50 members, 7 có quyền vote |
| **Đọc từ Replica** | Cần cấu hình app | Cấu hình ReadPreference |
| **Consistency đọc** | Eventual (mặc định) | Tunable (local/majority/linearizable) |
| **Monitoring** | `SHOW REPLICA STATUS` | `rs.status()` |

### 9.6. Cluster và Failover

**Cluster khác Replication ở điểm nào?**
```
Replication: 1 node làm việc, các node còn lại chờ (Hot Standby)
             Chỉ Primary nhận ghi → Bottleneck vẫn ở Primary

Cluster:     NHIỀU node cùng hoạt động đồng thời
             Ghi có thể vào bất kỳ node nào
             Hệ thống đồng bộ với nhau qua consensus protocol
```

**MySQL InnoDB Cluster:**
- Tất cả node chạy Group Replication.
- MySQL Router biết ai là Primary → route write vào đó.
- Primary chết → Election → Router cập nhật tự động.
- RPO ≈ 0 (không mất data), RTO ≈ vài giây.

```sql
-- Kiểm tra trạng thái Group Replication
SELECT MEMBER_ID, MEMBER_HOST, MEMBER_STATE, MEMBER_ROLE
FROM performance_schema.replication_group_members;

-- Xem ai đang là Primary
SELECT VARIABLE_VALUE FROM performance_schema.global_status
WHERE VARIABLE_NAME = 'group_replication_primary_member';
```

**Galera Cluster:** app có thể ghi vào **bất kỳ node nào**, Galera đảm bảo mọi node có data giống nhau; 1 node chết → các node còn lại tiếp tục hoạt động.

```ini
# my.cnf cho Galera Cluster
[mysqld]
wsrep_on=ON
wsrep_provider=/usr/lib/galera/libgalera_smm.so
wsrep_cluster_name="ecommerce_cluster"
wsrep_cluster_address="gcomm://192.168.1.10,192.168.1.11,192.168.1.12"
wsrep_node_address="192.168.1.10"
wsrep_node_name="node1"
wsrep_sst_method=rsync
```

```sql
-- Kiểm tra trạng thái Galera
SHOW STATUS LIKE 'wsrep_%';
-- wsrep_cluster_size: 3          → 3 node đang online
-- wsrep_cluster_status: Primary  → cluster healthy
-- wsrep_local_state_comment: Synced → node này đồng bộ
-- wsrep_flow_control_paused: 0   → không bị throttle
```

**So sánh thời gian downtime khi Failover:**
```
MySQL (không có cluster):
  T+0s:   Primary chết
  T+30s:  Monitoring phát hiện
  T+60s:  DBA nhận alert
  T+120s: DBA chạy lệnh promote Replica
  T+180s: App reconfigure connection string
  DOWNTIME: 3-10 phút

MySQL InnoDB Cluster / Galera:
  T+0s:  Node chết
  T+3s:  Heartbeat timeout
  T+5s:  Election hoàn tất, Router cập nhật
  T+5s:  App tiếp tục kết nối (qua MySQL Router)
  DOWNTIME: ~5 giây

SQL Server Always On:
  T+0s:  Primary chết
  T+5s:  WSFC phát hiện
  T+20s: Automatic Failover hoàn tất
  T+20s: Listener IP chuyển sang Secondary mới
  DOWNTIME: ~20-30 giây
```

### 9.7. Sharding

> **Sharding** (Horizontal Partitioning) là kỹ thuật phân chia dữ liệu ra nhiều database server dựa theo một tiêu chí (Sharding Key).

**Dấu hiệu hệ thống cần Sharding:**
- Dataset > RAM của 1 server.
- Write throughput > 10.000 ops/sec trên 1 server.
- Disk I/O đạt giới hạn của 1 máy.
- Query bắt đầu timeout dù đã tối ưu index.

### 9.8. Load Balancing

**Vì sao cần Load Balancing:**
```
KHÔNG có Load Balancing:
App gửi 1000 requests/s → 1 DB server gánh hết → quá tải

CÓ Load Balancing:
App gửi 1000 requests/s → LB phân phối đều → 3 DB servers mỗi cái 333 req/s

Lưu ý quan trọng với Database:
- WRITE: Luôn vào Primary (không được load balance Write)
- READ:  Có thể load balance sang nhiều Replica
```

**Các thuật toán:**

| Thuật toán | Cách hoạt động | Phù hợp khi |
|---|---|---|
| **Round Robin** | Lần lượt: R1→R2→R3→R1→... | Các Replica có sức mạnh tương đương |
| **Weighted Round Robin** | R1 nhận 60%, R2 nhận 40% | Replica có phần cứng khác nhau |
| **Least Connections** | Gửi vào server ít kết nối nhất | Query có thời gian xử lý khác nhau |
| **Least Response Time** | Gửi vào server phản hồi nhanh nhất | Khi muốn tối ưu latency |
| **Random** | Ngẫu nhiên | Đơn giản, ít quan trọng |

**ProxySQL — giải pháp Load Balancing cho MySQL:**
```
          APP (port 3306 thông thường)
                    │
                    ▼
             PROXYSQL :6033
                    │
        ┌───────────┼───────────┐
        │ Query Rule Engine     │
        │ ^SELECT → Readers     │
        │ Else → Writers        │
        └───────────┬───────────┘
              Write │      Read
              ┌─────▼┐   ┌─┴──────────────┐
              │ HG 10│   │     HG 20       │
              │Master│   │Replica1 Replica2│
              └──────┘   └────────────────┘
```

```sql
-- Cấu hình Load Balancing với Weight trong ProxySQL
-- Replica 1 mạnh hơn → weight cao hơn → nhận nhiều request hơn
INSERT INTO mysql_servers (hostgroup_id, hostname, port, weight)
VALUES
  (20, 'replica1', 3306, 1000),  -- Nhận 66% traffic
  (20, 'replica2', 3306, 500);   -- Nhận 34% traffic

-- Xem thống kê traffic phân phối
SELECT srv_host, Queries, Bytes_data_sent
FROM stats_mysql_connection_pool
WHERE hostgroup = 20;
```

**HAProxy — giải pháp đơn giản hơn** (Layer 4/TCP, không hiểu SQL, phù hợp khi chỉ cần route đơn giản):
```
          APP
           │
           ▼
        HAPROXY :3306
           │
    ┌──────┴──────┐
    ▼             ▼
 MySQL 1       MySQL 2
(Primary)     (Replica)
```

```ini
# /etc/haproxy/haproxy.cfg — MySQL Replica load balancing
frontend mysql_read
    bind *:3307
    mode tcp
    default_backend mysql_replicas

backend mysql_replicas
    mode tcp
    balance leastconn
    option mysql-check user haproxy_check
    server replica1 192.168.1.11:3306 weight 100 check
    server replica2 192.168.1.12:3306 weight 100 check
    server replica3 192.168.1.13:3306 weight 50  check  # Server yếu hơn
```

---

## X. Tích hợp & Kết nối

### 10.1. Giao thức kết nối Database

Mỗi database dùng giao thức wire protocol riêng:

| Database | Giao thức | Port |
|---|---|---|
| MySQL | MySQL Protocol | 3306 |
| PostgreSQL | PostgreSQL Wire Protocol | 5432 |
| SQL Server | TDS (Tabular Data Stream) | 1433 |
| MongoDB | MongoDB Wire Protocol | 27017 |
| Redis | RESP (REdis Serialization Protocol) | 6379 |
| Oracle | SQL*Net / TNS | 1521 |

**Bảo mật giao thức — TLS/SSL:**
```
KHÔNG có TLS: Client ── username+password (plaintext) ──► Server
              Attacker có thể nghe lén mọi thứ!

CÓ TLS: Client ── TLS Handshake ──► Server, Certificate xác minh danh tính,
        toàn bộ data truyền đi đều được mã hóa
```

```sql
-- MySQL: bắt buộc TLS cho 1 user
ALTER USER 'appuser'@'%' REQUIRE SSL;

-- Kiểm tra kết nối có dùng SSL không
SHOW STATUS LIKE 'Ssl_cipher';
-- Ssl_cipher: TLS_AES_256_GCM_SHA384 → đang dùng TLS ✅
-- Ssl_cipher: (trống)                → không dùng TLS ⚠️
```

### 10.2. ODBC / JDBC và chuẩn kết nối

**Vấn đề khi không có chuẩn kết nối chung:** mỗi ngôn ngữ (Python, Java, C#) phải dùng driver riêng của từng hãng database → đổi database phải viết lại code kết nối.

**Giải pháp — ODBC (Open Database Connectivity):**
```
Application (Python, Excel, PowerBI, R)
        ↓ ODBC API calls (chuẩn)
ODBC Driver Manager (unixODBC trên Linux)
        ↓ Load đúng driver
MySQL ODBC Driver / PostgreSQL Driver / ...
        ↓ Wire Protocol
Database Server
```

**JDBC (Java Database Connectivity):**
```
Java Application (Spring Boot, Hibernate)
        ↓ java.sql.* / javax.sql.*
JDBC Driver (mysql-connector-j-9.x.jar)
        ↓ MySQL Wire Protocol
MySQL Server
```

**So sánh ODBC vs JDBC vs Native Driver:**

| Tiêu chí | ODBC | JDBC | Native Driver |
|---|---|---|---|
| Ngôn ngữ | C/C++, Python, R, Excel | Java, Kotlin, Scala | Từng ngôn ngữ riêng |
| Cấu hình | DSN trong OS (`odbcad32`) | Connection string trong code | Connection string |
| Hiệu suất | Trung bình (có overhead) | Tốt | Tốt nhất |
| Đa DB | ✅ Rất rộng | ✅ Rộng (JVM) | Chỉ 1 DB |
| Dùng khi | BI tools, báo cáo, Excel | Java app | Cần max performance |

```bash
# Cài ODBC trên Ubuntu 22.04
sudo apt install unixodbc unixodbc-dev -y
odbcinst -d -q       # Xem drivers đã đăng ký
isql -v MySQL_DSN    # Test kết nối DSN
```

### 10.3. API và Web Services cho Database

**REST API — Mapping với SQL:**

| HTTP Method | URL | SQL tương đương |
|---|---|---|
| `GET /orders` | Lấy danh sách | `SELECT * FROM orders` |
| `GET /orders/123` | Lấy 1 record | `SELECT * WHERE id=123` |
| `POST /orders` | Tạo mới | `INSERT INTO orders` |
| `PUT /orders/123` | Cập nhật toàn bộ | `UPDATE ... WHERE id=123` |
| `PATCH /orders/123` | Cập nhật một phần | `UPDATE SET field=val WHERE id=123` |
| `DELETE /orders/123` | Xóa | `DELETE WHERE id=123` |

**GraphQL vs REST — góc nhìn DB:**
```
Vấn đề của REST:
GET /users/1       → {id, name, email, address, phone, ...}  (lấy thừa)
GET /orders?user=1 → [...]                                    (request thứ 2)
GET /products/...  → [...]                                    (request thứ 3)
= 3 requests, nhiều data thừa

GraphQL:
POST /graphql
query {
  user(id: 1) {
    name                    ← chỉ lấy đúng field cần
    orders { id total }     ← join sẵn trong 1 request
  }
}
= 1 request, đúng data cần
```

| Tiêu chí | REST | GraphQL | gRPC |
|---|---|---|---|
| Overfetching | Phổ biến | Không | Không |
| Số request | Nhiều (N+1) | 1 request | Streaming |
| Caching | HTTP cache dễ | Khó hơn | Phức tạp |
| Learning curve | Thấp | Trung bình | Cao |
| Dùng khi | Public API đơn giản | Internal API, mobile | Microservices nội bộ |

**Database Gateway Pattern:**
```
App → API Gateway → Auth/Rate Limit → Database API → DB
                                           ↑
                              (Hasura, PostgREST, Supabase)
                              Tự động generate API từ DB schema
```

### 10.4. ETL — Extract, Transform, Load

**Khái niệm:** ETL là quy trình di chuyển và biến đổi dữ liệu giữa các hệ thống.
```
EXTRACT          TRANSFORM              LOAD
Trích xuất  →   Làm sạch, chuẩn hóa  →  Nạp vào đích
từ nguồn        Join, aggregate           (Data Warehouse,
                Convert, deduplicate      Analytics DB)

Ví dụ thực tế:
MySQL OLTP ──┐
CSV files   ─┼─► ETL Engine ──► BigQuery / Redshift (Data Warehouse)
REST API    ─┘
                 └► BI Tool (Grafana, Tableau, Metabase)
```

**ETL vs ELT:**

| Tiêu chí | ETL (truyền thống) | ELT (hiện đại) |
|---|---|---|
| Thứ tự | Extract → Transform → Load | Extract → Load → Transform |
| Transform ở đâu | Server ETL riêng | Trong Data Warehouse |
| Phù hợp | Data Warehouse cũ, on-premise | Cloud DW |
| Công cụ | Informatica, SSIS, Pentaho | dbt, Airbyte + dbt |
| Xu hướng 2026 | Đang giảm | Đang tăng |

**CDC — Change Data Capture (Real-time ETL):**
```
ETL batch truyền thống: Chạy mỗi đêm → Data lag 24 giờ
CDC: Đọc binlog MySQL → Stream events real-time → lag vài giây

MySQL binlog ──► Debezium ──► Kafka ──► Consumer
                 (đọc như       (event   ├─► Elasticsearch
                  replica)       bus)    ├─► Data Warehouse
                                         └─► Cache invalidation
```

```json
{
  "connector.class": "io.debezium.connector.mysql.MySqlConnector",
  "database.hostname": "192.168.1.10",
  "database.include.list": "ecommerce",
  "table.include.list": "ecommerce.orders"
}
```

**Công cụ ETL phổ biến 2026:**

| Công cụ | Loại | Điểm mạnh | Dùng khi |
|---|---|---|---|
| **Apache Airflow** | Orchestrator | DAG-based, Python, schedule | Cần lên lịch và monitor ETL jobs |
| **dbt** | Transform | SQL-based, version control, test | ELT trong Data Warehouse |
| **Airbyte** | Extract + Load | 300+ connectors, open source | Cần connector nhanh, ít code |
| **Debezium** | CDC | Real-time, đọc binlog | Sync real-time, event streaming |
| **Apache Spark** | Batch + Stream | Xử lý TB data, ML integration | Big Data, phức tạp |
| **Fivetran** | EL managed | Tự động, ít maintain | Team nhỏ, ưu tiên tốc độ setup |
