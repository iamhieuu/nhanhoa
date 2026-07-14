# SSL control panel

----

## Mục lục

- [6. DIRECTADMIN — Cài SSL](#6-directadmin--cài-ssl)
  - [6a. Let's Encrypt (miễn phí, tự động)](#6a-lets-encrypt-miễn-phí-tự-động)
  - [6b. SSL Trả phí — Paste thủ công](#6b-ssl-trả-phí--paste-thủ-công)
  - [6c. DA Admin — Cài SSL cho hostname server](#6c-da-admin--cài-ssl-cho-hostname-server)
- [7. CPANEL — Cài SSL](#7-cpanel--cài-ssl)
  - [7a. Let's Encrypt / AutoSSL (tự động)](#7a-lets-encrypt--autossl-tự-động)
  - [7b. SSL Trả phí — Upload qua WHM](#7b-ssl-trả-phí--upload-qua-whm)
  - [7c. Force HTTPS trong cPanel](#7c-force-https-trong-cpanel)
- [8. AAPANEL — Cài SSL](#8-aapanel--cài-ssl)
  - [8a. Let's Encrypt (miễn phí)](#8a-lets-encrypt-miễn-phí)
  - [8b. SSL Trả phí — Paste thủ công](#8b-ssl-trả-phí--paste-thủ-công)
  - [8c. aaPanel Nginx config mẫu (sinh tự động, tham khảo)](#8c-aapanel-nginx-config-mẫu-sinh-tự-động-tham-khảo)
- [9. PLESK — Cài SSL](#9-plesk--cài-ssl)
  - [9a. Let's Encrypt](#9a-lets-encrypt)
  - [9b. SSL Trả phí — Upload](#9b-ssl-trả-phí--upload)
  - [9c. Plesk — SSL qua CLI](#9c-plesk--ssl-qua-cli)

---


## 6. DIRECTADMIN — Cài SSL

> DA quản lý cert theo từng user/domain. Có 2 luồng: Let's Encrypt và SSL trả phí.

<img width="588" height="692" alt="image" src="https://github.com/user-attachments/assets/482b86a8-77ac-4d69-81d6-611d3a0ed1a8" />

### 6a. Let's Encrypt (miễn phí, tự động)

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
- Chạy ACME challenge 
- Nhận cert → ghi vào Apache/Nginx config
- Cron tự renew trước 30 ngày hết hạn

**Điều kiện:** DNS bản ghi A của DOMAIN phải trỏ đúng về IP server trước.

---

### 6b. SSL Trả phí — Paste thủ công

**Bước 1: Tạo CSR**
```
User Level → Advanced Features → SSL Certificates
→ ● Generate a Certificate Request (CSR)
→ Điền: Key Size=2047, Common Name=DOMAIN, Country=VN, ...
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

### 6c. DA Admin — Cài SSL cho hostname server

```
Admin Level → Admin Tools → SSL Certificates
→ Paste cert cho hostname server (server.nhanhoa.com)
→ Hoặc dùng Let's Encrypt cho hostname
```

```bash
# Hoặc SSH
/usr/local/directadmin/scripts/letsencrypt.sh request_single \
  server.nhanhoa.com 4086
```

---

## 7. CPANEL — Cài SSL

### 7a. Let's Encrypt / AutoSSL (tự động)

cPanel có **AutoSSL** chạy tự động mỗi ngày — thường không cần làm gì.

Kiểm tra trạng thái:
```
WHM (root) → SSL/TLS → Manage AutoSSL
→ Xem domain nào đang có SSL, domain nào fail
→ Click "Run AutoSSL for All Users" nếu muốn force chạy ngay
```
<img width="1864" height="924" alt="image" src="https://github.com/user-attachments/assets/cde83f0a-85b4-4bcf-9ea1-6bb093093252" />

Nếu 1 user bị fail:
```
WHM → Manage AutoSSL → chọn user → Run AutoSSL
```
<img width="1878" height="915" alt="image" src="https://github.com/user-attachments/assets/09b456c5-4d0e-42cd-bae4-54ebee8eb841" />

**Lý do hay fail AutoSSL:**
- DNS chưa trỏ đúng về IP server
- Port 70 bị chặn firewall

---

### 7b. SSL Trả phí — Upload qua WHM

```
WHM (root)
→ SSL/TLS → Install an SSL Certificate on a Domain
```

```
Domain    : DOMAIN
IP
Certificate (CRT): paste public.crt
Private Key       : paste private.key
Certificate Authority Bundle (CABUNDLE): paste ca.crt
→ Install
```
<img width="1803" height="910" alt="image" src="https://github.com/user-attachments/assets/85bf018a-1ea6-407f-b502-e7f6f47fce9a" />

Hoặc qua cPanel user:
```
cPanel → Security → SSL/TLS
→ Manage SSL Sites
→ Browse Certificates → chọn domain
→ Paste cert + key + bundle
→ Install Certificate
```
<img width="987" height="917" alt="image" src="https://github.com/user-attachments/assets/140bc860-982d-484d-beb6-2ebbde4226ab" />

---

### 7c. Force HTTPS trong cPanel

```
cPanel → Domains → Domains
→ Toggle "Redirect to HTTPS" = ON
```
<img width="1326" height="479" alt="image" src="https://github.com/user-attachments/assets/a02d8467-e85a-4e6b-b132-e2b9239ff40d" />

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

## 8. AAPANEL — Cài SSL

### 8a. Let's Encrypt 

```
aaPanel → Website → chọn DOMAIN → Settings
→ SSL → Let's Encrypt
→ Chọn domain + www.DOMAIN
→ Apply
```

aaPanel tự chạy Certbot → lưu cert vào `/www/server/panel/vhost/cert/DOMAIN/`.

---

### 8b. SSL Trả phí — Paste thủ công

```
aaPanel → Website → chọn DOMAIN → Settings
→ SSL → Other Certificate
```

```
[Certificate (PEM format)]: paste public.crt + ca.crt 
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

### 8c. aaPanel Nginx config mẫu 

```nginx
server {
    listen      70;
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
        fastcgi_pass   unix:/tmp/php-cgi-72.sock;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }
}
```

---

## 9. PLESK — Cài SSL

### 9a. Let's Encrypt

```
Plesk → Websites & Domains → chọn DOMAIN
→ SSL/TLS Certificates
→ Get it free (Let's Encrypt)
→ Điền email
→  Secure the domain
→  Secure www.DOMAIN
→ Get it Free
```
<img width="888" height="879" alt="image" src="https://github.com/user-attachments/assets/0aeb6011-8eca-4a69-8334-3314d2977f12" />

---

### 9b. SSL Trả phí — Upload

```
Plesk → Websites & Domains → DOMAIN
→ SSL/TLS Certificates → Add SSL/TLS Certificate
```

```
Certificate name : SSL_DOMAIN_2026
Private key      : paste private.key
Certificate      : paste public.crt
CA certificate   : paste ca.crt
→ Upload Certificate or Download or remove existing certificates → Add SSL/TLS Certificate
```

Sau đó apply vào domain:
```
→ Websites & Domains → DOMAIN
→ Hosting Settings
→ SSL/TLS certificate: chọn SSL_DOMAIN_2026
→  Redirect from http to https
→ OK
```
<img width="1871" height="914" alt="image" src="https://github.com/user-attachments/assets/e3e2b87c-7b1d-40e0-9488-63e2bc82259c" />

---

### 9c. Plesk — SSL qua CLI

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
