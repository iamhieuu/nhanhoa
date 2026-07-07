# Triển khai WordPress mô hình 2 Node (Web Server – Database Server)


---

## Thông tin hạ tầng

| Node | Vai trò | IP | Cài đặt |
|------|---------|-----|---------|
| **Node 1 — Web Server** | Apache/Nginx + PHP + WordPress | `192.168.136.131` | Ubuntu 22.04 LTS |
| **Node 2 — Database Server** | MySQL 8.0 | `192.168.136.145` | Ubuntu 22.04 LTS |

---

## Mục lục

- [1. Tổng quan mô hình 2 Node](#1-tổng-quan-mô-hình-2-node)
  - [1.1 Kiến trúc hệ thống](#11-kiến-trúc-hệ-thống)
  - [1.2 So sánh Single Node vs 2 Node](#12-so-sánh-single-node-vs-2-node)
  - [1.3 Luồng kết nối](#13-luồng-kết-nối)
- [2. Chuẩn bị hệ thống](#2-chuẩn-bị-hệ-thống)
  - [2.1 Cập nhật cả 2 node](#21-cập-nhật-cả-2-node)
  - [2.2 Cấu hình Firewall](#22-cấu-hình-firewall)
- [3. Cấu hình Node 2 — Database Server (192.168.136.145)](#3-cấu-hình-node-2--database-server-192168136145)
  - [3.1 Cài đặt MySQL Server](#31-cài-đặt-mysql-server)
  - [3.2 Bảo mật MySQL](#32-bảo-mật-mysql)
  - [3.3 Cấu hình MySQL cho phép kết nối từ xa](#33-cấu-hình-mysql-cho-phép-kết-nối-từ-xa)
  - [3.4 Tạo Database và User cho WordPress](#34-tạo-database-và-user-cho-wordpress)
  - [3.5 Cấu hình Firewall Database Server](#35-cấu-hình-firewall-database-server)
- [4. Cấu hình Node 1 — Web Server (192.168.136.131)](#4-cấu-hình-node-1--web-server-192168136131)
  - [4.1 Cài đặt LAMP/LEMP Stack](#41-cài-đặt-lamplemp-stack)
  - [4.2 Kiểm tra kết nối đến Database Server](#42-kiểm-tra-kết-nối-đến-database-server)
  - [4.3 Tải và cấu hình WordPress](#43-tải-và-cấu-hình-wordpress)
  - [4.4 Cấu hình wp-config.php (remote database)](#44-cấu-hình-wp-configphp-remote-database)
  - [4.5 Cấu hình Virtual Host Apache](#45-cấu-hình-virtual-host-apache)
  - [4.6 Cấu hình Virtual Host Nginx (LEMP)](#46-cấu-hình-virtual-host-nginx-lemp)
- [5. Hoàn tất cài đặt WordPress qua trình duyệt](#5-hoàn-tất-cài-đặt-wordpress-qua-trình-duyệt)
- [6. Cài đặt SSL (Let's Encrypt) trên Web Server](#6-cài-đặt-ssl-lets-encrypt-trên-web-server)
- [7. Kiểm tra và giám sát kết nối 2 node](#7-kiểm-tra-và-giám-sát-kết-nối-2-node)
- [8. Lỗi thường gặp và cách xử lý](#8-lỗi-thường-gặp-và-cách-xử-lý)
- [9. Bảng tóm tắt lệnh quan trọng](#9-bảng-tóm-tắt-lệnh-quan-trọng)
- [References](#references)

---

## 1. Tổng quan mô hình 2 Node

### 1.1 Kiến trúc hệ thống

```
                         Internet
                             │
                    ┌────────▼────────┐
                    │   hieucute.id.vn │
                    │  192.168.136.131 │
                    │                 │
                    │  Apache/Nginx   │   ← Node 1: Web Server
                    │  PHP 8.1        │
                    │  WordPress      │
                    └────────┬────────┘
                             │ MySQL TCP 3306
                    (Private Network)
                    192.168.136.0/24
                             │
                    ┌────────▼────────┐
                    │  192.168.136.145│
                    │                 │
                    │  MySQL 8.0      │   ← Node 2: Database Server
                    │  (No web access)│
                    └─────────────────┘
```

### 1.2 So sánh Single Node vs 2 Node

| Tiêu chí | Single Node | 2 Node |
|----------|------------|--------|
| **Chi phí** | Thấp | Cao hơn |
| **Hiệu năng** | Chia sẻ tài nguyên | Tối ưu từng layer |
| **Khả năng mở rộng** | Giới hạn | Mở rộng độc lập từng node |
| **Bảo mật DB** | DB exposed cùng Web | DB hoàn toàn tách biệt, không public |
| **Backup DB** | Ảnh hưởng đến Web khi backup | Backup DB không ảnh hưởng Web |
| **SPOF** | Một điểm lỗi | Vẫn còn SPOF nếu DB down |
| **Phù hợp** | Dev, SMB website | Production, website traffic cao |

### 1.3 Luồng kết nối

```
Client (Browser)
    │ HTTP/HTTPS
    ▼
Node 1 — Web Server (192.168.136.131)
    │ Apache/Nginx nhận request
    │ PHP xử lý logic WordPress
    │ WordPress cần truy vấn database
    │ TCP port 3306 (Private Network)
    ▼
Node 2 — Database Server (192.168.136.145)
    │ MySQL xử lý query
    │ Trả kết quả về Web Server
    ▼
Node 1 — Web Server
    │ PHP render HTML từ kết quả query
    │ HTTP/HTTPS response
    ▼
Client (Browser)
```

>  **Nguyên tắc bảo mật quan trọng:** Database Server (192.168.136.145) **không mở port 3306 ra Internet** — chỉ cho phép kết nối từ IP của Web Server (192.168.136.131) qua private network.

---

## 2. Chuẩn bị hệ thống

### 2.1 Cập nhật cả 2 node

Thực hiện trên **cả hai node**:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl wget vim net-tools
```

### 2.2 Cấu hình Firewall

**Trên Node 1 — Web Server:**

```bash
sudo ufw allow OpenSSH
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable
sudo ufw status verbose
```

**Trên Node 2 — Database Server:**

```bash
sudo ufw allow OpenSSH
# Chỉ cho phép Web Server kết nối MySQL — KHÔNG mở port 3306 ra Internet
sudo ufw allow from 192.168.136.131 to any port 3306
sudo ufw enable
sudo ufw status verbose
```

Kết quả mong đợi trên Database Server:
```
To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW IN    Anywhere
3306                       ALLOW IN    192.168.136.131
```

>  **Tuyệt đối không** `ufw allow 3306` mà không chỉ định IP nguồn — điều này mở port MySQL ra toàn Internet, cực kỳ nguy hiểm.

---

## 3. Cấu hình Node 2 — Database Server (192.168.136.145)

> Thực hiện toàn bộ mục 3 trên **Node 2 (192.168.136.145)**.

### 3.1 Cài đặt MySQL Server

```bash
sudo apt install -y mysql-server
sudo systemctl start mysql
sudo systemctl enable mysql
sudo systemctl status mysql
```

### 3.2 Bảo mật MySQL

```bash
sudo mysql_secure_installation
```

| Câu hỏi | Trả lời khuyến nghị |
|---------|---------------------|
| VALIDATE PASSWORD component | `Y` |
| Password policy level | `2` (STRONG) |
| Remove anonymous users | `Y` |
| Disallow root login remotely | `Y` |
| Remove test database | `Y` |
| Reload privilege tables | `Y` |

### 3.3 Cấu hình MySQL cho phép kết nối từ xa

Mặc định MySQL chỉ lắng nghe trên `127.0.0.1` (localhost). Cần sửa để lắng nghe trên interface kết nối với Web Server.

Mở file cấu hình MySQL:

```bash
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
```

Tìm dòng `bind-address` và sửa từ `127.0.0.1` thành IP của Database Server:

```ini
# Trước khi sửa:
# bind-address = 127.0.0.1

# Sau khi sửa — lắng nghe trên IP private của Database Server:
bind-address = 192.168.136.145
```

>  Có thể đặt `bind-address = 0.0.0.0` để lắng nghe tất cả interface, nhưng **không khuyến nghị** vì UFW sẽ phải kiểm soát toàn bộ. Tốt hơn là chỉ định IP cụ thể.

Restart MySQL để áp dụng:

```bash
sudo systemctl restart mysql
```

Xác nhận MySQL đang lắng nghe đúng:

```bash
sudo ss -tlnp | grep mysql
# Kết quả đúng: 192.168.136.145:3306
```

### 3.4 Tạo Database và User cho WordPress

```bash
sudo mysql -u root -p
```

```sql
-- Tạo database với charset UTF-8 đầy đủ (hỗ trợ emoji)
CREATE DATABASE wordpress
    DEFAULT CHARACTER SET utf8mb4
    COLLATE utf8mb4_unicode_ci;

-- Tạo user cho kết nối localhost (optional, dùng khi test trực tiếp trên DB server)
CREATE USER 'wpuser'@'localhost' IDENTIFIED BY 'StrongP@ssw0rd_2024';
GRANT ALL PRIVILEGES ON wordpress.* TO 'wpuser'@'localhost';

-- Tạo user cho Web Server kết nối từ xa — IP phải khớp với IP Node 1
CREATE USER 'wpuser'@'192.168.136.131' IDENTIFIED BY 'StrongP@ssw0rd_2024';
GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, INDEX, ALTER
    ON wordpress.*
    TO 'wpuser'@'192.168.136.131';

-- Áp dụng quyền
FLUSH PRIVILEGES;

-- Xác nhận
SHOW DATABASES;
SELECT User, Host FROM mysql.user WHERE User = 'wpuser';
SHOW GRANTS FOR 'wpuser'@'192.168.136.131';

EXIT;
```

>  **Phân quyền tối thiểu cần thiết cho WordPress:**
> `SELECT, INSERT, UPDATE, DELETE` — thao tác dữ liệu hàng ngày.
> `CREATE, DROP, INDEX, ALTER` — cần khi cài đặt, cập nhật plugin/theme tạo/sửa bảng.
> Không cần `SUPER`, `FILE`, `PROCESS` cho WordPress thông thường.

>  **Lưu lại thông tin kết nối:**
> ```
> DB_HOST:     192.168.136.145
> DB_NAME:     wordpress
> DB_USER:     wpuser
> DB_PASSWORD: StrongP@ssw0rd_2024
> DB_PORT:     3306
> ```

### 3.5 Cấu hình Firewall Database Server

Xác nhận lại UFW chỉ cho phép Web Server:

```bash
sudo ufw status verbose
```

Kết quả đúng:
```
Status: active
To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW IN    Anywhere
3306                       ALLOW IN    192.168.136.131
```

---

## 4. Cấu hình Node 1 — Web Server (192.168.136.131)

> Thực hiện toàn bộ mục 4 trên **Node 1 (192.168.136.131)**.

### 4.1 Cài đặt LAMP/LEMP Stack

**Cài LAMP (Apache + PHP):**

```bash
sudo apt install -y apache2
sudo apt install -y php libapache2-mod-php
sudo apt install -y php-{common,mysql,xml,xmlrpc,curl,gd,imagick,cli,dev,imap,mbstring,opcache,soap,zip,intl,bcmath}
sudo a2enmod rewrite
sudo systemctl restart apache2
```

**Hoặc cài LEMP (Nginx + PHP-FPM):**

```bash
sudo apt install -y nginx
sudo apt install -y php8.1-fpm
sudo apt install -y php8.1-{common,mysql,xml,xmlrpc,curl,gd,imagick,cli,dev,imap,mbstring,opcache,soap,zip,intl,bcmath}
sudo systemctl start php8.1-fpm
sudo systemctl enable php8.1-fpm
```

### 4.2 Kiểm tra kết nối đến Database Server

**Bước quan trọng:** Xác nhận Web Server kết nối được MySQL trên Database Server **trước** khi cấu hình WordPress.

```bash
# Cài mysql-client nếu chưa có
sudo apt install -y mysql-client

# Kết nối thử đến Database Server
mysql -u wpuser -h 192.168.136.145 -p wordpress
```

Nhập mật khẩu `StrongP@ssw0rd_2024` → nếu thấy prompt `mysql>` là kết nối thành công.

```sql
-- Kiểm tra trong MySQL prompt
SHOW TABLES;
EXIT;
```

>  Nếu kết nối thất bại, kiểm tra theo thứ tự:
> 1. UFW trên Database Server có cho phép IP `192.168.136.131` không?
> 2. MySQL đang bind đúng IP `192.168.136.145` chưa? (`ss -tlnp | grep mysql`)
> 3. User `wpuser@192.168.136.131` đã được tạo chưa? (`SELECT User, Host FROM mysql.user;`)

### 4.3 Tải và cấu hình WordPress

```bash
# Tải WordPress
cd /tmp
curl -LO https://wordpress.org/latest.tar.gz

# Giải nén
tar xzvf /tmp/latest.tar.gz -C /tmp/

# Copy file cấu hình mẫu
cp /tmp/wordpress/wp-config-sample.php /tmp/wordpress/wp-config.php

# Di chuyển vào thư mục web
sudo mkdir -p /var/www/hieucute.id.vn
sudo cp -a /tmp/wordpress/. /var/www/hieucute.id.vn/

# Phân quyền
sudo chown -R www-data:www-data /var/www/hieucute.id.vn/
sudo find /var/www/hieucute.id.vn/ -type d -exec chmod 755 {} \;
sudo find /var/www/hieucute.id.vn/ -type f -exec chmod 644 {} \;
sudo chmod 600 /var/www/hieucute.id.vn/wp-config.php
```

### 4.4 Cấu hình wp-config.php (remote database)

```bash
sudo nano /var/www/hieucute.id.vn/wp-config.php
```

**Phần 1 — Kết nối Database (điểm khác biệt chính so với Single Node):**

```php
/** Tên database */
define( 'DB_NAME', 'wordpress' );

/** Tên user database */
define( 'DB_USER', 'wpuser' );

/** Mật khẩu database */
define( 'DB_PASSWORD', 'StrongP@ssw0rd_2024' );

/**
 * DB_HOST: Điểm khác biệt quan trọng nhất giữa Single Node và 2 Node
 * Single Node: 'localhost'
 * 2 Node:      IP của Database Server
 */
define( 'DB_HOST', '192.168.136.145' );

/** Charset */
define( 'DB_CHARSET', 'utf8mb4' );
```

>  **Đây là sự khác biệt cốt lõi** giữa cấu hình Single Node và 2 Node:
> - Single Node: `DB_HOST = 'localhost'`
> - 2 Node: `DB_HOST = '192.168.136.145'` (IP của Database Server)

**Phần 2 — Secret Keys**:

```bash
curl -s https://api.wordpress.org/secret-key/1.1/salt/
```

Thay thế 8 dòng key placeholder trong `wp-config.php`.

**Phần 3 — Cấu hình bổ sung:**

```php
define( 'WP_DEBUG',             false );
define( 'FS_METHOD',            'direct' );
define( 'WP_AUTO_UPDATE_CORE',  'minor' );
define( 'WP_POST_REVISIONS',    5 );
define( 'EMPTY_TRASH_DAYS',     30 );
```

### 4.5 Cấu hình Virtual Host Apache

```bash
sudo nano /etc/apache2/sites-available/hieucute.id.vn.conf
```

```apache
<VirtualHost *:80>
    ServerAdmin admin@hieucute.id.vn
    ServerName hieucute.id.vn
    ServerAlias www.hieucute.id.vn
    DocumentRoot /var/www/hieucute.id.vn

    ErrorLog  /var/log/apache2/hieucute.id.vn_error.log
    CustomLog /var/log/apache2/hieucute.id.vn_access.log combined

    <Directory /var/www/hieucute.id.vn/>
        Options FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

```bash
sudo a2ensite hieucute.id.vn.conf
sudo a2dissite 000-default.conf
sudo apache2ctl configtest
sudo systemctl reload apache2
```

### 4.6 Cấu hình Virtual Host Nginx (LEMP)

```bash
sudo nano /etc/nginx/sites-available/hieucute.id.vn
```

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name hieucute.id.vn www.hieucute.id.vn;

    root /var/www/hieucute.id.vn;
    index index.php index.html index.htm;

    access_log /var/log/nginx/hieucute.id.vn_access.log;
    error_log  /var/log/nginx/hieucute.id.vn_error.log;

    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.1-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~* \.(jpg|jpeg|png|gif|ico|css|js|woff2|svg)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
        log_not_found off;
    }

    location ~ /\.ht           { deny all; }
    location ~* /wp-config\.php { deny all; }
    location ~* /xmlrpc\.php    { deny all; }
}
```

```bash
sudo ln -s /etc/nginx/sites-available/hieucute.id.vn /etc/nginx/sites-enabled/
sudo unlink /etc/nginx/sites-enabled/default
sudo nginx -t
sudo systemctl reload nginx
```

---

## 5. Hoàn tất cài đặt WordPress qua trình duyệt

Truy cập `http://hieucute.id.vn` từ trình duyệt.

**Bước 1 — Chọn ngôn ngữ:** Tiếng Việt → **Tiếp tục**.

**Bước 2 — Điền thông tin:**

| Trường | Giá trị |
|--------|---------|
| Tiêu đề trang | `hieucute.id.vn` |
| Tên người dùng | `admin_hieucute` (**không dùng `admin`**) |
| Mật khẩu | *(mật khẩu mạnh ≥16 ký tự)* |
| Email | `admin@hieucute.id.vn` |

**Bước 3:** Nhấn **Cài đặt WordPress** → Đăng nhập tại `http://hieucute.id.vn/wp-admin`.

**Bước 4 — Cấu hình Permalink:**

Dashboard → **Cài đặt** → **Đường dẫn tĩnh** → **Tên bài viết** → **Lưu thay đổi**.

---

## 6. Cài đặt SSL (Let's Encrypt) trên Web Server

Thực hiện trên **Node 1 — Web Server (192.168.136.131)**.

**Cho Apache:**

```bash
sudo apt install -y certbot python3-certbot-apache
sudo certbot --apache -d hieucute.id.vn -d www.hieucute.id.vn
```

**Cho Nginx:**

```bash
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d hieucute.id.vn -d www.hieucute.id.vn
```

Kiểm tra tự động gia hạn:

```bash
sudo certbot renew --dry-run
```

---

## 7. Kiểm tra và giám sát kết nối 2 node

### 7.1 Kiểm tra kết nối từ Web Server đến Database Server

```bash
# Từ Node 1 (Web Server) — kiểm tra port 3306 mở chưa
nc -zv 192.168.136.145 3306

# Kết nối MySQL trực tiếp
mysql -u wpuser -h 192.168.136.145 -p wordpress -e "SHOW TABLES;"
```

### 7.2 Kiểm tra trạng thái MySQL trên Database Server

```bash
# Trên Node 2 (Database Server)
sudo systemctl status mysql

# Xem connection đang active
sudo mysql -e "SHOW STATUS LIKE 'Threads_connected';"
sudo mysql -e "SHOW PROCESSLIST;"
```

### 7.3 Monitor kết nối theo thời gian thực

**Trên Database Server:**

```bash
# Xem các kết nối đang active vào MySQL
sudo mysqladmin -u root -p processlist

# Xem số lượng kết nối hiện tại
sudo mysqladmin -u root -p status | grep -i threads

# Theo dõi slow query log
sudo tail -f /var/log/mysql/mysql-slow.log
```

**Trên Web Server:**

```bash
# Xem error log WordPress liên quan đến database
sudo grep -i "database\|mysql\|DB" /var/log/apache2/hieucute.id.vn_error.log | tail -20
```

### 7.4 Kiểm tra hiệu năng kết nối

```bash
# Từ Web Server — đo thời gian kết nối TCP đến MySQL
time mysql -u wpuser -h 192.168.136.145 -p wordpress -e "SELECT 1;" 2>/dev/null

# Kiểm tra latency mạng giữa 2 node
ping -c 10 192.168.136.145
```

---

## 8. Lỗi thường gặp và cách xử lý

| Lỗi | Node | Nguyên nhân | Cách xử lý |
|-----|------|------------|------------|
| **Error establishing a database connection** | Web Server | `DB_HOST` sai, hoặc user không có quyền remote | Kiểm tra `DB_HOST = '192.168.136.145'`, xem lại GRANT trên DB Server |
| **Can't connect to MySQL server** | Web Server | Port 3306 bị UFW chặn hoặc MySQL chưa bind đúng IP | Kiểm tra `ufw status` trên DB Server, kiểm tra `bind-address` trong `mysqld.cnf` |
| **Access denied for user** | Web Server | User `wpuser@192.168.136.131` chưa được tạo hoặc sai password | Đăng nhập DB Server, kiểm tra `SELECT User, Host FROM mysql.user;` |
| **MySQL bind-address sai** | DB Server | MySQL vẫn bind `127.0.0.1` | Sửa `/etc/mysql/mysql.conf.d/mysqld.cnf`, `bind-address = 192.168.136.145`, restart mysql |
| **WordPress chậm hơn Single Node** | Cả 2 | Network latency giữa 2 node | Bình thường — thêm caching (Redis/Memcached) để giảm số query |
| **DB Server quá tải khi Web Server nhiều request** | DB Server | Thiếu connection pool | Điều chỉnh `max_connections` trong MySQL, dùng connection pooler như ProxySQL |
| **Không kết nối được sau khi đổi IP** | Cả 2 | MySQL user gắn với IP cũ | Tạo lại user với IP mới: `CREATE USER 'wpuser'@'IP_mới' ...` |

**Xem log:**

```bash
# Web Server — lỗi kết nối database
sudo tail -f /var/log/apache2/hieucute.id.vn_error.log

# Database Server — từ chối kết nối
sudo tail -f /var/log/mysql/error.log

# Database Server — xem query đến từ Web Server
sudo tail -f /var/log/mysql/general.log  # Cần bật general_log trước
```

**Bật MySQL General Log để debug (chỉ dùng khi debug, tắt ngay sau đó):**

```sql
-- Bật log tạm thời
SET GLOBAL general_log = 'ON';
SET GLOBAL general_log_file = '/var/log/mysql/general.log';

-- Sau khi debug xong, tắt ngay (log này rất nặng)
SET GLOBAL general_log = 'OFF';
```

---

## 9. Bảng tóm tắt lệnh quan trọng

### Node 1 — Web Server (192.168.136.131)

| Lệnh | Mô tả |
|------|-------|
| `mysql -u wpuser -h 192.168.136.145 -p` | Kiểm tra kết nối từ Web đến DB |
| `nc -zv 192.168.136.145 3306` | Kiểm tra port 3306 mở chưa |
| `sudo systemctl reload apache2` | Reload Apache |
| `sudo systemctl reload nginx` | Reload Nginx |
| `sudo apache2ctl configtest` | Kiểm tra cú pháp Apache |
| `sudo nginx -t` | Kiểm tra cú pháp Nginx |
| `sudo tail -f /var/log/apache2/hieucute.id.vn_error.log` | Xem error log Web |

### Node 2 — Database Server (192.168.136.145)

| Lệnh | Mô tả |
|------|-------|
| `sudo systemctl status mysql` | Kiểm tra trạng thái MySQL |
| `sudo ss -tlnp \| grep mysql` | Xem MySQL đang lắng nghe IP nào |
| `sudo mysqladmin -u root -p processlist` | Xem connections đang active |
| `sudo tail -f /var/log/mysql/error.log` | Xem MySQL error log |
| `mysqldump -u wpuser -p wordpress > wp_db_backup.sql` | Backup database |
| `mysql -u wpuser -p wordpress < wp_db_backup.sql` | Restore database |

### MySQL — Quản lý user kết nối từ xa

| Query | Mô tả |
|-------|-------|
| `SELECT User, Host FROM mysql.user;` | Liệt kê tất cả user và host |
| `SHOW GRANTS FOR 'wpuser'@'192.168.136.131';` | Xem quyền của user |
| `SHOW PROCESSLIST;` | Xem connection đang active |
| `SHOW STATUS LIKE 'Threads_connected';` | Số kết nối hiện tại |
| `SHOW VARIABLES LIKE 'bind_address';` | Xem MySQL đang bind IP nào |
| `SHOW VARIABLES LIKE 'max_connections';` | Xem giới hạn kết nối tối đa |

---

## References

1. [Set Up a Remote Database to Optimize Site Performance with MySQL — DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-remote-database-to-optimize-site-performance-with-mysql)
2. [How To Install WordPress on Ubuntu 22.04 with a LAMP Stack — DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-install-wordpress-on-ubuntu-22-04-with-a-lamp-stack)
3. [MySQL 8.0 — Configuring Remote Access — MySQL Documentation](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_bind_address)
4. [WordPress Security — WordPress.org](https://wordpress.org/documentation/article/hardening-wordpress/)
5. [UFW Essentials — DigitalOcean](https://www.digitalocean.com/community/tutorials/ufw-essentials-common-firewall-rules-and-commands)
