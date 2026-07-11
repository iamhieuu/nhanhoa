# 04. MongoDB — Cài đặt, CRUD, Schema Design, So sánh RDBMS

## Mục lục

- [I. Tổng quan & thuật ngữ](#i-tổng-quan--thuật-ngữ)
- [II. Cài đặt MongoDB 7.0 trên Ubuntu 22.04](#ii-cài-đặt-mongodb-70-trên-ubuntu-2204)
- [III. Kết nối & thao tác CRUD cơ bản](#iii-kết-nối--thao-tác-crud-cơ-bản)
- [IV. File cấu hình `/etc/mongod.conf`](#iv-file-cấu-hình-etcmongodconf)
- [V. Tối ưu hiệu năng](#v-tối-ưu-hiệu-năng)
  - [5.1. Index](#51-index)
  - [5.2. Aggregation Pipeline](#52-aggregation-pipeline)
  - [5.3. Profiler](#53-profiler)
  - [5.4. Monitoring nhanh trong mongosh](#54-monitoring-nhanh-trong-mongosh)
- [VI. Bảo mật MongoDB](#vi-bảo-mật-mongodb)
  - [6.1. Bật Authentication](#61-bật-authentication)
  - [6.2. Tạo user riêng cho từng app](#62-tạo-user-riêng-cho-từng-app)
  - [6.3. Firewall](#63-firewall)
  - [6.4. TLS/SSL](#64-tlsssl)
  - [6.5. Disable HTTP Interface & kiểm tra cấu hình](#65-disable-http-interface--kiểm-tra-cấu-hình)
---

## I. Tổng quan & thuật ngữ

MongoDB lưu trữ dữ liệu dưới dạng các Document có cấu trúc linh hoạt tương tự JSON — chính xác hơn là **BSON** (Binary JSON).

| Khái niệm SQL | Tương ứng trong MongoDB |
|---|---|
| Database | Database  |
| Table (Bảng) | **Collection**  |
| Row (Hàng/Bản ghi) | **Document** |
| Column (Cột) | **Field**  |

---

## II. Cài đặt MongoDB 7.0 trên Ubuntu 22.04

**Bước 1: Import GPG key & thêm official repo**
```bash
# Import signing key
curl -fsSL https://www.mongodb.org/static/pgp/server-7.0.asc \
  | sudo gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg --dearmor

# Thêm repo MongoDB 7.0 cho Ubuntu 22.04
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] \
https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" \
  | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list
```

**Bước 2: Cài MongoDB**
```bash
sudo apt update
sudo apt install -y mongodb-org
```

**Bước 3: Khởi động & bật auto-start**
```bash
sudo systemctl start mongod
sudo systemctl enable mongod
sudo systemctl status mongod
```

---

## III. Kết nối & thao tác CRUD cơ bản

**Kết nối bằng `mongosh`** — shell mới thay thế `mongo`:
```
show dbs              // liệt kê databases
use myapp_db           // chuyển sang / tạo database
show collections       // liệt kê collections
db.version()           // kiểm tra version
exit                    // thoát
```

**Tạo database & collection đầu tiên** — MongoDB tự động tạo database khi insert document đầu tiên, không cần `CREATE DATABASE` như MySQL:

```javascript
use myapp_db

// Insert document đầu tiên → tự động tạo DB + collection
db.products.insertOne({
  name: "Laptop Dell XPS",
  price: 25000000,
  category: "electronics",
  specs: { ram: "16GB", storage: "512GB SSD" },
  tags: ["laptop", "dell", "sale"]
})

// Query lại
db.products.find({ category: "electronics" }).pretty()
```

**Các thao tác CRUD khác:**
```javascript
// Read — tìm nhiều điều kiện
db.products.find({ price: { $gt: 10000000 }, category: "electronics" })

// Update — cập nhật 1 field
db.products.updateOne(
  { name: "Laptop Dell XPS" },
  { $set: { price: 23000000 } }
)

// Update nhiều document cùng lúc
db.products.updateMany({ category: "electronics" }, { $inc: { price: -500000 } })

// Delete
db.products.deleteOne({ name: "Laptop Dell XPS" })
```

---

## IV. File cấu hình `/etc/mongod.conf`

```yaml
systemLog:
  destination: file
  logAppend: true
  path: /var/log/mongodb/mongod.log
  verbosity: 0             # 0=normal, 1-5=debug (tăng khi troubleshoot)

storage:
  dbPath: /var/lib/mongodb
  engine: wiredTiger        # storage engine mặc định và duy nhất nên dùng
  wiredTiger:
    engineConfig:
      cacheSizeGB: 2        # ~50% RAM — quan trọng nhất!
    collectionConfig:
      blockCompressor: snappy   # nén data, tiết kiệm disk

net:
  port: 27017
  bindIp: 127.0.0.1         # chỉ localhost. Remote app: thêm IP app server
  tls:
    mode: requireTLS         # bắt buộc TLS (production)
    certificateKeyFile: /etc/ssl/mongodb/mongodb.pem
    CAFile: /etc/ssl/mongodb/ca.pem

security:
  authorization: enabled    # BẮT BUỘC — xác thực user
  javascriptEnabled: false  # tắt JS execution server-side (bảo mật)

operationProfiling:
  mode: slowOp
  slowOpThresholdMs: 100    # log operation chậm hơn 100ms

replication:
  replSetName: "rs0"         # bỏ comment nếu dùng Replica Set
```

---

## V. Tối ưu hiệu năng

### 5.1. Index

```javascript
// Xem query có dùng index không
db.products.find({ category: "electronics" }).explain("executionStats")
// Tìm: "COLLSCAN" = không có index, "IXSCAN" = có index

// Single field index
db.products.createIndex({ category: 1 })

// Compound index — thứ tự trường quan trọng! (quy tắc ESR: Equality, Sort, Range)
db.orders.createIndex({ user_id: 1, status: 1, created_at: -1 })

// Text index cho full-text search
db.articles.createIndex({ title: "text", content: "text" })
db.articles.find({ $text: { $search: "mongodb devops" } })

// TTL index — tự xóa document sau X giây (dùng cho session, log)
db.sessions.createIndex({ createdAt: 1 }, { expireAfterSeconds: 3600 })

// Xem tất cả index của collection
db.products.getIndexes()
```

### 5.2. Aggregation Pipeline

Ví dụ: tính doanh thu theo category, top 5:
```javascript
db.orders.aggregate([
  // Stage 1: Lọc đơn hàng đã hoàn thành
  { $match: { status: "completed" } },

  // Stage 2: Unwind array items
  { $unwind: "$items" },

  // Stage 3: Nhóm theo category, tính tổng
  { $group: {
    _id: "$items.category",
    total_revenue: { $sum: { $multiply: ["$items.price", "$items.qty"] } },
    order_count: { $sum: 1 }
  }},

  // Stage 4: Sort giảm dần
  { $sort: { total_revenue: -1 } },

  // Stage 5: Lấy top 5
  { $limit: 5 }
])
```

### 5.3. Profiler

```javascript
// Bật profiler level 1 (log slow ops > 100ms)
db.setProfilingLevel(1, { slowms: 100 })

// Xem slow operations gần nhất
db.system.profile.find({}).sort({ ts: -1 }).limit(10).pretty()

// Xem operation chậm nhất, chỉ lấy thông tin quan trọng
db.system.profile.find({}, {
  op: 1, ns: 1, millis: 1,
  "command.filter": 1, planSummary: 1
}).sort({ millis: -1 }).limit(5)

// Tắt profiler khi xong (tốn tài nguyên)
db.setProfilingLevel(0)
```

### 5.4. Monitoring nhanh trong mongosh

```javascript
// Server status tổng quan
db.serverStatus()

// Chỉ xem memory & connections
const s = db.serverStatus()
print("Cache used:", s.wiredTiger.cache["bytes currently in the cache"] / 1024/1024, "MB")
print("Connections:", s.connections.current, "/", s.connections.available)
print("Ops/s:", s.opcounters)

// Xem collection stats
db.products.stats()
// → size, count, avgObjSize, totalIndexSize
```

---

## VI. Bảo mật MongoDB

> MongoDB bị tấn công nhiều nhất trong số các DB phổ biến do **mặc định không có auth** và port 27017 hay bị để lộ ra internet. Hàng trăm nghìn instance từng bị xóa data và đòi ransom. Phần bảo mật này phải làm **trước** khi làm bất cứ thứ gì khác.

### 6.1. Bật Authentication

```javascript
use admin
db.createUser({
  user: "mongoAdmin",
  pwd: "Str0ngAdm!nPass",
  roles: [{ role: "userAdminAnyDatabase", db: "admin" },
          { role: "readWriteAnyDatabase", db: "admin" },
          { role: "dbAdminAnyDatabase", db: "admin" }]
})
exit
```
```bash
# Bật auth trong mongod.conf (thủ công hoặc qua sed):
sudo sed -i 's/#security:/security:\n  authorization: enabled/' /etc/mongod.conf
sudo systemctl restart mongod

# Đăng nhập lại với auth
mongosh -u mongoAdmin -p --authenticationDatabase admin
```

### 6.2. Tạo user riêng cho từng app

```javascript
// Tạo user chỉ đọc/ghi trên 1 database cụ thể
use myapp_db
db.createUser({
  user: "myapp_user",
  pwd: "AppP@ss456!",
  roles: [{ role: "readWrite", db: "myapp_db" }]
})

// User chỉ đọc (cho reporting, analytics)
db.createUser({
  user: "reporter",
  pwd: "ReadOnly789!",
  roles: [{ role: "read", db: "myapp_db" }]
})

// Xem tất cả user
db.getUsers()
```

### 6.3. Firewall

```bash
sudo ufw enable
sudo ufw allow from 10.0.1.50 to any port 27017
sudo ufw allow from 10.0.1.51 to any port 27017

# Kiểm tra không có rule nào mở 27017 ra 0.0.0.0
sudo ufw status verbose

# Scan kiểm tra port có bị expose không
nmap -p 27017 <your-server-ip>
```

### 6.4. TLS/SSL

```bash
# Tạo self-signed cert (dev) hoặc dùng Let's Encrypt/CA cert (prod)
sudo mkdir -p /etc/ssl/mongodb
sudo openssl req -x509 -nodes -newkey rsa:2048 \
  -subj "/CN=mongodb-server" \
  -keyout /etc/ssl/mongodb/mongodb.key \
  -out /etc/ssl/mongodb/mongodb.crt -days 365

# MongoDB cần file .pem (cert + key ghép lại)
sudo cat /etc/ssl/mongodb/mongodb.crt /etc/ssl/mongodb/mongodb.key \
  | sudo tee /etc/ssl/mongodb/mongodb.pem

sudo chown mongodb:mongodb /etc/ssl/mongodb/mongodb.pem
sudo chmod 600 /etc/ssl/mongodb/mongodb.pem
```

### 6.5. Disable HTTP Interface & kiểm tra cấu hình

```bash
# Kiểm tra MongoDB có đang listen ra ngoài không
ss -tlnp | grep 27017
# Kết quả an toàn: 127.0.0.1:27017
# Kết quả nguy hiểm: 0.0.0.0:27017

# Kiểm tra auth đã bật chưa
mongosh --eval "db.adminCommand({getCmdLineOpts:1})" | grep -i auth
```

---



