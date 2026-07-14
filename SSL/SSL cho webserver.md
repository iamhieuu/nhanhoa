# VirtualHost SSL — Bản rút gọn (Nginx / Apache / Tomcat)

> Trình bày theo quy trình 4 bước cho từng loại server: **(1) Xác định path cert/key → (2) Gán vào VirtualHost → (3) Check cấu hình → (4) Restart/Reload**

---
## Mục lục
- [0. Đường dẫn cert/key theo loại SSL](#0-đường-dẫn-certkey-theo-loại-ssl)
- [1. NGINX](#1-nginx)
- [2. APACHE](#2-apache)
- [3. TOMCAT (Nginx làm SSL termination)](#3-tomcat-nginx-làm-ssl-termination)
- [4. NGINX → APACHE reverse proxy](#4-nginx--apache-reverse-proxy)

---

## 0. Đường dẫn cert/key SSL

| Loại SSL | Cert lấy từ đâu | Vị trí file thường đặt | File cần có |
|---|---|---|---|
| **Trả phí** (Comodo/Sectigo) | file (certificate + ca) qua mail/ticket | Tự tạo thư mục, ví dụ `/etc/ssl/DOMAIN/` | `certificate` + `ca` → gộp thành `fullchain.crt` + `private.key`  |
| **Miễn phí** (Let's Encrypt qua Certbot) | Certbot tự lấy và tự gia hạn | `/etc/letsencrypt/live/DOMAIN/` | `fullchain.pem` (=certificate+ca gộp sẵn) + `privkey.pem` |

**Ghi chú:**
- SSL trả phí: certificate và ca là 2 file riêng lúc nhận về, phải tự gộp lại thành `fullchain.crt` (lệnh gộp có ở Bước 1 của mục Nginx). Key phải đúng file lúc tạo CSR.
- Let's Encrypt: đã gộp sẵn `fullchain.pem`, không cần gộp tay.
- Log: chỉ cần đổi `DOMAIN` trong tên file log (`access_log`/`error_log` ở Nginx, `CustomLog`/`ErrorLog` ở Apache, `prefix` trong `AccessLogValve` ở Tomcat) — không cần đổi thư mục gốc.

---

## 1. NGINX

**File cấu hình:** `/etc/nginx/conf.d/DOMAIN.conf`  

Ta có thể vào virtual host để lấy path hoặc đơn giản là tạo file rồi gán vào    

### Bước 1 — Xác định path cert/key 

```bash
# SSL trả phí (Comodo/Sectigo): gộp certificate + ca trước
cat CERT_PATH/certificate.crt CERT_PATH/ca.crt > CERT_PATH/fullchain.crt
chmod 600 CERT_PATH/private.key
# → CERT_PATH ví dụ: /etc/ssl/DOMAIN

# Let's Encrypt: không cần gộp, dùng thẳng
# → CERT_PATH = /etc/letsencrypt/live/DOMAIN
```

### Bước 2 — Gán vào VirtualHost

```nginx
server {
    listen 80;
    server_name DOMAIN www.DOMAIN;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    http2 on;
    server_name DOMAIN www.DOMAIN;

    # Gán SSL — path lấy từ Bước 1
    ssl_certificate     CERT_PATH/fullchain.crt;
    ssl_certificate_key CERT_PATH/private.key;
    ssl_protocols TLSv1.2 TLSv1.3;

    root  /home/USER/public_html;
    index index.php index.html;

    access_log /var/log/nginx/DOMAIN-access.log;
    error_log  /var/log/nginx/DOMAIN-error.log;

    location ~ \.php$ {
        fastcgi_pass   unix:/var/run/USER-fpm.sock;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }
}
```

### Bước 3 — Check cấu hình

```bash
nginx -t
```
Kết quả phải thấy `syntax is ok` và `test is successful` 

### Bước 4 — Restart/Reload

```bash
systemctl reload nginx
```

---

## 2. APACHE

**File cấu hình:** `/etc/httpd/conf.d/DOMAIN.conf` (CentOS) hoặc `/etc/apache2/sites-available/DOMAIN.conf` (Ubuntu)

### Bước 1 — Xác định path cert/key (CERT_PATH)

```bash
# SSL trả phí: dùng chung file fullchain.crt đã gộp ở phần Nginx (nếu cùng server)
# Let's Encrypt: CERT_PATH = /etc/letsencrypt/live/DOMAIN
```
Apache bản mới dùng chung 1 file `fullchain.crt` cho cả certificate + ca, không cần tách riêng.

### Bước 2 — Gán vào VirtualHost

```apache
<VirtualHost *:80>
    ServerName DOMAIN
    ServerAlias www.DOMAIN
    RewriteEngine On
    RewriteRule ^(.*)$ https://%{HTTP_HOST}$1 [R=301,L]
</VirtualHost>

<VirtualHost *:443>
    ServerName DOMAIN
    ServerAlias www.DOMAIN
    DocumentRoot /home/USER/public_html

    ErrorLog  /var/log/httpd/DOMAIN-error.log
    CustomLog /var/log/httpd/DOMAIN-access.log combined

    # ── Gán SSL — path lấy từ Bước 1 ──────────────────────────
    SSLEngine on
    SSLCertificateFile    CERT_PATH/fullchain.crt
    SSLCertificateKeyFile CERT_PATH/private.key

    <FilesMatch \.php$>
        SetHandler "proxy:unix:/var/run/USER-fpm.sock|fcgi://localhost"
    </FilesMatch>
</VirtualHost>
```

### Bước 3 — Check cấu hình

```bash
apachectl configtest
```
Kết quả mong muốn: `Syntax OK`.

### Bước 4 — Restart/Reload

```bash
systemctl reload httpd        # CentOS
# hoặc
a2ensite DOMAIN.conf && systemctl reload apache2   # Ubuntu 
```

---

## 3. TOMCAT (Nginx làm SSL termination)

> Tomcat không tự cầm cert — Nginx nhận SSL ở 443 rồi forward HTTP nội bộ vào Tomcat. Cert vẫn gán ở phía Nginx như mục 1.

### Bước 1 — Xác định path cert/key
Giống hệt Bước 1 của mục Nginx (mục 1) — dùng chung `CERT_PATH`.

### Bước 2 — Gán vào VirtualHost

**Nginx** (`/etc/nginx/conf.d/DOMAIN.conf`):
```nginx
server {
    listen 443 ssl;
    server_name DOMAIN;

    ssl_certificate     CERT_PATH/fullchain.crt;
    ssl_certificate_key CERT_PATH/private.key;

    access_log /var/log/nginx/DOMAIN-access.log;
    error_log  /var/log/nginx/DOMAIN-error.log;

    location / {
        proxy_pass http://127.0.0.1:APP_PORT;
        proxy_set_header Host              $host;
        proxy_set_header X-Real-IP         $remote_addr;
        proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

**Tomcat** (`/opt/tomcat/conf/server.xml`) — chỉ nghe nội bộ, không cần cert:
```xml
<Connector port="APP_PORT" protocol="HTTP/1.1" address="127.0.0.1"
           scheme="https" secure="true" proxyName="DOMAIN" proxyPort="443" />

<Host name="localhost" appBase="webapps" unpackWARs="true" autoDeploy="true">
    <Valve className="org.apache.catalina.valves.AccessLogValve"
           directory="logs" prefix="DOMAIN_access_log" suffix=".log"
           pattern="%h %l %u %t &quot;%r&quot; %s %b" />
</Host>
```

### Bước 3 — Check cấu hình

```bash
nginx -t
```

### Bước 4 — Restart/Reload

```bash
systemctl reload nginx
systemctl restart tomcat
tail -f /opt/tomcat/logs/catalina.out    # theo dõi log nếu start lỗi
```

---

## 4. NGINX → APACHE (reverse proxy)

> Apache có sẵn, thêm Nginx đứng trước xử lý SSL. Apache chuyển sang port 8080 và không cầm cert nữa.

### Bước 1 — Xác định path cert/key
Giống Bước 1 của mục Nginx,cert chỉ gán ở Nginx, Apache không cần path cert.

### Bước 2 — Gán vào VirtualHost

**Đổi Apache sang port 8080 trước:** sửa `Listen 80` → `Listen 8080` trong `/etc/httpd/conf/httpd.conf` (hoặc `ports.conf` Ubuntu), và `<VirtualHost *:80>` → `<VirtualHost 127.0.0.1:8080>`.

**Nginx** (`/etc/nginx/conf.d/DOMAIN.conf`):
```nginx
server {
    listen 443 ssl;
    server_name DOMAIN;

    ssl_certificate     CERT_PATH/fullchain.crt;
    ssl_certificate_key CERT_PATH/private.key;

    access_log /var/log/nginx/DOMAIN-access.log;
    error_log  /var/log/nginx/DOMAIN-error.log;

    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_set_header Host              $host;
        proxy_set_header X-Real-IP         $remote_addr;
        proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

**Apache** (`/etc/httpd/conf.d/DOMAIN.conf`):
```apache
<VirtualHost 127.0.0.1:8080>
    ServerName DOMAIN
    DocumentRoot /home/USER/public_html
    ErrorLog  /var/log/httpd/DOMAIN-error.log
    CustomLog /var/log/httpd/DOMAIN-access.log combined
</VirtualHost>
```

> `.htaccess` có rule force HTTPS → redirect loop (Nginx gửi HTTP xuống Apache → Apache redirect lại HTTPS). Fix: thêm điều kiện check `X-Forwarded-Proto` trong `.htaccess` trước khi redirect.

### Bước 3 — Check cấu hình

```bash
nginx -t
apachectl configtest
```

### Bước 4 — Restart/Reload

```bash
systemctl reload nginx
systemctl reload httpd     # hoặc: systemctl reload apache2
```

---

## Tổng kết nhanh — CERT_PATH theo từng loại

| | SSL trả phí (Comodo/Sectigo) | Let's Encrypt |
|---|---|---|
| Đường dẫn | tự đặt, vd `/etc/ssl/DOMAIN/` | `/etc/letsencrypt/live/DOMAIN/` |
| Certificate (dùng chung cho Nginx + Apache) | `fullchain.crt` — **phải tự gộp** `certificate.crt + ca.crt` | `fullchain.pem` — có sẵn |
| Key | `private.key` — key lúc tạo CSR | `privkey.pem` |
