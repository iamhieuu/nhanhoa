# Triển khai LAMP & LEMP Stack trên Ubuntu 22.04 LTS


> **Mục tiêu:** Cài đặt và cấu hình đầy đủ LAMP Stack (Apache + MySQL + PHP) và LEMP Stack (Nginx + MySQL + PHP-FPM) phục vụ hosting website production.

---

## Mục lục

- [1. Tổng quan LAMP và LEMP](#1-tổng-quan-lamp-và-lemp)
  - [1.1 LAMP Stack là gì](#11-lamp-stack-là-gì)
  - [1.2 LEMP Stack là gì](#12-lemp-stack-là-gì)
  - [1.3 So sánh LAMP và LEMP](#13-so-sánh-lamp-và-lemp)
  - [1.4 Kiến trúc luồng xử lý request](#14-kiến-trúc-luồng-xử-lý-request)
- [2. Chuẩn bị hệ thống](#2-chuẩn-bị-hệ-thống)
  - [2.1 Cập nhật hệ thống](#21-cập-nhật-hệ-thống)
  - [2.2 Cấu hình Firewall (UFW)](#22-cấu-hình-firewall-ufw)
  - [2.3 Cấu hình Hostname và DNS](#23-cấu-hình-hostname-và-dns)
- [3. Triển khai LAMP Stack](#3-triển-khai-lamp-stack)
  - [3.1 Cài đặt Apache](#31-cài-đặt-apache)
  - [3.2 Cấu hình Virtual Host Apache](#32-cấu-hình-virtual-host-apache)
  - [3.3 Cài đặt MySQL Server](#33-cài-đặt-mysql-server)
  - [3.4 Cài đặt PHP](#34-cài-đặt-php)
  - [3.5 Kết hợp PHP với Apache](#35-kết-hợp-php-với-apache)
  - [3.6 Kiểm tra LAMP hoàn chỉnh](#36-kiểm-tra-lamp-hoàn-chỉnh)
- [4. Triển khai LEMP Stack](#4-triển-khai-lemp-stack)
  - [4.1 Cài đặt Nginx](#41-cài-đặt-nginx)
  - [4.2 Cài đặt PHP-FPM](#42-cài-đặt-php-fpm)
  - [4.3 Cấu hình Virtual Host Nginx](#43-cấu-hình-virtual-host-nginx)
  - [4.4 Kết hợp Nginx với PHP-FPM](#44-kết-hợp-nginx-với-php-fpm)
  - [4.5 Kiểm tra LEMP hoàn chỉnh](#45-kiểm-tra-lemp-hoàn-chỉnh)
- [5. Cài đặt và cấu hình MySQL (dùng chung cho cả LAMP và LEMP)](#5-cài-đặt-và-cấu-hình-mysql-dùng-chung)
  - [5.1 Bảo mật MySQL](#51-bảo-mật-mysql)
  - [5.2 Tạo Database và User cho Website](#52-tạo-database-và-user-cho-website)
- [6. Cài đặt SSL (Let's Encrypt) với Certbot](#6-cài-đặt-ssl-lets-encrypt-với-certbot)
  - [6.1 Cài đặt Certbot](#61-cài-đặt-certbot)
  - [6.2 Cấp SSL cho Apache](#62-cấp-ssl-cho-apache)
  - [6.3 Cấp SSL cho Nginx](#63-cấp-ssl-cho-nginx)
  - [6.4 Tự động gia hạn SSL](#64-tự-động-gia-hạn-ssl)
- [7. Tối ưu hóa hiệu năng](#7-tối-ưu-hóa-hiệu-năng)
  - [7.1 Bật Gzip Compression](#71-bật-gzip-compression)
  - [7.2 Bật Browser Caching](#72-bật-browser-caching)
  - [7.3 Tối ưu PHP-FPM](#73-tối-ưu-php-fpm)
- [8. Bash Script tự động cài đặt](#8-bash-script-tự-động-cài-đặt)
  - [8.1 Script cài đặt LAMP](#81-script-cài-đặt-lamp)
  - [8.2 Script cài đặt LEMP](#82-script-cài-đặt-lemp)
- [9. Lỗi thường gặp và cách xử lý](#9-lỗi-thường-gặp-và-cách-xử-lý)
- [10. Bảng tóm tắt lệnh quan trọng](#10-bảng-tóm-tắt-lệnh-quan-trọng)
- [References](#references)

---

## 1. Tổng quan LAMP và LEMP

### 1.1 LAMP Stack là gì

**LAMP** là viết tắt của bốn thành phần phần mềm mã nguồn mở, xếp chồng lên nhau tạo thành nền tảng để triển khai ứng dụng web động:

| Thành phần | Vai trò | Phiên bản khuyến nghị (Ubuntu 22.04) |
|-----------|---------|--------------------------------------|
| **L**inux | Hệ điều hành nền tảng | Ubuntu 22.04 LTS |
| **A**pache | Web Server — xử lý HTTP/HTTPS request | Apache 2.4.x |
| **M**ySQL | Hệ quản trị cơ sở dữ liệu quan hệ | MySQL 8.0 / MariaDB 10.6 |
| **P**HP | Ngôn ngữ lập trình phía server | PHP 8.1 / 8.2 |

**Luồng hoạt động:**
```
Trình duyệt → Apache (nhận request) → PHP (xử lý logic) → MySQL (truy vấn dữ liệu) → Apache (trả HTML) → Trình duyệt
```

**Ứng dụng phổ biến chạy trên LAMP:** WordPress, Joomla, Drupal, Magento, Laravel.

---

### 1.2 LEMP Stack là gì

**LEMP** khác LAMP ở chỗ thay **Apache** bằng **Nginx** (đọc là "engine-x" — chữ **E** trong LEMP).

| Thành phần | Vai trò | Phiên bản khuyến nghị (Ubuntu 22.04) |
|-----------|---------|--------------------------------------|
| **L**inux | Hệ điều hành nền tảng | Ubuntu 22.04 LTS |
| **E**ngine-x (Nginx) | Web Server / Reverse Proxy | Nginx 1.18.x |
| **M**ySQL | Hệ quản trị cơ sở dữ liệu quan hệ | MySQL 8.0 / MariaDB 10.6 |
| **P**HP-FPM | Ngôn ngữ lập trình phía server (FastCGI Process Manager) | PHP 8.1-FPM |

**Điểm khác biệt quan trọng:** Nginx không xử lý PHP trực tiếp như Apache (mod_php). Nginx chuyển request PHP sang **PHP-FPM** (FastCGI) qua Unix socket hoặc TCP socket.

---

### 1.3 So sánh LAMP và LEMP

| Tiêu chí | LAMP (Apache) | LEMP (Nginx) |
|----------|--------------|-------------|
| **Xử lý PHP** | mod_php nhúng trực tiếp | PHP-FPM qua FastCGI — tách biệt hẳn |
| **Static files** | Tốt | **Xuất sắc** — Nginx tối ưu cho static file |
| **Concurrent connections** | ~150–200 kết nối/worker | Hàng nghìn kết nối/worker (event-driven) |
| **RAM tiêu thụ** | Cao hơn (mỗi process = 1 thread) | **Thấp hơn đáng kể** (async event loop) |
| **Cấu hình** | `.htaccess` linh hoạt | Không có `.htaccess` — cấu hình tập trung |
| **Phù hợp với** | Shared hosting, site có nhiều `.htaccess` rule | VPS, high-traffic site, microservices |
| **WordPress** | Hoạt động tốt | Hoạt động tốt (cần cấu hình rewrite thêm) |

> 💡 **Khi nào chọn cái nào:**
> - Chọn **LAMP** nếu: ứng dụng cần `.htaccess`, cần mod_rewrite phức tạp, hoặc bạn quen quản lý Apache.
> - Chọn **LEMP** nếu: server cần xử lý nhiều traffic, tài nguyên RAM hạn chế, hoặc cần reverse proxy.

---

### 1.4 Kiến trúc luồng xử lý request

**LAMP:**
```
Client → Apache (port 80/443)
              │
              ├─ Static file (.html, .css, .jpg) → Trả về trực tiếp
              │
              └─ Dynamic file (.php) → mod_php (chạy trong process Apache) → MySQL
```

**LEMP:**
```
Client → Nginx (port 80/443)
              │
              ├─ Static file (.html, .css, .jpg) → Trả về trực tiếp (rất nhanh)
              │
              └─ Dynamic file (.php) → PHP-FPM (process riêng biệt, Unix socket) → MySQL
```

---

## 2. Chuẩn bị hệ thống

### 2.1 Cập nhật hệ thống

Luôn cập nhật hệ thống trước khi cài đặt bất kỳ thành phần nào — đảm bảo các package mới nhất, vá lỗi bảo mật đã được áp dụng.

```bash
sudo apt update && sudo apt upgrade -y
```

Cài đặt các công cụ cơ bản cần thiết:
```bash
sudo apt install -y curl wget git vim net-tools software-properties-common apt-transport-https ca-certificates
```

Kiểm tra phiên bản Ubuntu:
```bash
lsb_release -a
```

---

### 2.2 Cấu hình Firewall (UFW)

UFW (Uncomplicated Firewall) là firewall mặc định trên Ubuntu 22.04. Cần cấu hình trước khi cài web server để đảm bảo các port cần thiết được mở.

```bash
# Bật UFW
sudo ufw enable

# Cho phép SSH (bắt buộc — không mở SSH sẽ mất quyền truy cập server)
sudo ufw allow OpenSSH

# Cho phép HTTP và HTTPS
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Kiểm tra trạng thái
sudo ufw status verbose
```

Kết quả mong đợi:
```
Status: active
To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW IN    Anywhere
80/tcp                     ALLOW IN    Anywhere
443/tcp                    ALLOW IN    Anywhere
```

> ⚠️ **Quan trọng:** Luôn `allow OpenSSH` **trước** khi `enable ufw`. Nếu bật UFW mà chưa cho phép SSH, kết nối SSH hiện tại sẽ bị ngắt ngay lập tức — mất quyền truy cập server từ xa.

---

### 2.3 Cấu hình Hostname và DNS

```bash
# Đặt hostname cho server
sudo hostnamectl set-hostname hieucute.id.vn

# Kiểm tra
hostnamectl
```

Đảm bảo DNS đã trỏ đúng trước khi cài SSL:
```bash
dig A hieucute.id.vn +short
# Kết quả phải trả về: 103.159.51.228
```

---

## 3. Triển khai LAMP Stack

### 3.1 Cài đặt Apache

#### Cài đặt

```bash
sudo apt install -y apache2
```

#### Khởi động và bật auto-start

```bash
sudo systemctl start apache2
sudo systemctl enable apache2
```

#### Kiểm tra trạng thái

```bash
sudo systemctl status apache2
```

Kết quả đúng:
```
● apache2.service - The Apache HTTP Server
     Loaded: loaded (/lib/systemd/system/apache2.service; enabled)
     Active: active (running)
```

#### Bật các module cần thiết

```bash
# mod_rewrite — cần cho WordPress, Laravel và các framework MVC
sudo a2enmod rewrite

# mod_headers — cần cho CORS, security headers
sudo a2enmod headers

# mod_ssl — cần cho HTTPS
sudo a2enmod ssl

# Áp dụng thay đổi
sudo systemctl restart apache2
```

#### Kiểm tra Apache hoạt động

```bash
curl -I http://103.159.51.228
# hoặc
curl -I http://localhost
```

Kết quả đúng: `HTTP/1.1 200 OK` và `Server: Apache/2.4.x`

---

### 3.2 Cấu hình Virtual Host Apache

Virtual Host cho phép một server chạy nhiều website trên cùng một IP, phân biệt theo domain name.

#### Tạo cấu trúc thư mục website

```bash
# Tạo thư mục chứa code website
sudo mkdir -p /var/www/hieucute.id.vn/public_html

# Gán quyền sở hữu cho user hiện tại
sudo chown -R $USER:$USER /var/www/hieucute.id.vn

# Đặt quyền chuẩn
sudo chmod -R 755 /var/www/hieucute.id.vn
```

#### Tạo file index thử nghiệm

```bash
nano /var/www/hieucute.id.vn/public_html/index.html
```

Nội dung file:
```html
<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <title>hieucute.id.vn - LAMP Stack</title>
    <style>
        body { font-family: sans-serif; text-align: center; margin-top: 100px; }
        h1 { color: #2c3e50; }
        p { color: #7f8c8d; }
    </style>
</head>
<body>
    <h1>hieucute.id.vn</h1>
    <p>LAMP Stack đang hoạt động trên Ubuntu 22.04 LTS</p>
    <p>Apache | MySQL | PHP</p>
</body>
</html>
```

#### Tạo file cấu hình Virtual Host

```bash
sudo nano /etc/apache2/sites-available/hieucute.id.vn.conf
```

Nội dung chuẩn:
```apache
<VirtualHost *:80>
    ServerAdmin admin@hieucute.id.vn
    ServerName hieucute.id.vn
    ServerAlias www.hieucute.id.vn
    DocumentRoot /var/www/hieucute.id.vn/public_html

    # Ghi log riêng cho từng domain
    ErrorLog /var/log/apache2/hieucute.id.vn_error.log
    CustomLog /var/log/apache2/hieucute.id.vn_access.log combined

    # Cho phép .htaccess override (cần cho WordPress)
    <Directory /var/www/hieucute.id.vn/public_html>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
```

#### Kích hoạt Virtual Host

```bash
# Bật site mới
sudo a2ensite hieucute.id.vn.conf

# Tắt site mặc định
sudo a2dissite 000-default.conf

# Kiểm tra cú pháp cấu hình
sudo apache2ctl configtest
# Kết quả đúng: Syntax OK

# Reload Apache áp dụng thay đổi
sudo systemctl reload apache2
```

> 💡 Dùng `reload` thay vì `restart` trong production — `reload` không ngắt kết nối đang xử lý, `restart` ngắt hết.

---

### 3.3 Cài đặt MySQL Server

```bash
sudo apt install -y mysql-server
```

Khởi động và bật auto-start:
```bash
sudo systemctl start mysql
sudo systemctl enable mysql
```

Kiểm tra trạng thái:
```bash
sudo systemctl status mysql
```

> 💡 MySQL 8.0 trên Ubuntu 22.04 mặc định bật `validate_password` plugin. Mật khẩu phải đạt yêu cầu độ phức tạp nhất định.

---

### 3.4 Cài đặt PHP

Ubuntu 22.04 mặc định cung cấp PHP 8.1. Đây là phiên bản ổn định, được WordPress, Laravel, và hầu hết framework hiện đại hỗ trợ.

```bash
# Cài PHP và module cần thiết cho Apache (mod_php)
sudo apt install -y php libapache2-mod-php

# Cài các extension PHP phổ biến
sudo apt install -y php-{common,mysql,xml,xmlrpc,curl,gd,imagick,cli,dev,imap,mbstring,opcache,soap,zip,intl,bcmath,json}
```

Kiểm tra phiên bản:
```bash
php -v
```

Kết quả:
```
PHP 8.1.x (cli) (built: ...)
```

---

### 3.5 Kết hợp PHP với Apache

`libapache2-mod-php` đã tự động cài và enable module PHP cho Apache. Kiểm tra lại:

```bash
sudo a2enmod php8.1
sudo systemctl restart apache2
```

Cấu hình PHP ưu tiên đọc `index.php` trước `index.html`:

```bash
sudo nano /etc/apache2/mods-enabled/dir.conf
```

Sửa dòng DirectoryIndex:
```apache
DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm
```

```bash
sudo systemctl reload apache2
```

---

### 3.6 Kiểm tra LAMP hoàn chỉnh

Tạo file phpinfo để kiểm tra PHP hoạt động với Apache:

```bash
sudo nano /var/www/hieucute.id.vn/public_html/info.php
```

Nội dung:
```php
<?php
phpinfo();
?>
```

Truy cập trình duyệt:
```
http://hieucute.id.vn/info.php
```

Kết quả đúng: Trang PHP Info hiển thị đầy đủ thông tin PHP, Apache, Extensions.

> ⚠️ **Bắt buộc xóa sau khi kiểm tra xong** — file `info.php` tiết lộ thông tin cấu hình server, là mục tiêu của hacker:
> ```bash
> sudo rm /var/www/hieucute.id.vn/public_html/info.php
> ```

---

## 4. Triển khai LEMP Stack

> ⚠️ Không nên cài cả Apache và Nginx trên cùng một server — cả hai đều cạnh tranh port 80/443. Chọn một trong hai. Nếu đã cài LAMP, hãy dừng/gỡ Apache trước khi cài Nginx.

```bash
# Dừng Apache nếu đã cài
sudo systemctl stop apache2
sudo systemctl disable apache2
```

---

### 4.1 Cài đặt Nginx

```bash
sudo apt install -y nginx
```

Khởi động và bật auto-start:
```bash
sudo systemctl start nginx
sudo systemctl enable nginx
```

Kiểm tra trạng thái:
```bash
sudo systemctl status nginx
```

Kiểm tra Nginx hoạt động:
```bash
curl -I http://localhost
# Kết quả đúng: Server: nginx/1.18.x
```

---

### 4.2 Cài đặt PHP-FPM

Nginx không xử lý PHP trực tiếp — cần **PHP-FPM** (FastCGI Process Manager) làm cầu nối.

```bash
# Cài PHP-FPM
sudo apt install -y php8.1-fpm

# Cài các extension PHP
sudo apt install -y php8.1-{common,mysql,xml,xmlrpc,curl,gd,imagick,cli,dev,imap,mbstring,opcache,soap,zip,intl,bcmath,json}
```

Khởi động PHP-FPM:
```bash
sudo systemctl start php8.1-fpm
sudo systemctl enable php8.1-fpm
sudo systemctl status php8.1-fpm
```

Kiểm tra PHP-FPM socket tồn tại:
```bash
ls /run/php/php8.1-fpm.sock
# Kết quả đúng: /run/php/php8.1-fpm.sock
```

---

### 4.3 Cấu hình Virtual Host Nginx

#### Tạo cấu trúc thư mục

```bash
sudo mkdir -p /var/www/hieucute.id.vn/public_html
sudo chown -R $USER:$USER /var/www/hieucute.id.vn
sudo chmod -R 755 /var/www/hieucute.id.vn
```

Tạo file index thử nghiệm:
```bash
nano /var/www/hieucute.id.vn/public_html/index.html
```

```html
<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <title>hieucute.id.vn - LEMP Stack</title>
    <style>
        body { font-family: sans-serif; text-align: center; margin-top: 100px; }
        h1 { color: #2c3e50; }
    </style>
</head>
<body>
    <h1>hieucute.id.vn</h1>
    <p>LEMP Stack đang hoạt động trên Ubuntu 22.04 LTS</p>
    <p>Nginx | MySQL | PHP-FPM</p>
</body>
</html>
```

#### Tạo file cấu hình Virtual Host

```bash
sudo nano /etc/nginx/sites-available/hieucute.id.vn
```

Nội dung chuẩn production:
```nginx
server {
    listen 80;
    listen [::]:80;
    server_name hieucute.id.vn www.hieucute.id.vn;
    root /var/www/hieucute.id.vn/public_html;

    # Trang mặc định — ưu tiên index.php
    index index.php index.html index.htm;

    # Ghi log riêng cho domain
    access_log /var/log/nginx/hieucute.id.vn_access.log;
    error_log  /var/log/nginx/hieucute.id.vn_error.log;

    # Xử lý static file và rewrite
    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    # Chuyển file .php sang PHP-FPM qua Unix socket
    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.1-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }

    # Chặn truy cập file .htaccess (bảo mật)
    location ~ /\.ht {
        deny all;
    }

    # Chặn truy cập file ẩn
    location ~ /\. {
        deny all;
        access_log off;
        log_not_found off;
    }
}
```

---

### 4.4 Kết hợp Nginx với PHP-FPM

#### Kích hoạt Virtual Host

```bash
# Tạo symlink kích hoạt site
sudo ln -s /etc/nginx/sites-available/hieucute.id.vn /etc/nginx/sites-enabled/

# Tắt site mặc định
sudo unlink /etc/nginx/sites-enabled/default

# Kiểm tra cú pháp cấu hình
sudo nginx -t
# Kết quả đúng:
# nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
# nginx: configuration file /etc/nginx/nginx.conf test is successful

# Reload Nginx
sudo systemctl reload nginx
```

> 💡 Dùng `reload` thay vì `restart` để không ngắt kết nối đang phục vụ.

---

### 4.5 Kiểm tra LEMP hoàn chỉnh

```bash
sudo nano /var/www/hieucute.id.vn/public_html/info.php
```

```php
<?php
phpinfo();
?>
```

Truy cập:
```
http://hieucute.id.vn/info.php
```

Kết quả đúng: Server API hiển thị **FPM/FastCGI** (không phải Apache 2.0 Handler như LAMP).

> ⚠️ **Xóa ngay sau kiểm tra:**
> ```bash
> sudo rm /var/www/hieucute.id.vn/public_html/info.php
> ```

---

## 5. Cài đặt và cấu hình MySQL (dùng chung)

### 5.1 Bảo mật MySQL

Chạy script bảo mật tích hợp sẵn của MySQL:

```bash
sudo mysql_secure_installation
```

Hệ thống sẽ hỏi lần lượt — trả lời như sau:

| Câu hỏi | Khuyến nghị |
|---------|------------|
| VALIDATE PASSWORD component? | `Y` — bật kiểm tra độ mạnh mật khẩu |
| Password validation policy | `2` — STRONG (chữ hoa + chữ thường + số + ký tự đặc biệt) |
| Remove anonymous users? | `Y` |
| Disallow root login remotely? | `Y` — root chỉ đăng nhập qua localhost |
| Remove test database? | `Y` |
| Reload privilege tables? | `Y` |

Kiểm tra đăng nhập MySQL:
```bash
sudo mysql -u root -p
```

---

### 5.2 Tạo Database và User cho Website

Thực hành tốt nhất trong production: **không bao giờ dùng tài khoản root MySQL cho ứng dụng web**. Tạo user riêng với quyền giới hạn.

```sql
-- Đăng nhập MySQL
sudo mysql -u root -p

-- Tạo database cho website
CREATE DATABASE hieucute_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- Tạo user riêng cho ứng dụng
CREATE USER 'hieucute_user'@'localhost' IDENTIFIED BY 'StrongPassword@2024';

-- Cấp quyền chỉ trên database đó
GRANT ALL PRIVILEGES ON hieucute_db.* TO 'hieucute_user'@'localhost';

-- Áp dụng thay đổi quyền
FLUSH PRIVILEGES;

-- Kiểm tra
SHOW DATABASES;
SELECT User, Host FROM mysql.user;

-- Thoát
EXIT;
```

Kiểm tra kết nối với user mới:
```bash
mysql -u hieucute_user -p hieucute_db
```

---

## 6. Cài đặt SSL (Let's Encrypt) với Certbot

### 6.1 Cài đặt Certbot

```bash
sudo apt install -y certbot
```

Cài plugin tương ứng với web server:
```bash
# Cho Apache
sudo apt install -y python3-certbot-apache

# Cho Nginx
sudo apt install -y python3-certbot-nginx
```

> ⚠️ **Bắt buộc:** DNS của `hieucute.id.vn` phải đã trỏ về `103.159.51.228` và port 80 phải mở trước khi cấp SSL.

---

### 6.2 Cấp SSL cho Apache

```bash
sudo certbot --apache -d hieucute.id.vn -d www.hieucute.id.vn
```

Certbot sẽ:
1. Xác minh domain qua HTTP challenge (tạo file tạm trong `/.well-known/acme-challenge/`).
2. Cấp chứng chỉ và lưu tại `/etc/letsencrypt/live/hieucute.id.vn/`.
3. Tự động sửa Virtual Host Apache để bật HTTPS và redirect.

---

### 6.3 Cấp SSL cho Nginx

```bash
sudo certbot --nginx -d hieucute.id.vn -d www.hieucute.id.vn
```

Certbot tự động thêm cấu hình SSL vào file Virtual Host Nginx và tạo server block redirect HTTP → HTTPS.

---

### 6.4 Tự động gia hạn SSL

Certbot cài sẵn systemd timer tự động gia hạn. Kiểm tra:

```bash
sudo systemctl status certbot.timer
```

Kiểm tra thủ công (không thực sự gia hạn):
```bash
sudo certbot renew --dry-run
```

Kết quả đúng:
```
Congratulations, all simulated renewals succeeded
```

Cấu hình cron thủ công (nếu cần):
```bash
sudo crontab -e
# Thêm dòng này:
0 3 * * * /usr/bin/certbot renew --quiet --post-hook "systemctl reload apache2"
# Với Nginx:
0 3 * * * /usr/bin/certbot renew --quiet --post-hook "systemctl reload nginx"
```

---

## 7. Tối ưu hóa hiệu năng

### 7.1 Bật Gzip Compression

**Apache:**
```bash
sudo a2enmod deflate
sudo nano /etc/apache2/sites-available/hieucute.id.vn.conf
```

Thêm vào bên trong `<VirtualHost>`:
```apache
<IfModule mod_deflate.c>
    AddOutputFilterByType DEFLATE text/html text/plain text/xml
    AddOutputFilterByType DEFLATE text/css text/javascript
    AddOutputFilterByType DEFLATE application/javascript application/json
</IfModule>
```

**Nginx:**
```bash
sudo nano /etc/nginx/nginx.conf
```

Trong block `http {}`, thêm hoặc bỏ comment:
```nginx
gzip on;
gzip_vary on;
gzip_min_length 1024;
gzip_types text/plain text/css application/json application/javascript text/xml application/xml;
```

---

### 7.2 Bật Browser Caching

**Apache** — thêm vào `.htaccess` hoặc Virtual Host:
```apache
<IfModule mod_expires.c>
    ExpiresActive On
    ExpiresByType image/jpg "access plus 1 year"
    ExpiresByType image/jpeg "access plus 1 year"
    ExpiresByType image/png "access plus 1 year"
    ExpiresByType text/css "access plus 1 month"
    ExpiresByType application/javascript "access plus 1 month"
</IfModule>
```

**Nginx** — thêm vào server block:
```nginx
location ~* \.(jpg|jpeg|png|gif|ico|css|js|woff2)$ {
    expires 1y;
    add_header Cache-Control "public, immutable";
}
```

---

### 7.3 Tối ưu PHP-FPM

```bash
sudo nano /etc/php/8.1/fpm/pool.d/www.conf
```

Các thông số quan trọng cần điều chỉnh theo RAM server:

| Thông số | Giá trị (server 2GB RAM) | Ý nghĩa |
|----------|--------------------------|---------|
| `pm` | `dynamic` | Quản lý process động |
| `pm.max_children` | `20` | Số process PHP tối đa |
| `pm.start_servers` | `5` | Số process khởi động ban đầu |
| `pm.min_spare_servers` | `3` | Số process rảnh tối thiểu |
| `pm.max_spare_servers` | `10` | Số process rảnh tối đa |
| `pm.max_requests` | `500` | Request tối đa mỗi process trước khi restart |

```bash
sudo systemctl restart php8.1-fpm
```

---

## 8. Bash Script tự động cài đặt

### 8.1 Script cài đặt LAMP

```bash
touch lamp_install.sh && chmod +x lamp_install.sh
nano lamp_install.sh
```

```bash
#!/bin/bash
# ==============================================
# Script cài đặt LAMP Stack trên Ubuntu 22.04
# Domain: hieucute.id.vn
# Tác giả: iamhieu
# Ngày: 2024
# ==============================================

set -e  # Dừng script nếu có lệnh lỗi

# Màu sắc output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m'

DOMAIN="hieucute.id.vn"
WEBROOT="/var/www/${DOMAIN}/public_html"
APACHE_CONF="/etc/apache2/sites-available/${DOMAIN}.conf"

echo -e "${GREEN}[1/6] Cập nhật hệ thống...${NC}"
apt update && apt upgrade -y

echo -e "${GREEN}[2/6] Cài đặt Apache...${NC}"
apt install -y apache2
systemctl start apache2
systemctl enable apache2
a2enmod rewrite headers ssl
systemctl reload apache2

echo -e "${GREEN}[3/6] Cài đặt MySQL...${NC}"
apt install -y mysql-server
systemctl start mysql
systemctl enable mysql

echo -e "${GREEN}[4/6] Cài đặt PHP 8.1...${NC}"
apt install -y php libapache2-mod-php
apt install -y php-{common,mysql,xml,xmlrpc,curl,gd,imagick,cli,dev,imap,mbstring,opcache,soap,zip,intl,bcmath}

echo -e "${GREEN}[5/6] Cấu hình Virtual Host...${NC}"
mkdir -p "${WEBROOT}"
chown -R www-data:www-data "/var/www/${DOMAIN}"
chmod -R 755 "/var/www/${DOMAIN}"

cat > "${APACHE_CONF}" <<EOF
<VirtualHost *:80>
    ServerAdmin admin@${DOMAIN}
    ServerName ${DOMAIN}
    ServerAlias www.${DOMAIN}
    DocumentRoot ${WEBROOT}
    ErrorLog /var/log/apache2/${DOMAIN}_error.log
    CustomLog /var/log/apache2/${DOMAIN}_access.log combined
    <Directory ${WEBROOT}>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
EOF

a2ensite "${DOMAIN}.conf"
a2dissite 000-default.conf

cat > "${WEBROOT}/index.html" <<EOF
<!DOCTYPE html>
<html lang="vi">
<head><meta charset="UTF-8"><title>${DOMAIN} - LAMP Stack</title></head>
<body><h1>${DOMAIN} - LAMP Stack hoạt động!</h1><p>Apache | MySQL | PHP 8.1</p></body>
</html>
EOF

apache2ctl configtest && systemctl reload apache2

echo -e "${GREEN}[6/6] Cấu hình UFW Firewall...${NC}"
ufw allow OpenSSH
ufw allow 80/tcp
ufw allow 443/tcp
ufw --force enable

echo -e "${GREEN}=============================${NC}"
echo -e "${GREEN}LAMP Stack đã cài xong!${NC}"
echo -e "${GREEN}Website: http://${DOMAIN}${NC}"
echo -e "${GREEN}PHP version: $(php -r 'echo PHP_VERSION;')${NC}"
echo -e "${YELLOW}Bước tiếp theo: Chạy mysql_secure_installation${NC}"
echo -e "${YELLOW}                Cài SSL: certbot --apache -d ${DOMAIN}${NC}"
echo -e "${GREEN}=============================${NC}"
```

Chạy script:
```bash
sudo bash lamp_install.sh
```

---

### 8.2 Script cài đặt LEMP

```bash
touch lemp_install.sh && chmod +x lemp_install.sh
nano lemp_install.sh
```

```bash
#!/bin/bash
# ==============================================
# Script cài đặt LEMP Stack trên Ubuntu 22.04
# Domain: hieucute.id.vn
# Tác giả: iamhieu
# Ngày: 2024
# ==============================================

set -e

RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m'

DOMAIN="hieucute.id.vn"
WEBROOT="/var/www/${DOMAIN}/public_html"
NGINX_CONF="/etc/nginx/sites-available/${DOMAIN}"
PHP_SOCK="/run/php/php8.1-fpm.sock"

echo -e "${GREEN}[1/6] Cập nhật hệ thống...${NC}"
apt update && apt upgrade -y

echo -e "${GREEN}[2/6] Cài đặt Nginx...${NC}"
apt install -y nginx
systemctl start nginx
systemctl enable nginx

echo -e "${GREEN}[3/6] Cài đặt PHP 8.1-FPM...${NC}"
apt install -y php8.1-fpm
apt install -y php8.1-{common,mysql,xml,xmlrpc,curl,gd,imagick,cli,dev,imap,mbstring,opcache,soap,zip,intl,bcmath}
systemctl start php8.1-fpm
systemctl enable php8.1-fpm

echo -e "${GREEN}[4/6] Cài đặt MySQL...${NC}"
apt install -y mysql-server
systemctl start mysql
systemctl enable mysql

echo -e "${GREEN}[5/6] Cấu hình Virtual Host Nginx...${NC}"
mkdir -p "${WEBROOT}"
chown -R www-data:www-data "/var/www/${DOMAIN}"
chmod -R 755 "/var/www/${DOMAIN}"

cat > "${NGINX_CONF}" <<EOF
server {
    listen 80;
    listen [::]:80;
    server_name ${DOMAIN} www.${DOMAIN};
    root ${WEBROOT};
    index index.php index.html index.htm;
    access_log /var/log/nginx/${DOMAIN}_access.log;
    error_log  /var/log/nginx/${DOMAIN}_error.log;

    location / {
        try_files \$uri \$uri/ /index.php?\$query_string;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:${PHP_SOCK};
        fastcgi_param SCRIPT_FILENAME \$realpath_root\$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.ht { deny all; }
    location ~ /\. { deny all; access_log off; log_not_found off; }
}
EOF

ln -sf "${NGINX_CONF}" /etc/nginx/sites-enabled/
[ -L /etc/nginx/sites-enabled/default ] && unlink /etc/nginx/sites-enabled/default

cat > "${WEBROOT}/index.html" <<EOF
<!DOCTYPE html>
<html lang="vi">
<head><meta charset="UTF-8"><title>${DOMAIN} - LEMP Stack</title></head>
<body><h1>${DOMAIN} - LEMP Stack hoạt động!</h1><p>Nginx | MySQL | PHP-FPM 8.1</p></body>
</html>
EOF

nginx -t && systemctl reload nginx

echo -e "${GREEN}[6/6] Cấu hình UFW Firewall...${NC}"
ufw allow OpenSSH
ufw allow 80/tcp
ufw allow 443/tcp
ufw --force enable

echo -e "${GREEN}=============================${NC}"
echo -e "${GREEN}LEMP Stack đã cài xong!${NC}"
echo -e "${GREEN}Website: http://${DOMAIN}${NC}"
echo -e "${GREEN}PHP version: $(php -r 'echo PHP_VERSION;')${NC}"
echo -e "${YELLOW}Bước tiếp theo: Chạy mysql_secure_installation${NC}"
echo -e "${YELLOW}                Cài SSL: certbot --nginx -d ${DOMAIN}${NC}"
echo -e "${GREEN}=============================${NC}"
```

Chạy script:
```bash
sudo bash lemp_install.sh
```

---

## 9. Lỗi thường gặp và cách xử lý

| Lỗi | Nguyên nhân | Cách xử lý |
|-----|------------|------------|
| **403 Forbidden** | Sai quyền thư mục hoặc chủ sở hữu | `chmod 755` thư mục, `chown www-data:www-data` |
| **404 Not Found** | DocumentRoot sai hoặc file không tồn tại | Kiểm tra đường dẫn trong Virtual Host |
| **500 Internal Server Error** | Lỗi PHP hoặc `.htaccess` sai cú pháp | Xem log: `tail -f /var/log/apache2/error.log` |
| **502 Bad Gateway** (Nginx) | PHP-FPM chưa chạy hoặc socket sai | `systemctl status php8.1-fpm`, kiểm tra đường dẫn socket |
| **PHP không xử lý** (Nginx) | Thiếu cấu hình `fastcgi_pass` | Kiểm tra block `location ~ \.php$` trong Nginx conf |
| **MySQL: Access Denied** | Sai user/password hoặc chưa GRANT | Kiểm tra lại thông tin, chạy `FLUSH PRIVILEGES` |
| **SSL: domain not found** | DNS chưa propagate hoặc port 80 bị block | `dig A hieucute.id.vn`, kiểm tra UFW |
| **Nginx: Address already in use** | Port 80 đang bị Apache chiếm | `sudo systemctl stop apache2` trước khi start Nginx |

**Xem log nhanh:**
```bash
# Apache
sudo tail -f /var/log/apache2/hieucute.id.vn_error.log
sudo tail -f /var/log/apache2/error.log

# Nginx
sudo tail -f /var/log/nginx/hieucute.id.vn_error.log
sudo tail -f /var/log/nginx/error.log

# PHP-FPM
sudo tail -f /var/log/php8.1-fpm.log

# MySQL
sudo tail -f /var/log/mysql/error.log
```

---

## 10. Bảng tóm tắt lệnh quan trọng

### Quản lý dịch vụ

| Lệnh | Mô tả |
|------|-------|
| `sudo systemctl start\|stop\|restart\|reload\|status apache2` | Quản lý Apache |
| `sudo systemctl start\|stop\|restart\|reload\|status nginx` | Quản lý Nginx |
| `sudo systemctl start\|stop\|restart\|status mysql` | Quản lý MySQL |
| `sudo systemctl start\|stop\|restart\|status php8.1-fpm` | Quản lý PHP-FPM |
| `sudo apache2ctl configtest` | Kiểm tra cú pháp cấu hình Apache |
| `sudo nginx -t` | Kiểm tra cú pháp cấu hình Nginx |

### Apache Virtual Host

| Lệnh | Mô tả |
|------|-------|
| `sudo a2ensite <file>.conf` | Bật site |
| `sudo a2dissite <file>.conf` | Tắt site |
| `sudo a2enmod <module>` | Bật module |
| `sudo a2dismod <module>` | Tắt module |
| `ls /etc/apache2/sites-enabled/` | Xem site đang bật |

### Nginx Virtual Host

| Lệnh | Mô tả |
|------|-------|
| `sudo ln -s /etc/nginx/sites-available/<domain> /etc/nginx/sites-enabled/` | Bật site |
| `sudo unlink /etc/nginx/sites-enabled/<domain>` | Tắt site |
| `ls /etc/nginx/sites-enabled/` | Xem site đang bật |

### MySQL

| Lệnh | Mô tả |
|------|-------|
| `sudo mysql -u root -p` | Đăng nhập MySQL |
| `SHOW DATABASES;` | Liệt kê database |
| `SHOW GRANTS FOR 'user'@'localhost';` | Xem quyền user |
| `mysqldump -u user -p database > backup.sql` | Backup database |
| `mysql -u user -p database < backup.sql` | Restore database |

### Certbot SSL

| Lệnh | Mô tả |
|------|-------|
| `sudo certbot --apache -d hieucute.id.vn` | Cấp SSL cho Apache |
| `sudo certbot --nginx -d hieucute.id.vn` | Cấp SSL cho Nginx |
| `sudo certbot renew --dry-run` | Kiểm tra tự động gia hạn |
| `sudo certbot certificates` | Liệt kê cert đang có |

---

## References

1. [LAMP Stack – AWS Documentation](https://aws.amazon.com/vi/what-is/lamp-stack/)
2. [LEMP Stack – GeeksforGeeks](https://www.geeksforgeeks.org/what-is-lemp-stack/)
3. [Install Apache on Ubuntu 22.04 – DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-install-the-apache-web-server-on-ubuntu-22-04)
4. [Install Nginx on Ubuntu 22.04 – DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-22-04)
5. [Install MySQL on Ubuntu 22.04 – DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-install-mysql-on-ubuntu-22-04)
6. [Let's Encrypt with Certbot – Certbot Documentation](https://certbot.eff.org/instructions)
7. [PHP-FPM Configuration – PHP Manual](https://www.php.net/manual/en/install.fpm.configuration.php)
8. [Ubuntu 22.04 Server Guide – Canonical](https://ubuntu.com/server/docs)
