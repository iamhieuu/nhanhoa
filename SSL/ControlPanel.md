## 7. DIRECTADMIN — Cài SSL

> DA quản lý cert theo từng user/domain. Có 2 luồng: Let's Encrypt (1 click) và SSL trả phí (paste thủ công).

### 7a. Let's Encrypt (miễn phí, tự động)

```
User Level
→ Advanced Features
→ SSL Certificates
→ ● Free & automatic certificate from Let's Encrypt
→ Tích: DOMAIN + www.DOMAIN
→ Save
```

DA tự động:
- Tạo CSR + private key
- Chạy ACME challenge (HTTP-01 qua port 80)
- Nhận cert → ghi vào Apache/Nginx config
- Cron tự renew trước 30 ngày hết hạn

**Điều kiện:** DNS bản ghi A của DOMAIN phải trỏ đúng về IP server trước.

---

### 7b. SSL Trả phí — Paste thủ công

**Bước 1: Tạo CSR**
```
User Level → Advanced Features → SSL Certificates
→ ● Generate a Certificate Request (CSR)
→ Điền: Key Size=2048, Common Name=DOMAIN, Country=VN, ...
→ Save
```
Copy CSR → gửi nhà cung cấp SSL.

**Bước 2: Paste cert nhận về**
```
→ ● Paste a pre-generated certificate and key
```

```
[Certificate] box: paste nội dung public.crt
-----BEGIN CERTIFICATE-----
...
-----END CERTIFICATE-----

[Key] box: paste private.key (DA đã lưu sẵn từ lúc tạo CSR)
-----BEGIN PRIVATE KEY-----
...
-----END PRIVATE KEY-----
```
→ Save

**Bước 3: Paste CA Bundle**
```
→ Click "Save" xong kéo xuống
→ [CA Root Certificate] box: paste nội dung ca.crt
→ Save
```

**Bước 4: Force HTTPS**
```
User Level → Domain Setup → chọn DOMAIN
→  Force SSL with https redirect
→ Save
```

**Kiểm tra qua SSH:**
```bash
# Xem cert DA đang dùng cho domain
cat /usr/local/directadmin/data/users/USER/domains/DOMAIN.conf | grep ssl

# Cert lưu ở đây
ls /usr/local/directadmin/data/users/USER/domains/
# DOMAIN.key   DOMAIN.cert   DOMAIN.cacert
```

---

### 7c. DA Admin — Cài SSL cho hostname server

```
Admin Level → Admin Tools → SSL Certificates
→ Paste cert cho hostname server (server.nhanhoa.com)
→ Hoặc dùng Let's Encrypt cho hostname
```

```bash
# Hoặc SSH
/usr/local/directadmin/scripts/letsencrypt.sh request_single \
  server.nhanhoa.com 4096
```

---

## 8. CPANEL — Cài SSL

### 8a. Let's Encrypt / AutoSSL (tự động)

cPanel có **AutoSSL** chạy tự động mỗi ngày — thường không cần làm gì.

Kiểm tra trạng thái:
```
WHM (root) → SSL/TLS → Manage AutoSSL
→ Xem domain nào đang có SSL, domain nào fail
→ Click "Run AutoSSL for All Users" nếu muốn force chạy ngay
```

Nếu 1 user bị fail:
```
WHM → Manage AutoSSL → chọn user → Run AutoSSL
```

**Lý do hay fail AutoSSL:**
- DNS chưa trỏ đúng về IP server
- Domain đang dùng Cloudflare proxy (cam) → tắt proxy (xám) rồi chạy lại
- Port 80 bị chặn firewall

---

### 8b. SSL Trả phí — Upload qua WHM

```
WHM (root)
→ SSL/TLS → Install an SSL Certificate on a Domain
```

```
Domain    : DOMAIN
Certificate (CRT): paste public.crt
Private Key       : paste private.key
Certificate Authority Bundle (CABUNDLE): paste ca.crt
→ Install
```

Hoặc qua cPanel user:
```
cPanel → Security → SSL/TLS
→ Manage SSL Sites
→ Browse Certificates → chọn domain
→ Paste cert + key + bundle
→ Install Certificate
```

---

### 8c. Force HTTPS trong cPanel

```
cPanel → Domains → Domains
→ Toggle "Redirect to HTTPS" = ON
```

Hoặc thêm vào `.htaccess`:
```apache
RewriteEngine On
RewriteCond %{HTTPS} off
RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]
```

**Kiểm tra cert qua SSH:**
```bash
# Cert cPanel lưu ở
ls /var/cpanel/ssl/installed/
# hoặc
/usr/local/cpanel/bin/ssl_info DOMAIN
```

---

## 9. AAPANEL — Cài SSL

> aaPanel dùng giao diện đồ họa đơn giản, phù hợp VPS cá nhân.

### 9a. Let's Encrypt (miễn phí)

```
aaPanel → Website → chọn DOMAIN → Settings
→ SSL → Let's Encrypt
→ Chọn domain + www.DOMAIN
→ Apply
```

aaPanel tự chạy Certbot → lưu cert vào `/www/server/panel/vhost/cert/DOMAIN/`.

---

### 9b. SSL Trả phí — Paste thủ công

```
aaPanel → Website → chọn DOMAIN → Settings
→ SSL → Other Certificate
```

```
[Certificate (PEM format)]: paste public.crt + ca.crt (gộp lại)
-----BEGIN CERTIFICATE-----
(nội dung public.crt)
-----END CERTIFICATE-----
-----BEGIN CERTIFICATE-----
(nội dung ca.crt)
-----END CERTIFICATE-----

[Private Key (PEM format)]: paste private.key
-----BEGIN PRIVATE KEY-----
...
-----END PRIVATE KEY-----

→ Save
```

**Force HTTPS:**
```
→ Bật toggle "Force HTTPS"
→ Save
```

**Cert lưu ở:**
```bash
ls /www/server/panel/vhost/cert/DOMAIN/
# fullchain.pem  privkey.pem
```

**Nginx config aaPanel sinh ra tại:**
```bash
cat /www/server/panel/vhost/nginx/DOMAIN.conf
```

---

### 9c. aaPanel Nginx config mẫu (sinh tự động, tham khảo)

```nginx
server {
    listen      80;
    server_name DOMAIN www.DOMAIN;
    return 301  https://$host$request_uri;
}
server {
    listen      443 ssl http2;
    server_name DOMAIN www.DOMAIN;

    ssl_certificate     /www/server/panel/vhost/cert/DOMAIN/fullchain.pem;
    ssl_certificate_key /www/server/panel/vhost/cert/DOMAIN/privkey.pem;
    ssl_protocols       TLSv1.2 TLSv1.3;

    root  /www/wwwroot/DOMAIN;
    index index.php index.html;

    location ~ \.php$ {
        fastcgi_pass   unix:/tmp/php-cgi-82.sock;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }
}
```

---

## 10. PLESK — Cài SSL

### 10a. Let's Encrypt

```
Plesk → Websites & Domains → chọn DOMAIN
→ SSL/TLS Certificates
→ Get it free (Let's Encrypt)
→ Điền email
→  Secure the domain
→  Secure www.DOMAIN
→ Get it Free
```

---

### 10b. SSL Trả phí — Upload

```
Plesk → Websites & Domains → DOMAIN
→ SSL/TLS Certificates → Add SSL/TLS Certificate
```

```
Certificate name : SSL_DOMAIN_2026
Private key      : paste private.key
Certificate      : paste public.crt
CA certificate   : paste ca.crt
→ Upload Certificate
```

Sau đó apply vào domain:
```
→ Websites & Domains → DOMAIN
→ Hosting Settings
→ SSL/TLS certificate: chọn SSL_DOMAIN_2026
→  Redirect from http to https
→ OK
```

---

### 10c. Plesk — SSL qua CLI

```bash
# List cert hiện có
plesk bin certificate --list -domain DOMAIN

# Cài cert mới
plesk bin certificate --create SSL_DOMAIN_2026 \
  -domain DOMAIN \
  -key-file  /tmp/private.key \
  -cert-file /tmp/public.crt \
  -cacert-file /tmp/ca.crt

# Apply cho domain
plesk bin domain --update DOMAIN \
  -certificate-name SSL_DOMAIN_2026

# Bật redirect HTTPS
plesk bin domain --update DOMAIN \
  -https-redirect true
```

**Cert Plesk lưu ở:**
```bash
ls /usr/local/psa/var/certificates/
```

---
