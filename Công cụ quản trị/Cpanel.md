# Hướng dẫn sử dụng cPanel (2083) & WHM (2087) trên Ubuntu 22.04 LTS

> **Môi trường:** cPanel/WHM v134 · Ubuntu 22.04 LTS · Apache + PHP-FPM + MariaDB
> **Domain thực hành:** `hieucute.id.vn`
> **VPS IP:** `103.159.51.228`
> **Truy cập cPanel:** `https://103.159.51.228:2083` hoặc `https://hieucute.id.vn:2083`
> **Truy cập WHM:** `https://103.159.51.228:2087` (chỉ root/reseller)

---

## Mục lục

- [1. Quản lý Tên miền & Web](#1-quản-lý-tên-miền--web)
- [2. Quản lý File](#2-quản-lý-file)
- [3. Quản lý Tài khoản FTP](#3-quản-lý-tài-khoản-ftp)
- [4. Quản lý Cơ sở dữ liệu MySQL](#4-quản-lý-cơ-sở-dữ-liệu-mysql)
- [5. Bảo mật – Kích hoạt SSL miễn phí](#5-bảo-mật--kích-hoạt-ssl-miễn-phí)
- [6. Quản lý Email & Chống Spam](#6-quản-lý-email--chống-spam)
- [7. Mailing List](#7-mailing-list)
- [8. Cron Jobs](#8-cron-jobs)
- [9. Backup & Restore (cấp User)](#9-backup--restore-cấp-user)
- [10. Triển khai Next.js trên cPanel](#10-triển-khai-nextjs-trên-cpanel)
- [11. 5 lỗi phổ biến nhất trên cPanel 134 / Ubuntu 22.04](#11-5-lỗi-phổ-biến-nhất-trên-cpanel-134--ubuntu-2204)
- [12. WHM (2087) – Quản trị cấp Server](#12-whm-2087--quản-trị-cấp-server)
- [13. Webmail độc lập (2096)](#13-webmail-độc-lập-2096)
- [14. Bảng quyết định nhanh: Vào cổng nào?](#14-bảng-quyết-định-nhanh-vào-cổng-nào)
- [15. Học cPanel/WHM bằng tư duy DirectAdmin (Mapping kiến thức)](#15-học-cpanelwhm-bằng-tư-duy-directadmin-mapping-kiến-thức)
- [16. Thực chiến: Tạo Package → Tạo User → SSL → WordPress](#16-thực-chiến-tạo-package--tạo-user--ssl--wordpress)

---

Truy cập: `https://<IP-hoặc-domain>:<port>` — ví dụ `https://103.159.51.228:2087`.

> Chủ yếu dùng **2087 (WHM)** để xử lý ticket cấp server, và **2083 (cPanel)** để hỗ trợ/kiểm tra thay mặt khách hàng.

---

## 1. Quản lý Tên miền & Web

### 1.1 Trỏ tên miền về VPS

Trước khi làm bất kỳ thao tác nào trên cPanel, phải đảm bảo DNS của `hieucute.id.vn` đã trỏ về đúng IP VPS.

**Đăng nhập trang quản lý tên miền tại Nhân Hòa** → Zone DNS → kiểm tra/thêm bản ghi:

| Host | Type | Value | TTL |
|------|------|-------|-----|
| `@` | A | `103.159.51.228` | 300 |
| `www` | CNAME | `hieucute.id.vn.` | 300 |
| `mail` | A | `103.159.51.228` | 300 |

Kiểm tra DNS đã propagate chưa:
```bash
nslookup hieucute.id.vn
# hoặc
dig A hieucute.id.vn
```

Kết quả đúng phải trả về `103.159.51.228`.

---

### 1.2 Thêm Domain mới (Addon Domain / Alias)

Trong cPanel 134, Addon Domain và Alias Domain được gộp vào mục **Domains**.

**Bước 1:** Đăng nhập `https://103.159.51.228:2083` → mục **Domains** → chọn **Domains**.

**Bước 2:** Nhấn **Create A New Domain**.

**Bước 3:** Điền thông tin:

| Trường | Giá trị ví dụ | Ghi chú |
|--------|--------------|---------|
| Domain | `shop.hieucute.id.vn` | Nhập FQDN, không có `www` hay `http://` |
| Document Root | `/home/hieu/shop.hieucute.id.vn` | Tự động điền, có thể sửa |
| Share document root | **Bỏ tick** | Để domain chạy website riêng độc lập |

**Bước 4:** Nhấn **Submit**.

**Bước 5:** Thêm A record tại DNS của Nhân Hòa:
```
shop.hieucute.id.vn.   A   103.159.51.228   TTL 300
```

> ⚠️ **Lưu ý:**
> - Nếu tick **Share document root**, domain mới hiển thị **cùng nội dung** với domain chính — khó hoàn tác.
> - Chờ DNS propagate 5–30 phút trước khi kiểm tra.
> - VirtualHost được tạo tự động tại `/etc/apache2/conf.d/userdata/` trên Ubuntu 22.04.

---

### 1.3 Cài đặt WordPress qua WP Toolkit

**WP Toolkit** là công cụ chính thức của cPanel, nhanh và toàn diện hơn Softaculous.

**Bước 1:** cPanel → **Software** → **WordPress Toolkit**.

**Bước 2:** Nhấn **Install WordPress**.

**Bước 3:** Điền thông tin:

| Trường | Giá trị thực tế | Ghi chú |
|--------|----------------|---------|
| Domain | `hieucute.id.vn` | Chọn domain chính |
| Directory | Để trống | Cài tại root domain |
| WordPress version | Latest stable | Luôn dùng phiên bản mới nhất |
| Admin username | `admin_hieucute` | **Không dùng `admin`** |
| Admin password | *(tạo mạnh ≥12 ký tự)* | Lưu lại cẩn thận |
| Admin email | `info@hieucute.id.vn` | |

**Bước 4:** Nhấn **Install** → chờ 1–2 phút.

**Bước 5:** Truy cập `https://hieucute.id.vn/wp-admin`.

**Tính năng nổi bật của WP Toolkit:**
- **Smart Update:** Cập nhật an toàn, tự rollback nếu lỗi
- **Security Check:** Quét lỗ hổng bảo mật
- **Clone:** Tạo staging environment
- **Backup/Restore:** Backup từng WP site

> ⚠️ Nếu không thấy WP Toolkit: WHM → **Manage Plugins** → cài **WP Toolkit Deluxe**.

---

### 1.4 Cài WordPress qua Softaculous

**Bước 1:** cPanel → **Software** → **Softaculous Apps Installer**.

**Bước 2:** Tìm **WordPress** → nhấn **Install Now**.

**Bước 3:** Điền thông tin tương tự WP Toolkit → nhấn **Install**.

---

## 2. Quản lý File

### 2.1 Upload file lên public_html

**Cách 1: File Manager (file nhỏ)**

**Bước 1:** cPanel → **Files** → **File Manager**.

**Bước 2:** Điều hướng vào `public_html`.

**Bước 3:** Nhấn **Upload** → kéo thả hoặc chọn file → hệ thống hiển thị tiến trình realtime.

**Cách 2: FTP/SFTP (khuyến nghị cho project lớn)**

```
Host:     hieucute.id.vn  (hoặc 103.159.51.228)
Port:     21 (FTP) / 22 (SFTP)
Username: tài khoản cPanel
Password: mật khẩu cPanel
```

> Ưu tiên **SFTP (port 22)** thay FTP thông thường để mã hóa đường truyền.

---

### 2.2 Giải nén file

**Qua File Manager:**

**Bước 1:** Upload file `.zip` hoặc `.tar.gz` vào `public_html`.

**Bước 2:** Chuột phải vào file → **Extract** → chỉ định thư mục đích → **Extract File(s)**.

**Qua Terminal (file > 100MB):**
```bash
cd ~/public_html
unzip archive.zip
# hoặc
tar -xzvf archive.tar.gz
```

> ⚠️ Sau khi giải nén `.zip`, source code thường nằm trong `public_html/public_html/`. Cần **Select All → Move** ra đúng thư mục `public_html/`.

---

### 2.3 Phân quyền file (Chmod)

**Qua File Manager:**

Chọn file/thư mục → chuột phải → **Change Permissions** → nhập số quyền → **Change Permissions**.

**Quyền chuẩn cho WordPress:**

| Đối tượng | Quyền | Số |
|-----------|-------|----|
| Thư mục `public_html` | `rwxr-xr-x` | `755` |
| File `.php`, `.html` | `rw-r--r--` | `644` |
| File `wp-config.php` | `rw-------` | `600` |
| Thư mục `wp-content/uploads` | `rwxrwxr-x` | `775` |

**Phân quyền hàng loạt qua Terminal:**
```bash
# Tất cả thư mục → 755
find ~/public_html -type d -exec chmod 755 {} \;

# Tất cả file → 644
find ~/public_html -type f -exec chmod 644 {} \;

# wp-config.php → 600
chmod 600 ~/public_html/wp-config.php
```

> ⚠️ **Lỗi thường gặp:**
> - **403 Forbidden** → thư mục bị chmod `700` hoặc `777`. Sửa về `755`.
> - **500 Internal Server Error** sau chmod → file `.php` bị set `777`. Sửa về `644`.
> - **Không upload được ảnh WP** → thư mục `wp-content/uploads` cần ít nhất `755`.

---

## 3. Quản lý Tài khoản FTP

FTP (File Transfer Protocol) cho phép quản lý file hosting qua phần mềm FTP client (FileZilla, WinSCP...). Tất cả gói hosting cPanel đều hỗ trợ FTP qua port **21**.

### 3.1 Tạo tài khoản FTP

**Bước 1:** cPanel → **Files** → **FTP Accounts**.

**Bước 2:** Nhấn **Create FTP Account** (hoặc điền thông tin trong form ở trên cùng).

**Bước 3:** Điền thông tin:

| Trường | Giá trị ví dụ | Ghi chú |
|--------|--------------|---------|
| Log In | `ftpuser` | Tên đăng nhập đầy đủ: `ftpuser@hieucute.id.vn` |
| Password | *(tạo mạnh)* | Dùng Password Generator |
| Directory | `/home/hieu/public_html` | Giới hạn quyền truy cập trong thư mục này |
| Quota | `500 MB` | Hoặc Unlimited |

**Bước 4:** Nhấn **Create FTP Account**.

**Bước 5:** Mở FileZilla → nhập thông tin:

```
Host:     ftp.hieucute.id.vn  (hoặc 103.159.51.228)
Username: ftpuser@hieucute.id.vn
Password: (mật khẩu vừa tạo)
Port:     21
```

### 3.2 Quản lý và xóa tài khoản FTP

- cPanel → **FTP Accounts** → danh sách hiển thị bên dưới.
- Nhấn **Change Password** để đổi mật khẩu.
- Nhấn **Delete** → chọn có xóa thư mục hay không → xác nhận.

### 3.3 FTP mặc định

Tài khoản FTP chính (dùng thông tin đăng nhập cPanel) luôn tồn tại và không thể xóa. Đây là tài khoản có quyền truy cập toàn bộ hosting.

> ⚠️ **Lưu ý bảo mật:**
> - Ưu tiên dùng **SFTP (port 22)** thay FTP thông thường để mã hóa dữ liệu truyền tải.
> - Tạo FTP account riêng với thư mục giới hạn khi cần chia sẻ quyền cho bên thứ ba.
> - Không dùng `Anonymous FTP` trên môi trường production.

---

## 4. Quản lý Cơ sở dữ liệu MySQL

### 4.1 Tạo Database, User và gán quyền

#### Cách 1: Database Wizard (khuyến nghị)

**Bước 1:** cPanel → **Databases** → **MySQL Database Wizard**.

**Bước 2 – Tạo Database:** Nhập `wpdb` → **Next Step**.
→ Tên thực tế: `hieu_wpdb` *(có prefix tài khoản cPanel)*.

**Bước 3 – Tạo User:** Nhập `wpuser` + Password → **Create User**.
→ Tên thực tế: `hieu_wpuser`.

**Bước 4 – Gán quyền:** Tích **ALL PRIVILEGES** → **Next Step**.

**Bước 5:** Lưu lại thông tin kết nối:

```
DB_NAME:     hieu_wpdb
DB_USER:     hieu_wpuser
DB_PASSWORD: (mật khẩu vừa tạo)
DB_HOST:     localhost
```

#### Cách 2: MySQL Databases (thủ công)

**Bước 1:** cPanel → **MySQL Databases**.

**Tạo Database:** Nhập tên → **Create Database**.

**Tạo User:** Kéo xuống **MySQL Users** → nhập Username + Password → **Create User**.

**Gán quyền:** Mục **Add User To Database** → chọn User và Database → **Add** → tích **ALL PRIVILEGES** → **Make Changes**.

#### Kiểm tra qua phpMyAdmin

cPanel → **Databases** → **phpMyAdmin** → chọn database ở panel trái → xác nhận tồn tại.

> ⚠️ **Lưu ý quan trọng:**
> - Tên DB và User **tự động thêm prefix** tên tài khoản cPanel. Phải điền tên đầy đủ (có prefix) vào `wp-config.php`.
> - Mật khẩu DB User phải đạt tối thiểu **65/100** theo thước đo của cPanel.
> - Để truy cập DB từ IP ngoài: **Remote Database Access** → thêm IP cho phép.

---

## 5. Bảo mật – Kích hoạt SSL miễn phí

### 5.1 AutoSSL (Let's Encrypt tự động)

Hosting Nhân Hòa tích hợp sẵn AutoSSL — cấp SSL miễn phí, tự gia hạn, không cần can thiệp thủ công.

**Bước 1:** cPanel → **Security** → **SSL/TLS Status**.

**Bước 2:** Danh sách domain hiển thị trạng thái SSL hiện tại.

**Bước 3:** Tick chọn `hieucute.id.vn` và `www.hieucute.id.vn` → nhấn **Run AutoSSL**.

**Bước 4:** Chờ 1–3 phút → trạng thái chuyển ✅.

**Bước 5:** Kiểm tra tại `https://hieucute.id.vn`.

---

### 5.2 Cài SSL có sẵn (từ ZeroSSL, Sectigo...) theo hướng dẫn Nhân Hòa

Nếu bạn đã có file chứng chỉ từ CA, thực hiện theo các bước:

**Bước 1:** cPanel → **Security** → **SSL/TLS** → **Manage SSL Sites**.

**Bước 2:** Chọn domain `hieucute.id.vn` cần cài.

**Bước 3:** Paste nội dung 3 file chứng chỉ vào đúng ô:

| Ô nhập | File tương ứng | Nội dung bắt đầu bằng |
|--------|---------------|----------------------|
| Certificate (CRT) | `certificate.crt` hoặc `.pem` | `-----BEGIN CERTIFICATE-----` |
| Private Key (KEY) | `server.key` | `-----BEGIN PRIVATE KEY-----` |
| Certificate Authority Bundle (CABUNDLE) | `ca_bundle.crt` | `-----BEGIN CERTIFICATE-----` |

**Bước 4:** Nhấn **Install Certificate**.

**Bước 5:** Quay lại cPanel → **Domains** → chọn `hieucute.id.vn` → bật **Force HTTPS Redirect**.

---

### 5.3 Gỡ SSL cũ trước khi chạy lại AutoSSL (khi AutoSSL bị lỗi)

Theo hướng dẫn Nhân Hòa, khi AutoSSL thất bại cần xóa sạch cert cũ trước:

**Bước 1:** cPanel → **SSL/TLS** → **Private Keys** → xóa key cũ.

**Bước 2:** **SSL/TLS** → **Certificates** → **Uninstall** cert cũ → **Delete Key**.

**Bước 3:** **File Manager** → `public_html` → xóa thư mục `.well-known` (nếu có).

**Bước 4:** **SSL/TLS Status** → tick tất cả domain → **Run AutoSSL**.

**Bước 5:** Chờ 5–10 phút → kiểm tra lại.

---

### 5.4 Cấu hình Force HTTPS Redirect

**Qua giao diện cPanel:**
cPanel → **Domains** → chọn `hieucute.id.vn` → bật toggle **Force HTTPS Redirect** → **Save**.

**Qua `.htaccess`:**
```apache
RewriteEngine On
RewriteCond %{HTTPS} !=on
RewriteRule ^ https://%{HTTP_HOST}%{REQUEST_URI} [R=301,L]
```

> ⚠️ **Lưu ý và lỗi thường gặp:**
> - **AutoSSL thất bại** → DNS chưa trỏ đúng hoặc port 80 bị block. Kiểm tra: `curl -I http://hieucute.id.vn`
> - **Lỗi CAA record** → thêm: `hieucute.id.vn. CAA 0 issue "letsencrypt.org"`
> - **Mixed Content sau HTTPS** → WordPress: Settings → General → đổi `http://` → `https://`. Cài plugin **Really Simple SSL**.
> - **Chứng chỉ hết hạn** → AutoSSL tự gia hạn, nhưng nếu DNS thay đổi có thể thất bại. Kiểm tra log: WHM → **AutoSSL** → **Logs**.

---

## 6. Quản lý Email & Chống Spam

### 6.1 Tạo tài khoản Email theo tên miền

**Bước 1:** cPanel → **Email** → **Email Accounts**.

**Bước 2:** Nhấn **Create (+)**.

**Bước 3:** Điền thông tin:

| Trường | Giá trị | Ghi chú |
|--------|---------|---------|
| Username | `info` | Địa chỉ: `info@hieucute.id.vn` |
| Domain | `hieucute.id.vn` | Chọn từ dropdown |
| Password | *(tạo mạnh)* | Dùng Password Generator |
| Storage Space | `1024 MB` | Hoặc Unlimited nếu gói cho phép |

**Bước 4:** Nhấn **Create**.

**Bước 5:** Truy cập Webmail:
- `https://hieucute.id.vn/webmail`
- `https://103.159.51.228/webmail`
- Hoặc trong danh sách Email Accounts → nhấn **Check Email**.

> ⚠️ Khi đăng nhập Webmail phải nhập **đầy đủ**: `info@hieucute.id.vn`, không chỉ `info`.

---

### 6.2 Restrict – Giới hạn email gửi đi (chống spam từ tài khoản bị xâm phạm)

Khi nghi ngờ một địa chỉ email đang bị spam hoặc bị xâm phạm, dùng tính năng **Restrict** để giới hạn chiều gửi đi.

**Bước 1:** cPanel → **Email** → **Email Accounts**.

**Bước 2:** Tìm email cần hạn chế → nhấn **Manage**.

**Bước 3:** Tìm mục **Restrictions** (hoặc **Outgoing Mail Limit**).

**Bước 4:** Đặt giới hạn số email gửi mỗi giờ → **Save**.

> Tính năng này đặc biệt hữu ích khi phát hiện một email trong domain đang gửi spam, giúp ngăn chặn nhanh mà không cần xóa tài khoản.

---

### 6.3 Cấu hình SPF, DKIM, DMARC trong cPanel

Truy cập trực tiếp: cPanel → **Email** → **Email Deliverability**.

Trang này hiển thị trạng thái SPF, DKIM, DMARC của từng domain. Nhấn **Repair** để cPanel tự động sửa nếu phát hiện vấn đề.

#### SPF

| Host | Type | Value |
|------|------|-------|
| `@` | TXT | `v=spf1 ip4:103.159.51.228 ~all` |

#### DKIM

**Bước 1:** Email Deliverability → chọn `hieucute.id.vn` → nhấn **Enable** DKIM.

**Bước 2:** cPanel tự tạo key và hiển thị bản ghi DNS:
```
default._domainkey.hieucute.id.vn   TXT   "v=DKIM1; k=rsa; p=MIIBIjAN..."
```

**Bước 3:** Copy bản ghi này → vào DNS Zone tại Nhân Hòa → thêm bản ghi TXT.

**Kiểm tra:**
```bash
dig TXT default._domainkey.hieucute.id.vn
```

#### DMARC – Triển khai theo lộ trình

**Thêm vào Zone Editor:** cPanel → **Domains** → **Zone Editor** → `hieucute.id.vn` → **Manage** → **Add Record** → Type: TXT.

| Giai đoạn | Host | Value |
|-----------|------|-------|
| 1 – Mới cài | `_dmarc` | `v=DMARC1; p=none; rua=mailto:admin@hieucute.id.vn` |
| 2 – Sau 1–2 tuần | `_dmarc` | `v=DMARC1; p=quarantine; pct=100; rua=mailto:admin@hieucute.id.vn` |
| 3 – Ổn định | `_dmarc` | `v=DMARC1; p=reject; pct=100; rua=mailto:admin@hieucute.id.vn` |

**Kiểm tra toàn bộ:**
```bash
dig TXT hieucute.id.vn | grep spf
dig TXT default._domainkey.hieucute.id.vn
dig TXT _dmarc.hieucute.id.vn
```

Kiểm tra online: https://mxtoolbox.com/emailhealth/

---

## 7. Mailing List

### 7.1 Tạo Mailing List

**Bước 1:** cPanel → **Email** → **Mailing Lists**.

**Bước 2:** Điền thông tin:

| Trường | Ví dụ | Ghi chú |
|--------|-------|---------|
| List Name | `newsletter` | Địa chỉ: `newsletter@hieucute.id.vn` |
| Domain | `hieucute.id.vn` | |
| Password | *(tạo mạnh)* | Dùng để quản lý danh sách |

**Bước 3:** Nhấn **Add Mailing List**.

### 7.2 Giới hạn thành viên được phép gửi vào Mailing List

Mặc định cPanel cho phép **tất cả thành viên** của Mailing List gửi mail vào danh sách. Nếu chỉ muốn một số email cụ thể được phép gửi:

**Bước 1:** cPanel → **Email** → **Mailing Lists** → nhấn **Manage** (hoặc **Edit**) cạnh Mailing List cần cấu hình.

**Bước 2:** Hệ thống chuyển đến giao diện quản lý **Mailman**.

**Bước 3:** Vào **Privacy options** → **Sender filters**.

**Bước 4:** Tại mục **"Restrict posting privilege to list members"**:
- Chọn **Yes** để chỉ thành viên trong list mới được gửi.
- Hoặc thêm địa chỉ cụ thể vào **"List of non-member addresses whose postings should be automatically accepted"**.

**Bước 5:** Nhấn **Submit Your Changes**.

> ⚠️ Chỉ email được liệt kê mới có thể gửi vào List — email từ người khác sẽ bị giữ lại chờ duyệt hoặc bị từ chối tùy cấu hình.

---

## 8. Cron Jobs

### 8.1 Cron Job là gì

Cron Job là tác vụ được đặt lịch chạy tự động theo thời gian định sẵn trên server — không cần thao tác thủ công. Ứng dụng phổ biến: tự động backup, gửi email định kỳ, đồng bộ dữ liệu, dọn dẹp file log...

### 8.2 Tạo Cron Job trong cPanel

**Bước 1:** cPanel → **Advanced** → **Cron Jobs**.

**Bước 2:** Mục **Add New Cron Job** — điền thời gian chạy:

| Trường | Ý nghĩa | Phạm vi |
|--------|---------|---------|
| Minute | Phút | 0–59 |
| Hour | Giờ | 0–23 |
| Day | Ngày trong tháng | 1–31 |
| Month | Tháng | 1–12 |
| Weekday | Thứ trong tuần | 0–7 (0 và 7 đều là Chủ nhật) |

Dùng `*` = mọi giá trị (mỗi phút, mỗi giờ...).

**Bước 3:** Nhập lệnh cần chạy vào ô **Command**.

**Bước 4:** Nhấn **Add New Cron Job**.

### 8.3 Bảng cú pháp cron phổ biến

| Cú pháp | Ý nghĩa |
|---------|---------|
| `* * * * *` | Mỗi phút |
| `0 * * * *` | Mỗi giờ (vào phút 0) |
| `0 2 * * *` | Mỗi ngày lúc 2:00 sáng |
| `0 2 * * 0` | Mỗi Chủ nhật lúc 2:00 sáng |
| `0 2 1 * *` | Ngày 1 mỗi tháng lúc 2:00 sáng |
| `*/5 * * * *` | Mỗi 5 phút |
| `0 2,14 * * *` | Mỗi ngày lúc 2:00 và 14:00 |

### 8.4 Ví dụ thực tế

**Gia hạn SSL tự động (Let's Encrypt):**
```bash
0 12 * * * /usr/bin/certbot renew --quiet
```

**Backup database hàng ngày lúc 2:00 sáng:**
```bash
0 2 * * * mysqldump -u hieu_wpuser -p'password' hieu_wpdb | gzip > /home/hieu/backups/db_$(date +\%Y-\%m-\%d).sql.gz
```

**Chạy WordPress cron (thay WP-Cron mặc định để tăng hiệu năng):**
```bash
*/15 * * * * wget -q -O /dev/null "https://hieucute.id.vn/wp-cron.php?doing_wp_cron"
```

**Xóa file backup cũ hơn 7 ngày:**
```bash
0 3 * * * find /home/hieu/backups -type f -mtime +7 -delete
```

**Script backup tự động (tạo trên VPS, đặt cron chạy):**
```bash
# Tạo script /home/hieu/backup.sh
#!/bin/bash
BACKUP_DIR="/home/hieu/backups/$(date +%Y-%m-%d)"
mkdir -p $BACKUP_DIR

# Backup database
mysqldump -u hieu_wpuser -p'password' hieu_wpdb | gzip -9 > $BACKUP_DIR/db_hieucute.sql.gz

# Backup source code
zip -r $BACKUP_DIR/source_hieucute.zip /home/hieu/public_html/ -q

echo "Backup hoàn tất: $BACKUP_DIR"

# Xóa backup cũ hơn 3 ngày
find /home/hieu/backups -type d -mtime +3 -exec rm -rf {} +
```

```bash
chmod +x /home/hieu/backup.sh
```

Cron chạy hàng ngày lúc 1:00 sáng:
```
0 1 * * * /home/hieu/backup.sh
```

> ⚠️ **Lưu ý:**
> - Script kích hoạt Cron Job có thể chạy lệnh trực tiếp trên server → kiểm tra kỹ script trước khi đặt lịch.
> - Dùng đường dẫn **tuyệt đối** trong lệnh cron (ví dụ `/usr/bin/php` thay vì chỉ `php`).
> - Để nhận thông báo lỗi qua email: thêm `MAILTO=admin@hieucute.id.vn` ở đầu crontab.

---

## 9. Backup & Restore (cấp User)

### 9.1 Tạo Full Backup thủ công (theo hướng dẫn Nhân Hòa)

**Bước 1:** cPanel → **Files** → **Backup Wizard**.

**Bước 2:** Chọn **Back Up**.

**Bước 3:** Chọn loại backup:

| Loại | Nội dung | Dùng khi |
|------|----------|---------|
| **Full Account Backup** | Toàn bộ: file, DB, email, DNS, cấu hình | Di chuyển hosting, backup định kỳ |
| **Home Directory** | Chỉ file trong `/home/hieu/` | Backup code nhanh |
| **MySQL Databases** | Chỉ database | Trước khi cập nhật WordPress |

**Bước 4:** Chọn **Full Account Backup** → chọn đích lưu:

| Đích | Mô tả |
|------|-------|
| **Home Directory** | Lưu tại `/home/hieu/` trên server |
| **Remote FTP Server** | Gửi sang FTP server backup |
| **Secure Copy (SCP)** | Gửi qua SSH |

**Bước 5:** Điền email nhận thông báo → **Generate Backup**.

**Bước 6:** Tải file backup về máy:
- File Manager → tìm file `.tar.gz` trong thư mục home → chuột phải → **Download**.

---

### 9.2 Backup nhanh từng phần

**Backup database:**
cPanel → **Files** → **Backup** → **Download a MySQL Database Backup** → chọn database.

**Backup thư mục Home:**
cPanel → **Files** → **Backup** → **Download a Home Directory Backup**.

---

### 9.3 Restore – Quy trình chi tiết theo Nhân Hòa

#### Restore Source Code

**Bước 1:** cPanel → **Backup Wizard** → **Restore** → **Home Directory** → chọn file `.tar.gz` → **Upload**.

**Bước 2:** Sau khi hệ thống giải nén, vào **File Manager**:
- Chuột phải vào file backup → **Extract** (không điền thêm gì) → giải nén ra folder tên tương tự.
- Vào folder vừa tạo → `HomeDir` → `public_html` → **Select All** → chuột phải → **Move**.
- Đổi đường dẫn đích về `/home/hieu/public_html` → **Move Files**.

**Bước 3:** Upload lại source code nếu cần:
- Nén source thành `public_html.zip` → upload lên `public_html` → giải nén.
- Source sẽ nằm trong `public_html/public_html/` → **Select All** → **Move** ra `public_html/`.

#### Restore MySQL Database

**Bước 1:** cPanel → **Backup Wizard** → **Restore** → **MySQL Databases** → chọn file `.sql.gz` → **Upload**.

**Bước 2:** Hoặc import qua phpMyAdmin:
- phpMyAdmin → chọn database `hieu_wpdb` → tab **Import** → chọn file SQL → **Go**.

> ⚠️ **Lưu ý quan trọng:**
> - **Restore sẽ ghi đè dữ liệu hiện tại** → luôn tạo backup mới trước khi restore bản cũ.
> - Đảm bảo **database đích đã tồn tại** trước khi import file SQL.
> - File **Full Account Backup** (`.tar.gz` tổng hợp) cần restore qua **WHM** → **Restore a Full Backup/cpmove File** — không restore được qua cPanel user thường (xem mục 12.9).

---

## 10. Triển khai Next.js trên cPanel

cPanel hỗ trợ chạy ứng dụng Node.js (bao gồm Next.js) thông qua tính năng **Setup Node.js App** dùng Phusion Passenger.

### 10.1 Yêu cầu

- cPanel có tính năng **Setup Node.js App** (kiểm tra trong mục **Software**).
- Node.js version phù hợp với Next.js (Node ≥ 18 cho Next.js 14+).

### 10.2 Tạo Node.js App

**Bước 1:** cPanel → **Software** → **Setup Node.js App**.

**Bước 2:** Nhấn **Create Application**.

**Bước 3:** Điền thông tin:

| Trường | Giá trị ví dụ | Ghi chú |
|--------|--------------|---------|
| Node.js version | `18.x` hoặc `20.x` | Chọn phiên bản phù hợp |
| Application mode | `Production` | |
| Application root | `nextjs_app` | Thư mục chứa source, tương đối với home |
| Application URL | `hieucute.id.vn` | Domain muốn chạy |
| Application startup file | `app.js` | File khởi động (tự tạo ở bước sau) |

**Bước 4:** Nhấn **Create** → app được tạo.

### 10.3 Cài đặt Next.js

**Bước 1:** Copy đường dẫn terminal từ giao diện Setup Node.js App → truy cập **Terminal** trong cPanel (Advanced → Terminal).

**Bước 2:** Vào thư mục app và xóa file mặc định:
```bash
cd ~/nextjs_app
rm -rf *
```

**Bước 3:** Khởi tạo project Next.js:
```bash
npx create-next-app@latest . --yes
```

**Bước 4:** Build project:
```bash
npm run build
```

### 10.4 Tạo file khởi động app.js

Tạo file `app.js` trong thư mục project:
```javascript
const { createServer } = require('http');
const { parse } = require('url');
const next = require('next');

const dev = process.env.NODE_ENV !== 'production';
const hostname = 'localhost';
const port = process.env.PORT || 3000;

const app = next({ dev, hostname, port });
const handle = app.getRequestHandler();

app.prepare().then(() => {
    createServer(async (req, res) => {
        try {
            const parsedUrl = parse(req.url, true);
            await handle(req, res, parsedUrl);
        } catch (err) {
            console.error('Error occurred handling', req.url, err);
            res.statusCode = 500;
            res.end('internal server error');
        }
    }).listen(port, (err) => {
        if (err) throw err;
        console.log(`> Ready on http://${hostname}:${port}`);
    });
});
```

### 10.5 Khởi động App

**Bước 1:** cPanel → **Setup Node.js App** → tìm app `hieucute.id.vn`.

**Bước 2:** Nhấn **Stop App** → **Start App** để áp dụng thay đổi.

**Bước 3:** Truy cập `https://hieucute.id.vn` để kiểm tra.

> ⚠️ **Lưu ý:**
> - Nếu hosting không có Terminal: liên hệ kỹ thuật Nhân Hòa để enable tính năng.
> - Sau mỗi lần thay đổi code cần Stop → Start lại app.
> - Biến môi trường (API keys, DB...) điền vào ô **Environment variables** trong Setup Node.js App.
> - Next.js với `export` (Static) không cần Node.js runtime — có thể deploy thẳng file tĩnh vào `public_html`.

---

## 11. 5 lỗi phổ biến nhất trên cPanel 134 / Ubuntu 22.04

### Lỗi 1 – Website hiển thị "500 Internal Server Error"

**Nguyên nhân phổ biến:**
- File `.htaccess` bị sai cú pháp
- Quyền file sai (PHP file bị chmod 777 hoặc 000)
- PHP version không tương thích

**Kiểm tra log:**
- cPanel → **Metrics** → **Errors**
- File Manager → `public_html/error_log`

**Khắc phục:**
```bash
# Đổi tên .htaccess để test
mv ~/public_html/.htaccess ~/public_html/.htaccess_bak

# Sửa quyền hàng loạt
find ~/public_html -type f -name "*.php" -exec chmod 644 {} \;
find ~/public_html -type d -exec chmod 755 {} \;
```

- cPanel → **Software** → **MultiPHP Manager** → đổi PHP version.

---

### Lỗi 2 – Email gửi vào Spam hoặc bị từ chối

**Nguyên nhân phổ biến:**
- Thiếu SPF, DKIM, DMARC
- IP `103.159.51.228` bị blacklist
- Thiếu PTR record (Reverse DNS)

**Kiểm tra log:**
- cPanel → **Email** → **Email Deliverability**
- cPanel → **Email** → **Track Delivery**

**Khắc phục:**
1. Email Deliverability → **Repair** để tự động sửa SPF/DKIM.
2. Kiểm tra blacklist: https://mxtoolbox.com/blacklists.aspx
3. Liên hệ Nhân Hòa tạo PTR record: `103.159.51.228` → `mail.hieucute.id.vn`.

---

### Lỗi 3 – AutoSSL thất bại cho hieucute.id.vn

**Nguyên nhân phổ biến:**
- DNS chưa trỏ `hieucute.id.vn` về `103.159.51.228`
- Port 80 bị block
- Cert cũ bị conflict

**Kiểm tra:**
```bash
dig A hieucute.id.vn                    # Phải trả về 103.159.51.228
curl -I http://hieucute.id.vn           # Server phải phản hồi
dig CAA hieucute.id.vn                  # Kiểm tra CAA record
```

**Khắc phục:**
- Xóa cert cũ (theo bước 5.3) → Run AutoSSL lại.
- Nếu vẫn lỗi: kiểm tra firewall VPS có mở port 80 không.

---

### Lỗi 4 – WordPress lỗi "Error establishing a database connection"

**Nguyên nhân phổ biến:**
- Sai thông tin DB trong `wp-config.php` — thường do **quên prefix** tên tài khoản cPanel.

**Kiểm tra:**
- phpMyAdmin → thử đăng nhập bằng DB user/password để xác nhận thông tin.

**Khắc phục:**

File Manager → `public_html/wp-config.php` → **Edit**:
```php
define( 'DB_NAME',     'hieu_wpdb' );    // Phải có prefix "hieu_"
define( 'DB_USER',     'hieu_wpuser' );  // Phải có prefix "hieu_"
define( 'DB_PASSWORD', 'correct_password' );
define( 'DB_HOST',     'localhost' );
```

Nếu quên mật khẩu DB: cPanel → **MySQL Databases** → **Current Users** → **Change Password**.

---

### Lỗi 5 – Upload file lỗi "413 Request Entity Too Large" hoặc timeout

**Nguyên nhân:** Vượt giới hạn `upload_max_filesize` hoặc `post_max_size` trong PHP.

**Kiểm tra log:**
- cPanel → **Metrics** → **Errors** → tìm dòng `413`

**Khắc phục:**

**Cách 1: MultiPHP INI Editor:**
cPanel → **Software** → **MultiPHP INI Editor** → chọn `hieucute.id.vn` → sửa:

| Directive | Giá trị |
|-----------|---------|
| `upload_max_filesize` | `128M` |
| `post_max_size` | `128M` |
| `max_execution_time` | `300` |
| `memory_limit` | `256M` |

**Cách 2: File `.htaccess`:**
```apache
php_value upload_max_filesize 128M
php_value post_max_size 128M
php_value max_execution_time 300
php_value memory_limit 256M
```

---

## Bảng vị trí log quan trọng trong cPanel 134

| Log | Vị trí trong cPanel |
|-----|---------------------|
| Error log website | **Metrics → Errors** hoặc `~/public_html/error_log` |
| Access log | **Metrics → Raw Access** |
| Email gửi/nhận | **Email → Track Delivery** |
| Trạng thái SSL | **Security → SSL/TLS Status** |
| Bandwidth/Resource | **Metrics → Resource Usage** |
| Apache error (SSH) | `/var/log/apache2/error_log` |
| MySQL error (SSH) | `/var/lib/mysql/error.log` |
| cPanel logs (SSH) | `/usr/local/cpanel/logs/` |

---

## 12. WHM (2087) – Quản trị cấp Server

> WHM (Web Host Manager) là panel dành cho **root hoặc reseller**, khác hoàn toàn cPanel: cPanel quản lý **1 tài khoản hosting**, còn WHM quản lý **toàn bộ server** (nhiều tài khoản cPanel, dịch vụ hệ thống, firewall, backup server...).

**Đăng nhập:** `https://<server-ip>:2087` — chỉ root hoặc reseller được cấp quyền.

### 12.1 Quản lý tài khoản (việc KTV làm nhiều nhất)

**Tạo account cPanel mới:**
WHM → **Account Functions** → **Create a New Account**
- Domain, Username, Password
- Chọn **Package** (gói hosting đã tạo sẵn — dung lượng, băng thông, số email...)
- Chọn IP (Dedicated hoặc Shared)

**Suspend / Unsuspend** (khi khách chưa thanh toán, hoặc vi phạm ToS):
WHM → **Account Functions** → **Suspend/Unsuspend an Account**
> Lý do suspend nên ghi rõ, vì khách sẽ thấy message này khi truy cập site.

**Terminate (xóa vĩnh viễn) — CỰC KỲ NGUY HIỂM:**
WHM → **Account Functions** → **Terminate a Multi-User Account**
> ⚠️ Luôn tạo Full Backup trước khi terminate. Thao tác không thể hoàn tác.

**Đổi package / limit tài khoản:**
WHM → **Modify an Account** — đổi dung lượng, băng thông, số addon domain...

### 12.2 Package (gói hosting)

WHM → **Packages** → **Add a Package**

| Trường | Ví dụ |
|--------|-------|
| Disk Quota | 5000 MB |
| Bandwidth | 50000 MB |
| Max Addon Domains | 5 |
| Max Email Accounts | Unlimited |
| Max Databases | 10 |
| Max FTP Accounts | Unlimited |

### 12.3 Quản lý dịch vụ hệ thống

- **Service Manager**: bật/tắt/restart Apache, MySQL, Exim, DNS (BIND)...
- **Restart Services** → chọn dịch vụ → **Restart**

### 12.4 DNS Cluster / Zone (cấp server)

WHM → **DNS Functions** → **Edit DNS Zone** — sửa zone của bất kỳ domain nào trên server (khác với cPanel user chỉ sửa zone qua Zone Editor giới hạn).

### 12.5 SSL cấp server

- **Manage AutoSSL** → chọn provider (Let's Encrypt/Sectigo), chạy AutoSSL cho toàn bộ hoặc từng account.
- **Install an SSL Certificate** → cài cert cho cả server (ví dụ cert cho hostname `server.nhanhoa.vn`).

### 12.6 Bảo mật & Firewall

- **ConfigServer Security & Firewall (CSF)** nếu đã cài.
- **Security Center** → **Compilers** (khóa gcc để hạn chế user tự biên dịch mã độc), **Manage Shell Access**.

### 12.7 Giám sát tài nguyên server

- WHM → **Server Status** → **Service Status**, **Process Manager**
- **Show Current Disk Usage**, **Bandwidth**
- Log server: `/usr/local/cpanel/logs/`

### 12.8 WP Toolkit Deluxe

WHM → **Manage Plugins** → cài **WP Toolkit Deluxe** (nếu cPanel user không thấy WP Toolkit ở mục 1.3).

### 12.9 Backup cấp server (khác Backup Wizard của user)

WHM → **Backup** → **Backup Configuration**
- Cấu hình backup tự động toàn server (destination: local, FTP, S3, SCP...)
- **Restore a Full Backup/cpmove File** — đây là chỗ **duy nhất** phục hồi được file `.tar.gz` Full Account Backup (nhắc ở mục 9.3).

### 12.10 Bảng nhanh: Việc nào làm ở WHM, việc nào ở cPanel

| Việc cần làm | Làm ở đâu |
|--------------|-----------|
| Tạo account hosting mới cho khách | WHM |
| Suspend/Terminate account | WHM |
| Restart Apache/MySQL toàn server | WHM |
| Cấu hình CSF Firewall | WHM |
| Restore Full Account Backup (.tar.gz/cpmove) | WHM |
| Cài WordPress, quản lý 1 website | cPanel |
| Tạo email, FTP, database của 1 account | cPanel |
| Cài SSL cho 1 domain cụ thể | cPanel (hoặc WHM nếu cấp server) |
| Sửa .htaccess, chmod file | cPanel |

---

## 13. Webmail độc lập (2096)

Nếu khách không nhớ cách vào webmail qua cPanel, họ có thể vào thẳng:
```
https://mail.hieucute.id.vn:2096
```
hoặc
```
https://103.159.51.228:2096
```
Đăng nhập bằng **địa chỉ email đầy đủ** (`info@hieucute.id.vn`) + mật khẩu — không cần qua cPanel trước.

---

## 14. Bảng quyết định nhanh: Vào cổng nào?

| Tình huống | Vào cổng nào |
|------------|-------------|
| Khách quên mật khẩu email | 2083 (cPanel) để đổi pass, sau đó đăng nhập lại ở 2096 |
| Cần tạo account hosting mới cho khách | **2087 (WHM)** |
| Server bị đầy dịch vụ, cần restart Apache/MySQL | **2087 (WHM)** |
| Khách báo web lỗi 500, cần check code/permission | 2083 (cPanel), hoặc SSH trực tiếp |
| Cần suspend account do nợ cước | **2087 (WHM)** |
| Khách chỉ muốn check mail nhanh, không nhớ domain login | 2096 (Webmail) |
| Restore Full Account Backup (.tar.gz tổng hợp) | **2087 (WHM)** |
| Cài WordPress, tạo email/FTP/DB cho 1 site | 2083 (cPanel) |

---

## 15. Học cPanel/WHM bằng tư duy DirectAdmin (Mapping kiến thức)

> Bạn đã thuộc lòng hierarchy DirectAdmin (Admin → Reseller → User) và cách Nhân Hòa bypass tầng Reseller. cPanel/WHM có cùng triết lý phân quyền nhưng khác tên gọi và khác cách tổ chức UI. Học phần này bằng cách **map thẳng khái niệm cũ sang khái niệm mới**, đừng học lại từ số 0.

### 15.1 Bảng ánh xạ Hierarchy (quan trọng nhất)

| DirectAdmin | cPanel/WHM tương đương | Ghi chú khác biệt |
|-------------|------------------------|--------------------|
| **Admin** (toàn quyền server) | **root trong WHM** | Cả 2 đều toàn quyền hệ thống |
| **Reseller** (quản lý nhiều User, có Package riêng) | **Reseller account trong WHM** | WHM: Reseller đăng nhập chung giao diện WHM (giới hạn quyền), DirectAdmin: Reseller có UI riêng biệt hẳn với Admin |
| **User** (chủ 1 website/hosting) | **cPanel account (User)** | Tương đương 1-1, đăng nhập ở cổng riêng (2083 cPanel ↔ port User DirectAdmin) |
| **User Package** (gói giới hạn dung lượng/băng thông) | **Package** (WHM → Packages) | Tương đương 1-1 gần như hoàn toàn |
| Giao diện Admin (port riêng, thường 2222) | **WHM (port 2087)** | Cùng vai trò: quản trị server tổng |
| Giao diện User (port 2222, cùng port với Admin nhưng khác quyền) | **cPanel (port 2083)** | DirectAdmin dùng chung 1 port 2222 cho cả 3 vai trò (phân biệt bằng quyền đăng nhập); cPanel/WHM **tách hẳn port**: 2087 (WHM) và 2083 (cPanel) |

> 🔑 **Điểm khác biệt lớn nhất bạn cần nhớ:** DirectAdmin dùng **1 cổng duy nhất (2222)** cho cả Admin/Reseller/User, phân biệt bằng tài khoản đăng nhập. cPanel/WHM **tách vật lý theo cổng**: quản trị viên luôn vào 2087, khách hàng luôn vào 2083. Đây là lý do tài liệu này phải nói rõ "port nào làm việc gì" ngay từ đầu.

### 15.2 Bảng ánh xạ thao tác thường dùng

| Việc cần làm | DirectAdmin | cPanel/WHM |
|--------------|-------------|------------|
| Tạo gói hosting | Admin → **Package Manager** → Add Package | WHM → **Packages** → Add a Package |
| Tạo tài khoản khách | Admin → **Create User** (gán Package + domain) | WHM → **Create a New Account** (gán Package + domain) |
| Sửa quyền tài nguyên khách | Admin → **User Manager** → Edit | WHM → **Modify an Account** |
| Khóa tài khoản khách | Admin → **Suspend/Unsuspend Users** | WHM → **Suspend/Unsuspend an Account** |
| Xóa tài khoản khách | Admin → **Delete Users** | WHM → **Terminate a Multi-User Account** |
| Cài SSL cho 1 domain | User → **SSL Certificates** | cPanel → **SSL/TLS Status** (AutoSSL) |
| Cài WordPress | User → **Extended Features** → Installatron/Softaculous | cPanel → **WordPress Toolkit** hoặc Softaculous |
| Quản lý DNS Zone khách | User → **DNS Management** | cPanel → **Zone Editor** |
| Quản lý DNS Zone toàn server | Admin → **Admin Level DNS** | WHM → **DNS Functions → Edit DNS Zone** |
| Backup toàn tài khoản | User → **Backup/Restore** | cPanel → **Backup Wizard** (Full Account Backup) |
| Restore file `.tar.gz` tổng hợp | Admin → **Admin Backup/Transfer** | WHM → **Restore a Full Backup/cpmove File** |
| Cấu hình dịch vụ hệ thống (Apache/MySQL) | Admin → **Service Monitor** | WHM → **Service Manager / Restart Services** |
| Quản lý Firewall | Admin → CSF (cài thêm) | WHM → CSF (cài thêm, giống hệt) |

### 15.3 Điểm giống nhau gần như tuyệt đối

Vì bạn từng học kỹ CSF, Let's Encrypt, cron, DNS record types (SPF/DKIM/DMARC), Nginx reverse proxy — những kiến thức đó **dùng lại được 100%**, không đổi gì cả giữa DirectAdmin và cPanel/WHM:
- Cấu trúc DNS record (A, CNAME, MX, TXT) — giống hệt.
- Nguyên lý Let's Encrypt / AutoSSL — giống hệt, chỉ khác tên nút bấm.
- CSF Firewall — cài đặt và cấu hình giống hệt trên cả 2 panel.
- Cron job syntax (`* * * * *`) — giống hệt, chỉ khác giao diện nhập liệu.
- Nguyên lý chmod 755/644/600 cho WordPress — giống hệt.

### 15.4 Điểm cần "học lại" thật sự (không map được)

| Chủ đề | Vì sao khác |
|--------|-------------|
| Cấu trúc file cấu hình Apache | DirectAdmin dùng `/etc/httpd/conf/extra/httpd-vhosts.conf` theo template riêng; cPanel dùng `/etc/apache2/conf.d/userdata/` với hệ thống "userdata" riêng biệt |
| Package Extensions | DirectAdmin có "Custom Permissions" theo checkbox rất chi tiết; WHM Package đơn giản hơn nhưng có thêm **Feature List** (bật/tắt cả cụm tính năng UI) — khái niệm này DirectAdmin không có |
| WP Toolkit vs Softaculous/Installatron | Giao diện và luồng thao tác thực sự khác, cần làm quen tay |
| Tên gọi "Reseller" trong WHM | Reseller trong WHM vẫn dùng chung giao diện WHM (chỉ ẩn bớt menu), không có UI riêng biệt như DirectAdmin Reseller |

### 15.5 Lộ trình học đề xuất (giống cách bạn đã học DirectAdmin)

1. **Tuần 1:** Thuộc bảng ánh xạ ở 15.1–15.2, thực hành tạo Package + tạo Account trên WHM lab (xem mục 16).
2. **Tuần 2:** So sánh file cấu hình Apache thực tế giữa 2 hệ (đọc `/etc/apache2/conf.d/userdata/` trên cPanel, đối chiếu với vhost template DirectAdmin bạn đã biết).
3. **Tuần 3:** Thực hành troubleshooting chéo — dùng đúng bộ lỗi 500/403/404 bạn đã học ở DirectAdmin, áp lại vào cPanel (mục 11 ở trên) để thấy nguyên nhân/khắc phục gần như trùng khớp.
4. **Tuần 4:** Học sâu WHM-only: Backup Configuration cấp server, Restore cpmove file, Service Manager, Security Center.

---

## 16. Thực chiến: Tạo Package → Tạo User → SSL → WordPress

> Kịch bản: khách hàng "Công ty ABC" mua gói **Hosting Linux Business**, domain `abc-company.vn`. Server IP ví dụ `103.159.51.228`. Tương đương bài thực hành "tạo User Package + tạo User" bạn từng làm trên DirectAdmin.

### 16.1 Tạo Package trong WHM (làm 1 lần, tái sử dụng cho mọi khách cùng gói)

Đăng nhập `https://103.159.51.228:2087`

**WHM → Packages → Add a Package**

| Trường | Giá trị thực tế (gói Business Nhân Hòa) | Giải thích |
|--------|------------------------------------------|------------|
| Package Name | `Business_5GB` | Đặt tên có ý nghĩa, không dấu, không cách |
| Disk Quota | `5000` MB | Dung lượng ổ đĩa |
| Bandwidth | `Unlimited` | Băng thông |
| Max Email Accounts | `Unlimited` | |
| Max Email Lists | `10` | Mailing list |
| Max Databases | `10` | Số DB MySQL tối đa |
| Max FTP Accounts | `Unlimited` | |
| Max Subdomains | `Unlimited` | |
| Max Parked Domains (Alias) | `5` | |
| Max Addon Domains | `5` | |
| CGI Access | ✅ Tick | |
| Shell Access | ❌ Bỏ tick | Khách thường không cần SSH |
| Feature List | `default` | Trừ khi có gói riêng |

**Nhấn "Add"** → Package `Business_5GB` xuất hiện trong danh sách, tương đương "User Package" bạn đã tạo bên DirectAdmin.

> 💡 Thực tế Nhân Hòa: mỗi tier hosting (Bronze/Silver/Gold/Business...) tương ứng 1 Package trong WHM để tạo account nhanh, đồng nhất, không sai sót giữa các khách.

### 16.2 Tạo tài khoản (Account) cho khách — tương đương "Create User" bên DirectAdmin

**WHM → Account Functions → Create a New Account**

| Trường | Giá trị ví dụ | Ghi chú |
|--------|--------------|---------|
| Domain | `abc-company.vn` | Domain chính của khách |
| Username | `abccompany` | WHM tự gợi ý, có thể sửa |
| Password | *(dùng Password Generator)* | Copy lưu lại — mật khẩu **cPanel** của khách |
| Email | Email thật của khách | Nhận thông báo hệ thống |
| Package | Chọn `Business_5GB` | Package vừa tạo ở 16.1 |
| Nameservers | Giữ mặc định Nhân Hòa (vd `ns1.nhanhoa.com`, `ns2.nhanhoa.com`) | Nếu khách dùng DNS Nhân Hòa quản lý |
| IP Address | `Shared IP` (trừ khi khách yêu cầu Dedicated IP) | |

**Nhấn "Create"** → WHM tạo:
- 1 user Linux hệ thống tên `abccompany`
- 1 home directory `/home/abccompany`
- Database prefix `abccompany_`
- cPanel account riêng, đăng nhập độc lập tại `https://abc-company.vn:2083`

Kiểm tra log tạo account cuối trang — không có dòng đỏ (lỗi) là thành công.

### 16.3 Gửi thông tin đăng nhập cho khách

```
Domain:        abc-company.vn
cPanel URL:    https://abc-company.vn:2083 (hoặc https://103.159.51.228:2083)
Username:      abccompany
Password:      (mật khẩu đã tạo)

Nameserver cần trỏ tại nơi mua domain (nếu domain mua ngoài Nhân Hòa):
ns1.nhanhoa.com
ns2.nhanhoa.com
```

### 16.4 Xác minh DNS trước khi cài SSL (bắt buộc)

```bash
dig A abc-company.vn +short
```
Kết quả phải trả về `103.159.51.228`. Nếu chưa lên — **dừng lại**, AutoSSL sẽ luôn fail nếu DNS sai.

### 16.5 Cài SSL (đăng nhập cPanel thay khách)

Đăng nhập `https://abc-company.vn:2083`.

**cPanel → Security → SSL/TLS Status**
1. Tick `abc-company.vn` và `www.abc-company.vn`
2. Nhấn **Run AutoSSL** → chờ 1–3 phút → trạng thái ✅

Nếu fail lần đầu (rất hay gặp với domain mới trỏ), làm quy trình gỡ SSL cũ (xem mục 5.3), rồi Run AutoSSL lại.

**Bật Force HTTPS Redirect:**
cPanel → **Domains** → chọn `abc-company.vn` → bật toggle **Force HTTPS Redirect**

> ⚠️ Thứ tự bắt buộc: **cài SSL xong rồi mới bật Force HTTPS**. Bật trước khi cert active sẽ khiến site sập (redirect loop).

### 16.6 Cài WordPress cho khách

**cPanel → Software → WordPress Toolkit → Install WordPress**

| Trường | Giá trị | Lưu ý |
|--------|---------|-------|
| Domain | `abc-company.vn` | |
| Directory | Để trống | Cài ở root domain |
| WordPress version | Latest stable | |
| Admin username | `admin_abc` | **Không dùng `admin`** |
| Admin password | ≥12 ký tự, tạo mạnh | |
| Admin email | Email thật của khách | |

Nhấn **Install** → chờ 1–2 phút → kiểm tra `https://abc-company.vn/wp-admin`.

### 16.7 Checklist bàn giao khách

| Việc | Đã làm? |
|------|---------|
| DNS trỏ đúng IP | ☐ |
| SSL AutoSSL chạy thành công (ổ khóa xanh) | ☐ |
| Force HTTPS Redirect đã bật | ☐ |
| WordPress cài xong, đăng nhập `/wp-admin` được | ☐ |
| Đổi permalink WP sang "Post name" (SEO-friendly) | ☐ |
| Xóa theme/plugin mặc định không dùng | ☐ |
| Kiểm tra Mixed Content (nếu import site cũ) | ☐ |
| Gửi thông tin cPanel + WP admin cho khách | ☐ |
| Đặt lịch backup tự động | ☐ |

### 16.8 Backup tự động ngay sau khi bàn giao (nâng cao)

Đặt cron job backup định kỳ ở cấp server (chạy được trên VPS/WHM server, không phải trong cPanel user):
```bash
0 2 * * 0 /scripts/pkgacct abccompany
```
Hoặc bật **WHM → Backup → Backup Configuration** để hệ thống tự backup toàn server theo lịch, áp dụng cho toàn bộ account bao gồm `abccompany`.

---

## References

- [Hướng dẫn cài SSL trên Hosting cPanel – Nhân Hòa](https://wiki.nhanhoa.com/kb/huong-dan-cai-ssl-tren-hosting-cpanel/)
- [Hướng dẫn cài Next.js trên Hosting cPanel – Nhân Hòa](https://wiki.nhanhoa.com/kb/huong-dan-cai-next-js-tren-hosting-cpanel/)
- [Cách tạo Database trên Hosting dùng cPanel – Nhân Hòa](https://wiki.nhanhoa.com/kb/cach-tao-database-tren-hosting-dung-cpanel/)
- [cPanel – Cấu hình giới hạn thành viên gửi vào Mailing List – Nhân Hòa](https://wiki.nhanhoa.com/kb/cpanel-cau-hinh-gioi-han-thanh-vien-gui-vao-mailing-list/)
- [cPanel – Tính năng Restrict trong trang quản trị Mail – Nhân Hòa](https://wiki.nhanhoa.com/kb/cpanel-tinh-nang-restrict-trong-trang-quan-tri-mail/)
- [cPanel – Hướng dẫn tải Backup từ ShareHosting – Nhân Hòa](https://wiki.nhanhoa.com/kb/cpanel-huong-dan-tai-backup-tu-sharehosting/)
- [Hướng dẫn tạo tài khoản FTP trên Hosting cPanel – Nhân Hòa](https://wiki.nhanhoa.com/kb/huong-dan-tao-tai-khoan-ftp-tren-hosting-su-dung-cpanel/)
- [cPanel – Hướng dẫn tạo Cron Job – Nhân Hòa](https://wiki.nhanhoa.com/kb/cpanel-huong-dan-tao-cronjob-cpanel/)
- [cPanel Knowledge Base – docs.cpanel.net](https://docs.cpanel.net/knowledge-base/)
- [WHM Documentation – documentation.cpanel.net](https://documentation.cpanel.net/)
