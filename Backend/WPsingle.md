# Triển khai WordPress trên LAMP/LEMP Stack (Single Node)

---

## Mục lục

- [1. Giới thiệu WordPress](#1-giới-thiệu-wordpress)
  - [1.1 WordPress là gì](#11-wordpress-là-gì)
  - [1.2 Ưu điểm của WordPress](#12-ưu-điểm-của-wordpress)
  - [1.3 Kiến trúc Single Node](#13-kiến-trúc-single-node)
- [2. Chuẩn bị — Khởi tạo Database](#2-chuẩn-bị--khởi-tạo-database)
  - [2.1 Tạo Database và User MySQL](#21-tạo-database-và-user-mysql)
- [3. Tải và cấu hình WordPress](#3-tải-và-cấu-hình-wordpress)
  - [3.1 Tải WordPress](#31-tải-wordpress)
  - [3.2 Giải nén và phân quyền](#32-giải-nén-và-phân-quyền)
  - [3.3 Cấu hình wp-config.php](#33-cấu-hình-wp-configphp)
- [4. Cấu hình Web Server](#4-cấu-hình-web-server)
  - [4.1 Cấu hình Apache (LAMP)](#41-cấu-hình-apache-lamp)
  - [4.2 Cấu hình Nginx (LEMP)](#42-cấu-hình-nginx-lemp)
- [5. Hoàn tất cài đặt WordPress qua trình duyệt](#5-hoàn-tất-cài-đặt-wordpress-qua-trình-duyệt)
- [6. Cài đặt SSL (Let's Encrypt)](#6-cài-đặt-ssl-lets-encrypt)
  - [6.1 SSL cho Apache](#61-ssl-cho-apache)
  - [6.2 SSL cho Nginx](#62-ssl-cho-nginx)
- [7. Cấu hình bảo mật WordPress cơ bản](#7-cấu-hình-bảo-mật-wordpress-cơ-bản)
- [8. Lỗi thường gặp và cách xử lý](#8-lỗi-thường-gặp-và-cách-xử-lý)
- [9. Bảng tóm tắt lệnh quan trọng](#9-bảng-tóm-tắt-lệnh-quan-trọng)
- [References](#references)

---

## 1. Giới thiệu WordPress

### 1.1 WordPress là gì

WordPress là hệ thống quản trị nội dung mã nguồn mở, viết bằng **PHP** và sử dụng **MySQL/MariaDB** làm cơ sở dữ liệu. Ra mắt năm 2003, WordPress hiện là CMS phổ biến nhất thế giới, chiếm hơn **43% số website** trên toàn Internet

| Thông tin | Chi tiết |
|-----------|---------|
| Ngôn ngữ | PHP |
| Database | MySQL / MariaDB |

### 1.2 Ưu điểm của WordPress

- **Dễ sử dụng:** Giao diện quản trị trực quan, không yêu cầu kiến thức lập trình sâu.
- **Hệ sinh thái phong phú:** Hơn 60.000 plugin và 10.000 theme miễn phí/thương mại.
- **Cộng đồng lớn:** Tài liệu đầy đủ, hỗ trợ đông đảo trên toàn thế giới.
- **SEO thân thiện:** Cấu trúc URL sạch, tích hợp tốt với các công cụ SEO.
- **Đa ngôn ngữ:** Hỗ trợ tiếng Việt và hơn 70 ngôn ngữ khác.
- **Linh hoạt:** Từ blog cá nhân đến web bán hàng, tin tức, landing page.
- **Bảo mật:** Cập nhật vá lỗi thường xuyên từ đội ngũ WordPress Security Team.

### 1.3 Kiến trúc Single Node

Trong mô hình Single Node, toàn bộ stack (Web Server + PHP + MySQL + WordPress) chạy trên **một máy chủ duy nhất**:

```
                    ┌─────────────────────────────────────┐
                    │         103.159.51.228               │
                    │                                     │
Client ────HTTP/S──►│  Apache/Nginx ──PHP── WordPress     │
                    │                   │                 │
                    │              MySQL/MariaDB           │
                    │           (localhost:3306)           │
                    └─────────────────────────────────────┘
```

**Ưu điểm:** Đơn giản, chi phí thấp, dễ quản lý.

**Nhược điểm:** Không có khả năng mở rộng ngang, một điểm lỗi (SPOF — Single Point of Failure).

**Phù hợp với:** Website nhỏ đến trung bình, môi trường phát triển, blog cá nhân, doanh nghiệp vừa và nhỏ.

---

## 2. Chuẩn bị — Khởi tạo Database

### 2.1 Tạo Database và User MySQL

>  **Nguyên tắc bảo mật:** Không bao giờ dùng tài khoản `root` MySQL cho ứng dụng web. Tạo user riêng với quyền giới hạn chỉ trên database của WordPress.

Đăng nhập MySQL với quyền root:

```bash
sudo mysql
```

Tạo database với charset UTF-8:

```sql
CREATE DATABASE wordpress
    DEFAULT CHARACTER SET utf8mb4
    COLLATE utf8mb4_unicode_ci;
```

Tạo user riêng cho WordPress:

```sql
CREATE USER 'wpuser'@'localhost' IDENTIFIED BY 'StrongP@ssw0rd_2024';
```

Cấp quyền chỉ trên database `wordpress`:

```sql
GRANT ALL PRIVILEGES ON wordpress.* TO 'wpuser'@'localhost';
```

Áp dụng thay đổi quyền:

```sql
FLUSH PRIVILEGES;
```

Xác nhận:

```sql
SHOW DATABASES;
SHOW GRANTS FOR 'wpuser'@'localhost';
EXIT;
```

>  **Lưu lại 3 thông tin sau để dùng ở bước cấu hình wp-config.php:**
> ```
> DB_NAME:     wordpress
> DB_USER:     wpuser
> DB_PASSWORD: StrongP@ssw0rd_2024
> DB_HOST:     localhost
> ```

---

## 3. Tải và cấu hình WordPress

### 3.1 Tải WordPress

Tải source code WordPress mới nhất từ trang chính thức về thư mục `/tmp`:

```bash
cd /tmp
curl -LO https://wordpress.org/latest.tar.gz
```

Kiểm tra file đã tải:

```bash
ls -lh /tmp/latest.tar.gz
```

### 3.2 Giải nén và phân quyền

Giải nén source code:

```bash
tar xzvf /tmp/latest.tar.gz -C /tmp/
```

Copy file cấu hình mẫu:

```bash
cp /tmp/wordpress/wp-config-sample.php /tmp/wordpress/wp-config.php
```

Di chuyển toàn bộ vào thư mục web:

```bash
sudo cp -a /tmp/wordpress/. /var/www/hieucute.id.vn/
```

Gán quyền sở hữu cho user `www-data` (user chạy Apache/Nginx):

```bash
sudo chown -R www-data:www-data /var/www/hieucute.id.vn/
```

Đặt quyền chuẩn:

```bash
# Thư mục: 755
sudo find /var/www/hieucute.id.vn/ -type d -exec chmod 755 {} \;

# File: 644
sudo find /var/www/hieucute.id.vn/ -type f -exec chmod 644 {} \;
```

>  Không đặt quyền `777` cho bất kỳ file/thư mục nào — đây là lỗ hổng bảo mật nghiêm trọng.

### 3.3 Cấu hình wp-config.php

#### Bước 1 — Lấy WordPress Secret Keys

Truy cập API sinh key tự động của WordPress:

```bash
curl -s https://api.wordpress.org/secret-key/1.1/salt/
```

Kết quả trả về là 8 dòng giống như:

```php
define('AUTH_KEY',         'n|%+3[Qe...');
define('SECURE_AUTH_KEY',  'z&A_t7...');
...
```

Copy toàn bộ output này, dùng ở bước tiếp theo.

#### Bước 2 — Chỉnh sửa wp-config.php

```bash
sudo nano /var/www/hieucute.id.vn/wp-config.php
```

**Phần 1 — Cấu hình kết nối Database** (tìm và sửa 4 dòng `define`):

```php
define( 'DB_NAME',     'wordpress' );
define( 'DB_USER',     'wpuser' );
define( 'DB_PASSWORD', 'StrongP@ssw0rd_2024' );
define( 'DB_HOST',     'localhost' );
define( 'DB_CHARSET',  'utf8mb4' );
```

**Phần 2 — Thay thế Secret Keys** (tìm đoạn `define('AUTH_KEY',...` đến `define('NONCE_SALT,...`):

Xóa toàn bộ 8 dòng placeholder (dạng `put your unique phrase here`) và dán 8 dòng key vừa lấy từ API vào.

**Phần 3 — Thêm cấu hình bổ sung** (thêm vào cuối file, trước dòng `/* That's all, stop editing! */`):

```php
/* Cấu hình bổ sung */
define( 'WP_DEBUG',         false );
define( 'WP_DEBUG_LOG',     false );
define( 'FS_METHOD',        'direct' );
define( 'WP_AUTO_UPDATE_CORE', 'minor' );

/* Giới hạn post revision để tiết kiệm database */
define( 'WP_POST_REVISIONS', 5 );

/* Tự động dọn dẹp thùng rác sau 30 ngày */
define( 'EMPTY_TRASH_DAYS', 30 );
```

---

## 4. Cấu hình Web Server

### 4.1 Cấu hình Apache (LAMP)

#### Bật module cần thiết

```bash
sudo a2enmod rewrite
sudo systemctl restart apache2
```

>  `mod_rewrite` là bắt buộc để WordPress Permalink (đường dẫn thân thiện) hoạt động.

#### Tạo Virtual Host

```bash
sudo nano /etc/apache2/sites-available/hieucute.id.vn.conf
```

Nội dung chuẩn production:

```apache
<VirtualHost *:80>
    ServerAdmin admin@hieucute.id.vn
    ServerName hieucute.id.vn
    ServerAlias www.hieucute.id.vn
    DocumentRoot /var/www/hieucute.id.vn

    # Log riêng cho từng domain
    ErrorLog  /var/log/apache2/hieucute.id.vn_error.log
    CustomLog /var/log/apache2/hieucute.id.vn_access.log combined

    # Cho phép .htaccess override — bắt buộc cho WordPress
    <Directory /var/www/hieucute.id.vn/>
        Options FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

#### Kích hoạt Virtual Host

```bash
# Bật site WordPress
sudo a2ensite hieucute.id.vn.conf

# Tắt site mặc định
sudo a2dissite 000-default.conf

# Kiểm tra cú pháp
sudo apache2ctl configtest
# Kết quả đúng: Syntax OK

# Reload Apache
sudo systemctl reload apache2
```

#### Kiểm tra

Truy cập trình duyệt: `http://hieucute.id.vn`

Kết quả đúng: Trang cài đặt WordPress xuất hiện với các bước chọn ngôn ngữ.

---

### 4.2 Cấu hình Nginx (LEMP)

#### Tạo Virtual Host

```bash
sudo nano /etc/nginx/sites-available/hieucute.id.vn
```

Nội dung chuẩn production cho WordPress:

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name hieucute.id.vn www.hieucute.id.vn;

    root /var/www/hieucute.id.vn;
    index index.php index.html index.htm;

    # Log riêng cho từng domain
    access_log /var/log/nginx/hieucute.id.vn_access.log;
    error_log  /var/log/nginx/hieucute.id.vn_error.log;

    # WordPress Permalink — try_files bắt buộc
    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    # Xử lý PHP qua PHP-FPM
    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.1-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    # Tối ưu static file — cache 1 năm
    location ~* \.(jpg|jpeg|png|gif|ico|css|js|woff2|svg)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
        log_not_found off;
    }

    # Bảo mật — chặn truy cập file nhạy cảm
    location ~ /\.ht           { deny all; }
    location ~ /\.git          { deny all; }
    location ~* /wp-config\.php { deny all; }
    location ~* /xmlrpc\.php    { deny all; }
}
```

#### Kích hoạt Virtual Host

```bash
# Tạo symlink kích hoạt site
sudo ln -s /etc/nginx/sites-available/hieucute.id.vn /etc/nginx/sites-enabled/

# Tắt site mặc định
sudo unlink /etc/nginx/sites-enabled/default

# Kiểm tra cú pháp
sudo nginx -t
# Kết quả đúng:
# nginx: the configuration file syntax is ok
# nginx: configuration file test is successful

# Reload Nginx
sudo systemctl reload nginx
```

#### Kiểm tra

Truy cập: `http://hieucute.id.vn`

---

## 5. Hoàn tất cài đặt WordPress qua trình duyệt

Truy cập `http://hieucute.id.vn` — trình cài đặt WordPress sẽ tự động khởi chạy.

**Bước 1 — Chọn ngôn ngữ:** Chọn **Tiếng Việt** → **Tiếp tục**.

**Bước 2 — Điền thông tin website:**

| Trường | Giá trị khuyến nghị | Ghi chú |
|--------|---------------------|---------|
| Tiêu đề trang | `hieucute.id.vn` | Có thể đổi sau |
| Tên người dùng | `admin_hieucute` | **Không dùng `admin`** |
| Mật khẩu | *(tạo mạnh, ≥16 ký tự)* | Lưu lại cẩn thận |
| Email | `admin@hieucute.id.vn` | Email thật |
| Khả năng hiển thị | Tùy chọn | Bỏ tick nếu không muốn ẩn khỏi Google |

**Bước 3:** Nhấn **Cài đặt WordPress**.

**Bước 4:** Đăng nhập trang quản trị:
```
https://hieucute.id.vn/wp-admin
```

**Bước 5 — Cấu hình Permalink (bắt buộc):**

WordPress Dashboard → **Cài đặt** → **Đường dẫn tĩnh** → Chọn **Tên bài viết** → **Lưu thay đổi**.

>  Permalink dạng "Tên bài viết" (`/ten-bai-viet/`) thân thiện với SEO và dễ đọc hơn dạng mặc định (`/?p=123`).

---

## 6. Cài đặt SSL (Let's Encrypt)

### 6.1 SSL cho Apache

```bash
# Cài Certbot và plugin Apache
sudo apt install -y certbot python3-certbot-apache

# Cấp chứng chỉ SSL
sudo certbot --apache -d hieucute.id.vn -d www.hieucute.id.vn
```

Certbot tự động:
- Cấp chứng chỉ Let's Encrypt.
- Sửa Virtual Host Apache bật HTTPS.
- Tạo redirect HTTP → HTTPS (301).

### 6.2 SSL cho Nginx

```bash
# Cài Certbot và plugin Nginx
sudo apt install -y certbot python3-certbot-nginx

# Cấp chứng chỉ SSL
sudo certbot --nginx -d hieucute.id.vn -d www.hieucute.id.vn
```

### Kiểm tra tự động gia hạn

```bash
sudo certbot renew --dry-run
# Kết quả đúng: Congratulations, all simulated renewals succeeded
```

### Cập nhật WordPress URL sang HTTPS

Sau khi cài SSL, cần cập nhật địa chỉ trong WordPress:

**Cách 1 — Qua Dashboard:**
Cài đặt → Chung → Sửa **Địa chỉ WordPress** và **Địa chỉ trang web** từ `http://` thành `https://`.

**Cách 2 — Qua WP-CLI hoặc MySQL:**

```sql
UPDATE wp_options SET option_value = 'https://hieucute.id.vn'
WHERE option_name IN ('siteurl', 'home');
```

---

## 7. Cấu hình bảo mật WordPress cơ bản

### 7.1 Phân quyền file nghiêm ngặt

```bash
# Đặt quyền chuẩn
sudo find /var/www/hieucute.id.vn/ -type d -exec chmod 755 {} \;
sudo find /var/www/hieucute.id.vn/ -type f -exec chmod 644 {} \;

# Bảo vệ wp-config.php
sudo chmod 600 /var/www/hieucute.id.vn/wp-config.php
sudo chown www-data:www-data /var/www/hieucute.id.vn/wp-config.php
```

### 7.2 Bảo vệ wp-config.php qua .htaccess (Apache)

Thêm vào file `/var/www/hieucute.id.vn/.htaccess`:

```apache
# Bảo vệ wp-config.php
<Files wp-config.php>
    Order Allow,Deny
    Deny from all
</Files>

# Chặn truy cập file .htaccess
<Files .htaccess>
    Order Allow,Deny
    Deny from all
</Files>

# Tắt directory listing
Options -Indexes

# Chặn xmlrpc.php nếu không dùng
<Files xmlrpc.php>
    Order Allow,Deny
    Deny from all
</Files>
```

### 7.3 Bảo mật wp-admin

Giới hạn truy cập wp-admin chỉ từ IP cụ thể (thêm vào `.htaccess`):

```apache
# Chỉ cho phép IP của bạn truy cập wp-admin
<Files wp-login.php>
    Order Deny,Allow
    Deny from all
    Allow from 14.191.163.146
</Files>
```

### 7.4 Thêm Security Headers (Apache)

Thêm vào Virtual Host hoặc `.htaccess`:

```apache
<IfModule mod_headers.c>
    Header always set X-Content-Type-Options "nosniff"
    Header always set X-Frame-Options "SAMEORIGIN"
    Header always set X-XSS-Protection "1; mode=block"
    Header always set Referrer-Policy "no-referrer-when-downgrade"
    Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"
</IfModule>
```

---

## 8. Lỗi thường gặp và cách xử lý

| Lỗi | Nguyên nhân | Cách xử lý |
|-----|------------|------------|
| **Trang trắng hoàn toàn (White Screen of Death)** | Lỗi PHP, hết memory | Bật `WP_DEBUG`, kiểm tra log: `tail -f /var/log/apache2/hieucute.id.vn_error.log` |
| **Error establishing database connection** | Sai thông tin DB trong wp-config.php | Kiểm tra lại `DB_NAME`, `DB_USER`, `DB_PASSWORD`, `DB_HOST` |
| **404 trên tất cả trang (chỉ Home OK)** | Chưa bật `mod_rewrite` hoặc chưa set Permalink | `sudo a2enmod rewrite`, vào WP → Cài đặt → Đường dẫn tĩnh → Lưu |
| **403 Forbidden** | Sai quyền file/thư mục | `find ... -exec chmod 755/644 {} \;` |
| **Trang cài đặt lặp lại** | Đã cài nhưng wp-config chưa đúng | Kiểm tra lại `wp-config.php`, xóa `/tmp/latest.tar.gz` cũ |
| **Upload ảnh thất bại** | Thiếu quyền thư mục `wp-content/uploads` | `chown www-data:www-data wp-content/uploads && chmod 775 wp-content/uploads` |
| **Mixed content sau SSL** | Các URL trong DB vẫn là `http://` | Cài plugin **Better Search Replace**, đổi `http://hieucute.id.vn` → `https://hieucute.id.vn` |
| **Nginx: File not found (.php)** | Sai đường dẫn `fastcgi_pass` socket | Kiểm tra `ls /run/php/` để tìm tên socket đúng |

**Xem log nhanh:**

```bash
# Apache
sudo tail -f /var/log/apache2/hieucute.id.vn_error.log

# Nginx
sudo tail -f /var/log/nginx/hieucute.id.vn_error.log

# PHP-FPM
sudo tail -f /var/log/php8.1-fpm.log

# WordPress debug log (bật WP_DEBUG_LOG trước)
sudo tail -f /var/www/hieucute.id.vn/wp-content/debug.log
```

---

## 9. Bảng tóm tắt lệnh quan trọng

### Quản lý dịch vụ

| Lệnh | Mô tả |
|------|-------|
| `sudo systemctl reload apache2` | Reload Apache (không ngắt kết nối) |
| `sudo systemctl reload nginx` | Reload Nginx |
| `sudo systemctl restart php8.1-fpm` | Restart PHP-FPM |
| `sudo apache2ctl configtest` | Kiểm tra cú pháp Apache |
| `sudo nginx -t` | Kiểm tra cú pháp Nginx |

### Phân quyền WordPress

| Lệnh | Mô tả |
|------|-------|
| `sudo chown -R www-data:www-data /var/www/hieucute.id.vn/` | Gán chủ sở hữu |
| `sudo find ... -type d -exec chmod 755 {} \;` | Quyền thư mục |
| `sudo find ... -type f -exec chmod 644 {} \;` | Quyền file |
| `sudo chmod 600 wp-config.php` | Bảo vệ wp-config.php |

### MySQL WordPress

| Lệnh/Query | Mô tả |
|------------|-------|
| `sudo mysql -u root -p` | Đăng nhập MySQL |
| `SHOW DATABASES;` | Liệt kê database |
| `mysqldump -u wpuser -p wordpress > wp_backup.sql` | Backup DB |
| `mysql -u wpuser -p wordpress < wp_backup.sql` | Restore DB |

---

## References

1. [How To Install WordPress on Ubuntu 22.04 with a LAMP Stack — DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-install-wordpress-on-ubuntu-22-04-with-a-lamp-stack)
2. [How to Install WordPress with LEMP on Ubuntu — DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-install-wordpress-with-lemp-on-ubuntu)
3. [WordPress Hardening — WordPress.org](https://wordpress.org/documentation/article/hardening-wordpress/)
4. [Certbot Documentation — Let's Encrypt](https://certbot.eff.org/instructions)
5. [MySQL 8.0 Reference Manual — Oracle](https://dev.mysql.com/doc/refman/8.0/en/)
