# Hướng dẫn sử dụng cPanel (2083) & WHM (2087)

> **Hệ điều hành:** Ubuntu v22.04.5 
> **Phiên bản:** cPanel/WHM 134.0.43
> **Giao diện (Theme):** Jupiter
> **Domain thực hành:** `hieucute.id.vn`
> **Username cPanel:** `iamhieu`
> **VPS IP:** `103.159.51.228`
> **Truy cập cPanel:** `https://103.159.51.228:2083` hoặc `https://hieucute.id.vn:2083`
> **Truy cập WHM:** `https://103.159.51.228:2087`
> **Truy cập Webmail:** `https://103.159.51.228:2096`

---

## Mục lục

- [1. Tổng quan cổng truy cập](#1-tổng-quan-cổng-truy-cập)
- [2. WHM – Quản trị cấp Server (2087)](#2-whm--quản-trị-cấp-server-2087)
  - [2.1 Tạo Package (Gói hosting)](#21-tạo-package-gói-hosting)
  - [2.2 Tạo tài khoản mới](#22-tạo-tài-khoản-mới-create-a-new-account)
  - [2.3 Quản lý tài khoản hiện có](#23-quản-lý-tài-khoản-hiện-có)
  - [2.4 Feature List – Kiểm soát tính năng UI](#24-feature-list--kiểm-soát-tính-năng-ui-cho-từng-gói)
  - [2.5 Reseller – Tạo đại lý](#25-reseller--tạo-và-quản-lý-đại-lý)
  - [2.6 Quản lý dịch vụ hệ thống](#26-quản-lý-dịch-vụ-hệ-thống)
  - [2.7 Quản lý IP](#27-quản-lý-ip)
  - [2.8 DNS cấp server](#28-dns-cấp-server)
  - [2.9 SSL cấp server & AutoSSL](#29-ssl-cấp-server--autossl)
  - [2.10 Security Center](#210-security-center)
  - [2.11 EasyApache 4 – Cấu hình Apache/PHP](#211-easyapache-4--cấu-hình-apachephp-toàn-server)
  - [2.12 Tweak Settings – Tinh chỉnh server](#212-tweak-settings--tinh-chỉnh-hành-vi-server)
  - [2.13 MySQL/MariaDB Configuration](#213-mysqlmariadb-configuration)
  - [2.14 Cấu hình Backup (Backup Configuration)](#214-cấu-hình-backup-backup-configuration)
  - [2.15 Restore Full Backup (cpmove)](#215-restore-full-backup-cpmove)
  - [2.16 Bảng tóm tắt WHM vs cPanel](#216-bảng-tóm-tắt-whm-vs-cpanel)
- [3. Quản lý Tên miền & Web (cPanel)](#3-quản-lý-tên-miền--web-cpanel)
- [4. Quản lý File](#4-quản-lý-file)
- [5. Quản lý Tài khoản FTP](#5-quản-lý-tài-khoản-ftp)
- [6. Quản lý Cơ sở dữ liệu MySQL](#6-quản-lý-cơ-sở-dữ-liệu-mysql)
- [7. Bảo mật – Kích hoạt SSL](#7-bảo-mật--kích-hoạt-ssl)
- [8. Quản lý Email & Chống Spam](#8-quản-lý-email--chống-spam)
- [9. Mailing List](#9-mailing-list)
- [10. Cron Jobs](#10-cron-jobs)
- [11. Backup & Restore (cấp User)](#11-backup--restore-cấp-user)
- [12. Triển khai Next.js trên cPanel](#12-triển-khai-nextjs-trên-cpanel)
- [13. 5 lỗi phổ biến nhất](#13-5-lỗi-phổ-biến-nhất)
- [14.  Những lưu ý "chết người" – Dễ mất file/data](#14-️-những-lưu-ý-chết-người--dễ-mất-filedata)
- [15. Cấu hình nâng cao qua SSH (cho Admin)](#15-cấu-hình-nâng-cao-qua-ssh-cho-admin)
  - [15.1 Check log hệ thống & đăng nhập SSH](#151-check-log-hệ-thống--đăng-nhập-ssh)
  - [15.2 Check log tài khoản user cụ thể](#152-check-log-tài-khoản-user-cụ-thể)
  - [15.3 Check log Mail (Exim)](#153-check-log-mail-exim)
  - [15.4 Check log Web (Apache)](#154-check-log-web-apache)
  - [15.5 Check log cPanel & WHM](#155-check-log-cpanel--whm)
- [Bảng vị trí log quan trọng](#bảng-vị-trí-log-quan-trọng)
- [References](#references)

---

## 1. Tổng quan cổng truy cập

cPanel/WHM tách biệt hoàn toàn theo cổng — khác với DirectAdmin dùng chung cổng 2222 cho mọi cấp:

| Cổng | Dịch vụ | Dành cho | Địa chỉ thực tế |
|------|---------|----------|----------------|
| **2087** | WHM (HTTPS) | Root / Reseller — quản trị toàn server | `https://103.159.51.228:2087` |
| **2083** | cPanel (HTTPS) | User — quản lý 1 tài khoản hosting | `https://103.159.51.228:2083` |
| **2096** | Webmail (HTTPS) | Người dùng cuối — đọc/gửi email | `https://103.159.51.228:2096` |


---

## 2. WHM – Quản trị cấp Server (2087)

> WHM (Web Host Manager) là panel dành cho **root hoặc reseller**, quản lý toàn bộ server: nhiều tài khoản cPanel, dịch vụ hệ thống, firewall, cấu hình PHP/Apache, backup...

---

### 2.1 Tạo Package (Gói hosting)

Package định nghĩa giới hạn tài nguyên cho nhóm tài khoản. Tạo **một lần**, dùng lại cho mọi khách cùng gói — tương đương "User Package" bên DirectAdmin.

**Đường dẫn:** WHM → **Packages** → **Add a Package**

**Bước 1:** Điền tên Package — không dấu, không khoảng trắng, có ý nghĩa.

**Bước 2:** Cấu hình tài nguyên chuẩn gói Doanh Nghiệp Nhân Hòa:

| Trường | Giá trị ví dụ | Ghi chú |
|--------|--------------|---------|
| Package Name | `doanhnghiep_package` | Không dấu, không cách |
| Disk Quota | `6144` MB | 6 GB — khớp thực tế trên server |
| Bandwidth | `Unlimited` | Hoặc 50000 MB tùy gói |
| Max Email Accounts | `Unlimited` | |
| Max Mailing Lists | `10` | |
| Max Databases | `6` | |
| Max FTP Accounts | `Unlimited` | |
| Max Subdomains | `Unlimited` | |
| Max Parked Domains (Alias) | `Unlimited` | |
| Max Addon Domains | `3` | |
| CGI Access |  Bật | |
| Shell Access |  Tắt | Khách thường không cần SSH |
| Feature List | `default` | - |

**Bước 3:** Nhấn **Add**.

<img width="779" height="440" alt="image" src="https://github.com/user-attachments/assets/e72393d5-2559-4f29-990d-931c153ea453" />

>  Không thể xóa Package đang được gán cho tài khoản. Muốn xóa phải chuyển toàn bộ account sang Package khác trước.

---

### 2.2 Tạo tài khoản mới (Create a New Account)

**Đường dẫn:** WHM → **Account Functions** → **Create a New Account**

**Bước 1 – Domain Information:**

| Trường | Giá trị ví dụ | Quy tắc bắt buộc |
|--------|--------------|-----------------|
| Domain | `hieucute.id.vn` | FQDN, không có `http://` hay `/` |
| Username | `iamhieu` | **TUYỆT ĐỐI không đặt là `root`** |
| Password | *(dùng Password Generator)* | Xem bước 2 |
| Email | `admin@hieucute.id.vn` | Email nhận thông báo hệ thống |

>  **Quy tắc đặt Username:**
> - Viết liền, không dấu, không khoảng trắng, chỉ dùng chữ thường và số.
> - Tối đa 16 ký tự.
> - Tránh: `root`, `admin`, `test`, `mail`, `ftp` — những tên này có thể xung đột với user hệ thống.
> - Username trở thành tên thư mục home (`/home/iamhieu`) và là **prefix** của mọi database, FTP account.

**Bước 2 – Mật khẩu mạnh:**

Nhấn **Password Generator** → hệ thống tạo mật khẩu đạt **100/100 điểm** → nhấn **Use Password** → copy lưu lại.

>  Mật khẩu dưới 70/100 sẽ bị cPanel từ chối. Luôn dùng Password Generator.

**Bước 3 – Chọn Package:** Tại mục **Package** → chọn `doanhnghiep_package`.

**Bước 4 – Mail Routing Settings:**

Chọn **Automatically Detect Configuration**.

>  Tùy chọn này để cPanel tự xác định cấu hình email dựa trên DNS. Chỉ chọn **Local Mail Exchanger** khi chắc chắn server xử lý email nội bộ hoàn toàn và MX record đã trỏ đúng.

**Bước 5 – DNS Settings:**

| Tùy chọn | Trạng thái | Lý do |
|----------|-----------|-------|
|  **Enable DKIM** | Bật | Chữ ký số xác thực email — thiếu là vào Spam |
|  **Enable SPF** | Bật | Xác thực IP gửi mail — thiếu là vào Spam |
|  **Enable DMARC** | Bật | Chính sách email giả mạo — bảo vệ domain |

>  **Bắt buộc bật cả 3.** Kích hoạt sau khi tạo account phức tạp hơn nhiều so với bật ngay từ đầu.

**Bước 6:** Nhấn **Create** → quan sát log cuối trang. Không có dòng đỏ = thành công.

<img width="643" height="445" alt="image" src="https://github.com/user-attachments/assets/7730dbea-603f-494c-8905-6308434691f1" />

---

### 2.3 Quản lý tài khoản hiện có

#### Suspend / Unsuspend

**Đường dẫn:** WHM → **Account Functions** → **Manage Account Suspension**

>  Ghi rõ lý do suspend — khách sẽ thấy thông báo này khi truy cập website.

<img width="674" height="367" alt="image" src="https://github.com/user-attachments/assets/0c84f28b-ef12-4bd9-b20e-5519853f9bcf" />

#### Đổi Package / Nâng cấp tài khoản

**Đường dẫn:** WHM → **Account Functions** → **Modify an Account** → chọn username → đổi Package → **Save**.

<img width="724" height="434" alt="image" src="https://github.com/user-attachments/assets/65196393-aea0-4f5e-a3ef-d38265920e1b" />
>  Khi nâng Package (tăng tài nguyên), thay đổi có hiệu lực ngay lập tức — không cần restart dịch vụ.

#### Xem thông tin tổng hợp tất cả tài khoản

**Đường dẫn:** WHM → **Account Information** → **List Accounts**

Bảng hiển thị toàn bộ: domain, username, IP, Package, disk usage, bandwidth, trạng thái (active/suspended). Hữu ích khi cần kiểm tra nhanh server đang chạy bao nhiêu account, account nào dùng nhiều tài nguyên nhất.

#### Terminate (xóa vĩnh viễn)

**Đường dẫn:** WHM → **Account Functions** → **Terminate a Multi-User Account**

<img width="875" height="421" alt="image" src="https://github.com/user-attachments/assets/cc5d3c91-34a1-4b7e-9733-91300eeec2ce" />

>  **NGUY HIỂM** Xóa toàn bộ file, database, email — không thể hoàn tác. Bắt buộc tạo Full Backup trước.

---

### 2.4 Feature List – Kiểm soát tính năng UI cho từng gói

Feature List là khái niệm cho phép bật/tắt hẳn các nhóm tính năng hiển thị trong cPanel của user. Ví dụ: gói giá rẻ có thể tắt tính năng SSH Access, Git, Node.js để không gây nhầm lẫn cho khách.

**Đường dẫn:** WHM → **Packages** → **Feature Manager**

**Bước 1:** Nhấn **Add Feature List** → đặt tên (ví dụ: `basic_features`).

**Bước 2:** Tick/bỏ tick các tính năng muốn hiển thị với user thuộc Feature List này:

| Nhóm tính năng | Bật/tắt điển hình |
|---------------|------------------|
| File Manager |  Luôn bật |
| FTP Accounts |  Luôn bật |
| MySQL Databases |  Luôn bật |
| Email (toàn bộ) |  Luôn bật |
| SSL/TLS |  Nên bật |
| SSH/Shell Access |  Tắt với gói thường |
| Git Version Control |  Tắt với gói cơ bản |
| Setup Node.js App |  Tắt nếu không hỗ trợ |
| Softaculous |  Nên bật |

**Bước 3:** Nhấn **Save** → gán Feature List này vào Package .

>  Feature List chỉ ảnh hưởng đến **giao diện** — không ảnh hưởng đến tài nguyên thực tế. User vẫn có thể dùng SSH nếu có shell access ở cấp hệ thống, dù Feature List tắt hiển thị SSH trong cPanel.

---

### 2.5 Reseller – Tạo và quản lý đại lý

Reseller có thể tạo và quản lý các tài khoản cPanel dưới quyền mình, với giới hạn tài nguyên do Admin cấp.

#### Tạo Reseller Package

**Đường dẫn:** WHM → **Resellers** → **Reseller Center** → **Setup Reseller Accounts**

Hoặc: WHM → **Packages** → **Add a Package** → tích **Reseller Account** khi tạo.

#### Tạo tài khoản Reseller

**Bước 1:** WHM → **Account Functions** → **Create a New Account** → điền thông tin bình thường.

**Bước 2:** Sau khi tạo xong → WHM → **Resellers** → **Setup Reseller Accounts** → chọn username vừa tạo → tích **Make this account a Reseller** → **Save**.

**Bước 3:** WHM → **Resellers** → **Reseller Center** → chọn reseller → cấu hình **Resource Limits** (giới hạn băng thông, disk, số account họ có thể tạo).

#### Xem tài khoản do Reseller quản lý

**Đường dẫn:** WHM → **Account Information** → **List Accounts** → lọc theo **Owner** = tên reseller.

>  Reseller trong WHM **dùng chung giao diện WHM** (chỉ thấy menu trong phạm vi quyền được cấp). Khác DirectAdmin — Reseller có UI riêng biệt.

---

### 2.6 Quản lý dịch vụ hệ thống

#### Restart dịch vụ

**Đường dẫn:** WHM → **Restart Services**

<img width="518" height="182" alt="image" src="https://github.com/user-attachments/assets/ec338fd7-d8fa-43db-aaea-ccaf6a86d600" />

Danh sách dịch vụ có thể restart:

| Dịch vụ | Khi nào cần restart |
|---------|---------------------|
| **Apache Web Server** | Sau khi thay đổi cấu hình VirtualHost, EasyApache |
| **MySQL** | Sau khi sửa `my.cnf`, khi MySQL bị treo |
| **Exim Mail Server** | Sau khi sửa cấu hình mail, khi mail queue bị kẹt |
| **FTP Server (Pure-FTPd)** | Sau khi thay đổi cấu hình FTP |
| **cPanel** | Khi cPanel UI không phản hồi |
| **DNS (BIND)** | Sau khi sửa zone DNS quan trọng |

>  Restart Apache làm **gián đoạn toàn bộ website** trên server trong vài giây. Nên thực hiện ngoài giờ cao điểm hoặc dùng `graceful restart` khi có thể.

#### Kiểm tra trạng thái dịch vụ

**Đường dẫn:** WHM → **Server Status** → **Service Status**

Hiển thị màu xanh (running) / đỏ (stopped) cho từng dịch vụ. Dịch vụ nào đỏ cần restart hoặc điều tra log ngay.
<img width="800" height="439" alt="image" src="https://github.com/user-attachments/assets/bf9bae76-aa37-4773-9050-3aecac3bfc01" />

#### Giám sát tài nguyên realtime

**Đường dẫn:** WHM → **System Health** → **Process Manager**

Tương đương lệnh `top` — hiển thị CPU/RAM theo từng process, user. Hữu ích khi server chạy chậm bất thường để xác định process thủ phạm.
<img width="944" height="434" alt="image" src="https://github.com/user-attachments/assets/fa664dde-ea8d-4cf7-aed3-4ce734ee9d7a" />

**Đường dẫn:** WHM → **System Health** → **CPU and Memory Usage**

Biểu đồ lịch sử CPU/RAM — dùng để xác định thời điểm nào server bị quá tải.

---

### 2.7 Quản lý IP

**Đường dẫn:** WHM → **IP Functions**

#### Thêm IP mới vào server

WHM → **IP Functions** → **Add a New IP Address** → nhập địa chỉ IP và subnet mask → **Submit**.

>  IP mới cần được nhà cung cấp route về server trước. Thêm trong WHM chỉ là bước cấu hình phần mềm.

#### Gán IP riêng cho tài khoản

WHM → **Account Functions** → **Modify an Account** → chọn account → đổi **IP Address** từ Shared sang Dedicated IP → **Save**.

>  Dedicated IP cần thiết khi: cài SSL wildcard, chạy ứng dụng cần IP riêng, hoặc khách yêu cầu bảo mật cao hơn.

#### Xem danh sách IP và usage

WHM → **IP Functions** → **Show or Delete Current IP Addresses** — hiển thị từng IP đang dùng cho domain/account nào.

---

### 2.8 DNS cấp server

#### Sửa zone DNS bất kỳ domain

**Đường dẫn:** WHM → **DNS Functions** → **DNS Zone manager** → chọn domain → **Edit**

<img width="713" height="219" alt="image" src="https://github.com/user-attachments/assets/e4bb6d44-b5b5-48aa-9b14-196b96da657f" />

Cho phép sửa zone của mọi domain trên server — không giới hạn như Zone Editor trong cPanel user. Dùng khi cần sửa gấp DNS của khách mà không cần đăng nhập vào cPanel của họ.

#### Thêm zone DNS cho domain mới

**Đường dẫn:** WHM → **DNS Functions** → **Add DNS Zone** → nhập domain và IP → **Add**

>  Khi tạo account mới, WHM tự động tạo DNS zone. Mục này dùng khi cần thêm zone thủ công cho domain đặc biệt.

#### DNS Cluster (đồng bộ DNS nhiều server)

**Đường dẫn:** WHM → **Clusters** → **DNS Cluster**

Cấu hình đồng bộ zone DNS sang server NS phụ — dùng trong môi trường multi-server production. Đảm bảo nếu NS chính sập, NS phụ vẫn phân giải được.

---

### 2.9 SSL cấp server & AutoSSL

#### Cấu hình AutoSSL provider

**Đường dẫn:** WHM → **SSL/TLS** → **Manage AutoSSL**
<img width="948" height="431" alt="image" src="https://github.com/user-attachments/assets/159dedbd-238f-4c1c-b3d2-b0c6dcaaf0ab" />
Chọn provider:
- **Let's Encrypt** — miễn phí, tự động, thời hạn 90 ngày
- **Sectigo** — thương mại, được nhúng sẵn trong một số gói cPanel

Nhấn **Save** sau khi chọn provider.

#### Chạy AutoSSL cho toàn server

**Đường dẫn:** WHM → **SSL/TLS** → **Manage AutoSSL** → tab **Run AutoSSL for All Users** → **Run AutoSSL**

>  Nên chạy AutoSSL toàn server sau khi thêm nhiều account mới cùng lúc, hoặc sau khi DNS vừa được cập nhật.

#### Kiểm tra log AutoSSL

**Đường dẫn:** WHM → **SSL/TLS** → **Manage AutoSSL** → tab **Logs**

Hiển thị kết quả chi tiết từng domain: thành công, thất bại, lý do lỗi cụ thể.
<img width="532" height="341" alt="image" src="https://github.com/user-attachments/assets/a7fe47e6-85e1-47ae-b36b-c9f5d0a514ae" />

#### Cài SSL cho hostname server

**Đường dẫn:** WHM → **SSL/TLS** → **Install an SSL Certificate on a Domain**

Cài cert cho chính hostname của server (ví dụ `103-159-51-228.cprapid.com`) để giao diện WHM/cPanel hiển thị ổ khóa xanh.
<img width="470" height="256" alt="image" src="https://github.com/user-attachments/assets/3ffdf415-b216-4d8e-8dee-dca643988782" />

---

### 2.10 Security Center

**Đường dẫn:** WHM → **Security Center**

#### Shell Fork Bomb Protection

WHM → **Security Center** → **Shell Fork Bomb Protection** → **Enable**

<img width="823" height="182" alt="image" src="https://github.com/user-attachments/assets/2001877f-88cd-4026-8d66-bf9b870264c8" />

Giới hạn số tiến trình tối đa mỗi user có thể tạo — ngăn chặn script độc hại hoặc code lỗi tạo vô số process làm server sập.

#### Compiler Access

WHM → **Security Center** → **Compiler Access** → **Disable Compilers for Users**

<img width="386" height="208" alt="image" src="https://github.com/user-attachments/assets/1e6fa61a-59ee-4e95-89f9-e0c57ec96553" />

Tắt quyền dùng `gcc`, `g++` cho user thường. Ngăn hacker compile mã độc trực tiếp trên server nếu chiếm được tài khoản hosting.

>  Một số ứng dụng PHP cần compiler khi cài extension — cần bật lại tạm thời nếu gặp lỗi cài đặt.

#### Shell Access

WHM → **Account function** → **Manage Shell Access**

<img width="386" height="208" alt="image" src="https://github.com/user-attachments/assets/39894828-fa5e-4ce8-a8e8-7ed756b39e99" />

Kiểm soát tài khoản nào được phép SSH vào server. Có thể set thành `Disabled Shell` (jailed hoặc no shell) cho từng user.

#### cPHulk Brute Force Protection

**Đường dẫn:** WHM → **Security Center** → **cPHulk Brute Force Protection**

Tự động chặn IP đăng nhập sai nhiều lần vào cPanel/WHM/SSH. Cấu hình:
- Max failures before blocking: `5`
- Brute force period: `15` phút
- IP block time: `1` giờ

<img width="612" height="439" alt="image" src="https://github.com/user-attachments/assets/0498ab92-0c16-4e01-8ebf-8deec7c055d8" />

>  Xem log IP bị chặn và whitelist IP của admin để tránh tự chặn mình.

<img width="797" height="299" alt="image" src="https://github.com/user-attachments/assets/421c4973-8324-45d5-bb91-a85fd652a2ea" />

#### CSF (ConfigServer Security & Firewall)

**Đường dẫn:** WHM → **Plugins** → **ConfigServer Security & Firewall** (nếu đã cài)

CSF là firewall mạnh nhất cho cPanel — kiểm soát port, IP, rate limiting, chống DDoS cơ bản.

```bash
# Thêm IP vào whitelist (SSH)
csf -a 14.191.163.146 "Admin IP"

# Chặn IP
csf -d 1.2.3.4 "Spam source"

# Xem danh sách IP bị chặn
csf -l

# Restart CSF
csf -r
```

---

### 2.11 EasyApache 4 – Cấu hình Apache/PHP toàn server

**Đường dẫn:** WHM → **Software** → **EasyApache 4**

EasyApache cho phép cài đặt và cấu hình phiên bản PHP, Apache module, extension PHP cho **toàn server** — khác với MultiPHP Manager.

#### Cài thêm PHP version

**Bước 1:** WHM → **EasyApache 4** → **Customize** (bên cạnh profile **Currently Installed Packages**).

**Bước 2:** Tab **PHP Versions** → tick chọn phiên bản cần thêm (ví dụ: `PHP 8.1`, `PHP 8.2`, `PHP 8.3`).

**Bước 3:** Tab **PHP Extensions** → chọn extension cần (ví dụ: `php-redis`, `php-imagick`, `php-gmp`).

<img width="501" height="384" alt="image" src="https://github.com/user-attachments/assets/8b330f9e-395c-44f7-97ad-83ca9660bd5d" />

**Bước 4:** Nhấn **Review** → **Provision** → chờ quá trình build hoàn tất (5–15 phút).

>  **Provision sẽ restart Apache** — toàn bộ website gián đoạn trong quá trình build. Lên lịch thực hiện ngoài giờ cao điểm.

#### Kiểm tra PHP version đang available

**Đường dẫn:** WHM → **Software** → **MultiPHP Manager**
<img width="799" height="179" alt="image" src="https://github.com/user-attachments/assets/9b760e0f-26bd-4e57-a2fc-a30e83a3a097" />

Hiển thị PHP version mặc định của từng domain trên server. Admin có thể đổi PHP version cho bất kỳ account nào tại đây.

---

### 2.12 Tweak Settings – Tinh chỉnh hành vi server

**Đường dẫn:** WHM → **Server Configuration** → **Tweak Settings**

Đây là nơi tập trung hàng trăm cài đặt chi tiết cho toàn server. Các setting quan trọng nhất:

| Tab | Setting | Khuyến nghị |
|-----|---------|------------|
| **Mail** | Maximum hourly emails per domain | `200–500` (chống spam) |
| **Mail** | Apache SpamAssassin spam score threshold | `5` |
| **Mail** | Prevent "nobody" from sending mail |  Bật (chống PHP mail spam) |
| **System** | Maximum upload size | `128 MB` |
| **System** | PHP open_basedir Tweak |  Bật (bảo mật) |
| **PHP** | PHP-FPM (default for new accounts) |  Bật (hiệu năng) |
| **Security** | Require SSL for FTP connections |  Bật |
| **Security** | SMTP Restrictions |  Bật (chặn PHP gửi mail trực tiếp) |

>  Thay đổi Tweak Settings ảnh hưởng **toàn bộ server**. Đọc kỹ tooltip của từng setting trước khi thay đổi.

---

### 2.13 MySQL/MariaDB Configuration

**Đường dẫn:** WHM → **SQL Services** → **MySQL/MariaDB Configuration**

#### Tinh chỉnh my.cnf qua giao diện

WHM cung cấp giao diện để sửa các thông số quan trọng trong `my.cnf` mà không cần SSH:

| Thông số | Giá trị gợi ý | Mô tả |
|----------|--------------|-------|
| `innodb_buffer_pool_size` | 70% RAM | Buffer chính của InnoDB |
| `max_connections` | `150–300` | Số kết nối đồng thời tối đa |
| `query_cache_size` | `64M` | Cache kết quả query |
| `slow_query_log` | `ON` | Log query chậm để tối ưu |

**Đường dẫn:** WHM → **SQL Services** → **MySQL/MariaDB Configuration** → chỉnh → **Save**.

>  Sửa `my.cnf` sẽ **restart MySQL** — toàn bộ kết nối database bị ngắt trong vài giây. Tất cả website WordPress/Laravel/v.v trên server sẽ báo lỗi kết nối DB tạm thời.

#### Nâng cấp MariaDB version

**Đường dẫn:** WHM → **SQL Services** → **MySQL/MariaDB Upgrade**

>  Nâng cấp MariaDB là thao tác **không thể rollback** dễ dàng. Bắt buộc backup toàn bộ database trước.

---

### 2.14 Cấu hình Backup (Backup Configuration)

**Đường dẫn:** WHM → **Backup** → **Backup Configuration**

#### Bật Backup

Chuyển **Backup Status** sang **Enable** — nếu để Disable, mọi cấu hình bên dưới đều vô hiệu.

#### Loại Backup (Backup Type)

| Loại | Mô tả | Khi nào dùng |
|------|-------|-------------|
| **Compressed** | Nén thành `.tar.gz` — tiết kiệm dung lượng, tốn CPU khi nén/giải nén | Server ít account, disk backup hạn chế |
| **Uncompressed** | Giữ nguyên file — backup/restore nhanh, chiếm nhiều dung lượng | Server nhiều account, ưu tiên tốc độ |
| **Incremental (rsync)** | Chỉ sao lưu phần thay đổi — nhanh, tiết kiệm tối đa | Server dữ liệu lớn, backup hàng ngày |

<img width="448" height="373" alt="image" src="https://github.com/user-attachments/assets/0eb06be9-940b-458b-82f9-a91cba2e4b7e" />

>  Incremental hiệu quả nhất về thời gian và dung lượng nhưng **không tạo file `.tar.gz` đơn lẻ** để user tải về. Restore bắt buộc qua WHM.

#### Lịch chạy (Schedule)

Mỗi loại lịch có ô **"Retain X"** — số bản backup giữ lại trước khi xóa bản cũ nhất:

| Lịch | Cấu hình gợi ý | Retain |
|------|---------------|--------|
| **Daily** | Thứ 2–Thứ 6 | 7 (giữ 7 ngày gần nhất) |
| **Weekly** | Chủ nhật | 4 (giữ 4 tuần) |
| **Monthly** | Ngày 1 | 3 (giữ 3 tháng) |
<img width="469" height="380" alt="image" src="https://github.com/user-attachments/assets/d05269b8-cbc9-48aa-b397-faad1c236f82" />

#### Lựa chọn dữ liệu sao lưu

| Hạng mục | Quan trọng? | Ghi chú |
|----------|------------|---------|
| **Accounts** |  Cốt lõi | Toàn bộ dữ liệu khách hàng |
| **System Files** |  Nên bật | `/etc`, cấu hình Apache, Exim |
| **Suspended Accounts** |  Nên bật | Đảm bảo data của account bị khóa |
| **Bandwidth Data** |  Tùy chọn | Số liệu thống kê, không quan trọng |

#### Additional Destinations (Lưu trữ ngoại vi)

>  **TUYỆT ĐỐI không lưu backup trên cùng ổ đĩa với dữ liệu gốc.** Nếu ổ cứng hỏng, cả data lẫn backup đều mất. Áp dụng chiến lược **3-2-1**: 3 bản sao, 2 loại phương tiện, 1 bản off-site.

| Destination | Mức bảo mật | Ghi chú |
|-------------|------------|---------|
| **Local Directory** | Thấp | Partition/ổ đĩa riêng trên cùng server |
| **FTP** | Trung bình | Server backup riêng của doanh nghiệp |
| **SFTP/SCP** | Cao | Mã hóa, xác thực Private Key |
| **Amazon S3** | Cao nhất | Tiêu chuẩn production — tách hoàn toàn khỏi server gốc |

Cấu hình: nhập **Host, Username, Password/Key, Remote Directory, Port** → nhấn **Save and Validate Destination** để test kết nối.

#### Cảnh báo (Notifications)

Điền email nhận báo cáo backup sau mỗi phiên (thành công/thất bại).

>  Luôn bật Notifications. Tránh tình trạng hệ thống lỗi ngầm nhiều tuần, đến khi cần restore mới phát hiện không có backup.

---

### 2.15 Restore Full Backup (cpmove)

File Full Account Backup có dạng: `cpmove-username.tar.gz`

Cấu trúc bên trong:
```
cpmove-iamhieu.tar.gz
├── homedir/          ← Toàn bộ mã nguồn (public_html, mail...)
├── mysql/            ← Dump database (.sql)
├── meta/             ← Metadata cấu hình tài khoản
├── userdata/         ← Cấu hình VirtualHost Apache
└── homedir_paths.yaml
```

>  Đây là lý do file `cpmove` **không restore được qua cPanel user thường** — nó chứa metadata cấu hình hệ thống cần quyền root để ghi vào các file cấu hình của WHM.

#### Cách 1: Restore qua giao diện WHM

**Đường dẫn:** WHM → **Backup** → **Restore a Full Backup/cpmove File**

**Bước 1:** Chọn nguồn:
- **Restore from Home Directory:** file đã có sẵn trên server (upload qua SFTP vào `/home` trước).
- **Upload File:** upload trực tiếp qua trình duyệt — chỉ dùng với file nhỏ (tránh timeout).

**Bước 2:** Hệ thống tự nhận diện username từ tên file `cpmove-username.tar.gz`.

**Bước 3:** Chọn tùy chọn nâng cao:
- `Restore as Suspended` — khôi phục xong nhưng giữ tài khoản ở trạng thái khóa để kiểm tra trước khi public.
- `Overwrite existing account` — **CỰC KỲ CẨN THẬN**: xóa sạch dữ liệu hiện tại của account đó.

**Bước 4:** Nhấn **Submit** → theo dõi log → khi thấy thông báo thành công → Unsuspend nếu có tick bước 3.

#### Cách 2: Restore qua SSH (file lớn)

```bash
# Kiểm tra file tồn tại và kích thước
ls -lh /home/cpmove-iamhieu.tar.gz

# Kiểm tra tính toàn vẹn (không bị corrupt khi transfer)
gzip -t /home/cpmove-iamhieu.tar.gz && echo "File OK"

# Restore bằng script cPanel
/scripts/restorepkg --skipaccount iamhieu
```

`--skipaccount` — bỏ qua nếu account đã tồn tại (không ghi đè). Bỏ flag này nếu muốn ghi đè.

---

## 3. Quản lý Tên miền & Web (cPanel)

### 3.1 Trỏ tên miền về VPS

Trước khi làm bất kỳ thao tác nào, DNS của `hieucute.id.vn` phải trỏ đúng về IP VPS.

Đăng nhập quản lý tên miền tại Nhân Hòa → Zone DNS:

| Host | Type | Value | TTL |
|------|------|-------|-----|
| `@` | A | `103.159.51.228` | 300 |
| `www` | CNAME | `hieucute.id.vn.` | 300 |
| `mail` | A | `103.159.51.228` | 300 |

```bash
dig A hieucute.id.vn +short
# Kết quả đúng: 103.159.51.228
```
<img width="370" height="136" alt="image" src="https://github.com/user-attachments/assets/10993091-eb6b-48db-a593-8a49b37195cc" />

>  Không tiếp tục bất kỳ bước nào nếu DNS chưa đúng. AutoSSL luôn thất bại nếu DNS sai.

---

### 3.2 Thêm Domain mới  
**Đường dẫn:** cPanel → **Domains** → **Domains** → **Create A New Domain**

| Trường | Giá trị | Ghi chú |
|--------|---------|---------|
| Domain | `shop.hieucute.id.vn` | FQDN, không có `www` hay `http://` |
| Document Root | `/home/iamhieu/shop.hieucute.id.vn` | Tự động điền, có thể sửa |
| Share document root | **BỎ TICK** | Xem cảnh báo |

>  **CẢNH BÁO Share document root:** Nếu tick ô này, domain mới hiển thị **y hệt nội dung** domain chính. Đây là nhầm lẫn rất khó hoàn tác sau khi đã có dữ liệu. **Luôn bỏ tick** trừ khi có chủ đích mirror site.

<img width="792" height="152" alt="image" src="https://github.com/user-attachments/assets/4981f85c-1c05-46d9-abcb-28a3c5d5a32d" />

Sau khi tạo → thêm A record tại DNS:
```
shop.hieucute.id.vn.   A   103.159.51.228   TTL 300
```

---

### 3.3 Cài đặt WordPress qua WordPress Management

**Đường dẫn:** cPanel → sidebar trái → **WordPress Management**

<img width="894" height="430" alt="image" src="https://github.com/user-attachments/assets/569e3e8e-ec7c-4c3a-9fa3-7fd436f87aef" />

| Trường | Giá trị | Quy tắc |
|--------|---------|---------|
| Domain | `hieucute.id.vn` | - |
| Directory | Để trống | Cài tại root domain |
| WordPress version | Latest stable | Luôn phiên bản mới nhất |
| Admin username | `admin_hieucute` | **Không dùng `admin`** |
| Admin password | *(Password Generator)* | Tối thiểu 12 ký tự |
| Admin email | `admin@hieucute.id.vn` | Email thật |

Nhấn **Install** → chờ 1–2 phút → truy cập `https://hieucute.id.vn/wp-admin`.

---

## 4. Quản lý File

### 4.1 Upload file lên public_html
**Cách 1: File Manager**
cPanel → **Files** → **File Manager** → điều hướng vào `public_html` → **Upload**.
<img width="945" height="419" alt="image" src="https://github.com/user-attachments/assets/777503fb-e078-4bd2-8e19-1af347c924eb" />

**Cách 2: SFTP qua FileZilla (khuyến nghị)**
```
Host: 103.159.51.228  |  Port: 22  |  Username: iamhieu  |  Password: mật khẩu cPanel
```

### 4.2 Giải nén file

Chuột phải file `.zip`/`.tar.gz` → **Extract** → xác nhận thư mục đích.

>  Sau giải nén hay bị lồng thư mục: `public_html/public_html/`. Khắc phục: **Select All** → **Move** về đúng `/home/iamhieu/public_html`.

### 4.3 Phân quyền file (Chmod)

**Quyền chuẩn cho WordPress:**

| Đối tượng | Số | Lý do |
|-----------|-----|-------|
| Thư mục `public_html` | `755` | Apache cần execute |
| File `.php`, `.html` | `644` | Đọc được, không execute |
| File `wp-config.php` | `600` | Chỉ owner đọc |
| Thư mục `wp-content/uploads` | `775` | WP cần ghi |

<img width="935" height="419" alt="image" src="https://github.com/user-attachments/assets/5b3384ae-7d4d-469f-9aaf-b514e891d621" />

```bash
find ~/public_html -type d -exec chmod 755 {} \;
find ~/public_html -type f -exec chmod 644 {} \;
chmod 600 ~/public_html/wp-config.php
```

---

## 5. Quản lý Tài khoản FTP

**Đường dẫn:** cPanel → **Files** → **FTP Accounts**

| Trường | Giá trị | Ghi chú |
|--------|---------|---------|
| Log In | `ftpdev` | Đầy đủ: `ftpdev@hieucute.id.vn` |
| Password | *(Password Generator 100/100)* | |
| Directory | `/home/iamhieu/public_html` | Giới hạn truy cập |
| Quota | `500 MB` | |

Kết nối FileZilla:
```
Host: 103.159.51.228  |  Port: 22 (SFTP)  |  Username: ftpdev@hieucute.id.vn
```

>  Ưu tiên SFTP (port 22). Không bật **Anonymous FTP** trên production.

---

## 6. Quản lý Cơ sở dữ liệu MySQL

### 6.1 Tạo Database qua Database Wizard

**Đường dẫn:** cPanel → **Databases** → **Database Wizard**

**Bước 1:** Nhập `wpdb` → **Next Step** → Tên thực tế: `iamhieu_wpdb`.

**Bước 2:** Username `wpuser` + Password Generator → **Create User** → Tên thực tế: `iamhieu_wpuser`.

**Bước 3:** Tích **ALL PRIVILEGES** → **Next Step**.

**Bước 4:** Lưu thông tin kết nối:
```
DB_NAME:     iamhieu_wpdb
DB_USER:     iamhieu_wpuser
DB_PASSWORD: (mật khẩu vừa tạo)
DB_HOST:     localhost
```
<img width="793" height="130" alt="image" src="https://github.com/user-attachments/assets/61fc80d9-f565-4824-a20a-947f515c2b96" />

---

## 7. Bảo mật – Kích hoạt SSL

### 7.1 AutoSSL

**Đường dẫn:** cPanel → **Security** → **SSL/TLS Certificates** → tab **Domains** → tick domain → **Run AutoSSL**

Chờ 1–3 phút → trạng thái  Let's Encrypt.

### 7.2 Cài SSL có sẵn

cPanel → **Security** → **SSL/TLS Certificates** → **Install an SSL Certificate** → paste Certificate + Private Key + CA Bundle → **Install**.

### 7.3 Gỡ SSL cũ khi AutoSSL thất bại

1. **SSL/TLS Certificates** → **Private Keys** → xóa key cũ.
2. **SSL/TLS Certificates** → **Certificates** → **Uninstall** → **Delete Key**.
3. **File Manager** → xóa thư mục `.well-known` trong `public_html`.
4. Tick tất cả domain → **Run AutoSSL** → chờ 5–10 phút.

### 7.4 Force HTTPS Redirect

cPanel → **Domains** → chọn domain → bật **Force HTTPS Redirect**.

>  **Thứ tự bắt buộc:** Cài SSL xong, xác nhận cert hợp lệ **trước** khi bật Force HTTPS. Bật trước khi cert active sẽ gây redirect loop — website sập hoàn toàn.

---

## 8. Quản lý Email & Chống Spam

### 8.1 Tạo tài khoản Email

**Đường dẫn:** cPanel → **Email** → **Email Accounts** → **Create (+)**

| Trường | Giá trị | Ghi chú |
|--------|---------|---------|
| Username | `info` | Địa chỉ: `info@hieucute.id.vn` |
| Domain | `hieucute.id.vn` | |
| Password | *(Password Generator 100/100)* | |
| Storage Space | `1024 MB` | |

Webmail: `https://103.159.51.228:2096` — đăng nhập bằng `info@hieucute.id.vn` (không chỉ `info`).

<img width="799" height="437" alt="image" src="https://github.com/user-attachments/assets/fbd2a140-c4d5-4336-89b2-374a0f403260" />

### 8.2 Restrict – Giới hạn email gửi đi

cPanel → **Email** → **Email Accounts** → chọn email → **Manage** → mục **Restrictions** → đặt giới hạn gửi mỗi giờ.

<img width="532" height="370" alt="image" src="https://github.com/user-attachments/assets/2d3748d8-66fa-4064-bf7d-f2b2eae42c59" />

### 8.3 SPF, DKIM, DMARC

**Đường dẫn:** cPanel → **Email** → **Email Deliverability** → nhấn **Repair** để tự động sửa.

| Record | Bản ghi mẫu |
|--------|-------------|
| SPF | `v=spf1 ip4:103.159.51.228 ~all` |
| DKIM | Lấy từ Email Deliverability → copy vào DNS Zone |
| DMARC (giai đoạn 1) | `v=DMARC1; p=none; rua=mailto:admin@hieucute.id.vn` |
| DMARC (giai đoạn 2) | `v=DMARC1; p=quarantine; pct=100; rua=mailto:admin@hieucute.id.vn` |
| DMARC (giai đoạn 3) | `v=DMARC1; p=reject; pct=100; rua=mailto:admin@hieucute.id.vn` |

<img width="370" height="433" alt="image" src="https://github.com/user-attachments/assets/0b002e7b-de17-45d3-8e5e-621f702033f5" />

```bash
dig TXT hieucute.id.vn | grep spf
dig TXT default._domainkey.hieucute.id.vn
dig TXT _dmarc.hieucute.id.vn
```

---

## 9. Mailing List

**Đường dẫn:** cPanel → **Email** → **Mailing Lists**

Tạo list → **Manage** → giao diện Mailman → **Privacy options** → **Sender filters** → **Restrict posting privilege to list members: Yes**.

<img width="940" height="461" alt="image" src="https://github.com/user-attachments/assets/bd9708d0-1d94-4bd1-a6ea-7a94914c55fb" />

---

## 10. Cron Jobs

**Đường dẫn:** cPanel → **Advanced** → **Cron Jobs**

| Cú pháp | Ý nghĩa |
|---------|---------|
| `0 2 * * *` | Mỗi ngày 2:00 sáng |
| `0 2 * * 0` | Mỗi Chủ nhật 2:00 sáng |
| `*/5 * * * *` | Mỗi 5 phút |

**Ví dụ thực tế:**
```bash
# WordPress cron
*/15 * * * * wget -q -O /dev/null "https://hieucute.id.vn/wp-cron.php?doing_wp_cron"

# Backup database 2:00 sáng
0 2 * * * mysqldump -u iamhieu_wpuser -p'password' iamhieu_wpdb | gzip > /home/iamhieu/backups/db_$(date +\%Y-\%m-\%d).sql.gz

# Xóa backup cũ hơn 7 ngày
0 3 * * * find /home/iamhieu/backups -type f -mtime +7 -delete
```

>  Luôn dùng đường dẫn **tuyệt đối** trong lệnh cron. Thêm `MAILTO=admin@hieucute.id.vn` ở đầu crontab để nhận thông báo lỗi.

---

## 11. Backup & Restore (cấp User)

### 11.1 Full Backup

**Đường dẫn:** cPanel → **Files** → **Backup Wizard** → **Back Up** → **Full Account Backup** → chọn đích → điền email → **Generate Backup**.

Tải về: File Manager → tìm file `.tar.gz` trong home directory → **Download**.

### 11.2 Backup từng phần

- Database: **Backup** → **Download a MySQL Database Backup**
- Home Directory: **Backup** → **Download a Home Directory Backup**

### 11.3 Restore

**Restore Source Code:** Backup Wizard → Restore → Home Directory → upload `.tar.gz` → sau đó vào File Manager → folder `HomeDir/public_html` → Select All → Move về `/home/iamhieu/public_html`.

**Restore Database:** phpMyAdmin → chọn database `iamhieu_wpdb` → **Import** → chọn file SQL.

>  Restore ghi đè dữ liệu hiện tại. Full Account Backup chỉ restore được qua WHM (xem mục 2.15).

---

## 12. Triển khai Next.js trên cPanel

**Đường dẫn:** cPanel → **Software** → **Setup Node.js App** → **Create Application**

| Trường | Giá trị |
|--------|---------|
| Node.js version | `18.x` hoặc `20.x` |
| Application mode | `Production` |
| Application root | `nextjs_app` |
| Application URL | `hieucute.id.vn` |
| Startup file | `app.js` |

Cài qua Terminal:
```bash
cd ~/nextjs_app && rm -rf *
npx create-next-app@latest . --yes
npm run build
```

File `app.js`:
```javascript
const { createServer } = require('http');
const { parse } = require('url');
const next = require('next');
const app = next({ dev: false, hostname: 'localhost', port: process.env.PORT || 3000 });
const handle = app.getRequestHandler();
app.prepare().then(() => {
    createServer(async (req, res) => {
        try { await handle(req, res, parse(req.url, true)); }
        catch (err) { res.statusCode = 500; res.end('error'); }
    }).listen(process.env.PORT || 3000);
});
```

Sau khi tạo xong: **Stop App** → **Start App**.

---

## 13. 5 lỗi phổ biến nhất

### Lỗi 1 – 500 Internal Server Error

**Log:** cPanel → **Metrics** → **Errors** | `~/public_html/error_log`

```bash
mv ~/public_html/.htaccess ~/public_html/.htaccess_bak
find ~/public_html -type f -name "*.php" -exec chmod 644 {} \;
find ~/public_html -type d -exec chmod 755 {} \;
```

Đổi PHP version: cPanel → **Software** → **MultiPHP Manager**.

---

### Lỗi 2 – Email vào Spam / bị từ chối

**Log:** cPanel → **Email** → **Email Deliverability** | **Track Delivery**

1. Email Deliverability → **Repair**.
2. Kiểm tra blacklist: [mxtoolbox.com/blacklists.aspx](https://mxtoolbox.com/blacklists.aspx)
3. Liên hệ Nhân Hòa tạo PTR record: `103.159.51.228` → `mail.hieucute.id.vn`.

---

### Lỗi 3 – AutoSSL thất bại

```bash
dig A hieucute.id.vn +short          # Phải trả về 103.159.51.228
curl -I http://hieucute.id.vn         # Port 80 phải phản hồi
dig CAA hieucute.id.vn                # Kiểm tra CAA
```


---

### Lỗi 4 – WordPress "Error establishing a database connection"

File Manager → `public_html/wp-config.php` → **Edit**:
```php
define( 'DB_NAME',     'iamhieu_wpdb' );   // Phải có prefix "iamhieu_"
define( 'DB_USER',     'iamhieu_wpuser' ); // Phải có prefix "iamhieu_"
define( 'DB_PASSWORD', 'correct_password' );
define( 'DB_HOST',     'localhost' );
```

---

### Lỗi 5 – Upload file 413 / Timeout

cPanel → **Software** → **MultiPHP INI Editor** → chọn domain → sửa:

| Directive | Giá trị |
|-----------|---------|
| `upload_max_filesize` | `128M` |
| `post_max_size` | `128M` |
| `max_execution_time` | `300` |
| `memory_limit` | `256M` |

---

## 14.  Những lưu ý "chết người" – Dễ mất file/data

> Đây là tổng hợp các thao tác **không thể hoàn tác** hoặc **gây mất dữ liệu không cảnh báo** trong cPanel/WHM. Đọc kỹ trước khi thực hiện.

---

### 14.1 Terminate Account trong WHM — Xóa vĩnh viễn, không hỏi lại

**WHM → Account Functions → Terminate a Multi-User Account**

Nhấn **Terminate** là toàn bộ dữ liệu bay: file website, tất cả database, email, DNS zone, cấu hình. **Không có thùng rác, không có undo.**

>  **Bắt buộc:** Tạo Full Backup qua WHM → Backup → Generate Backup và xác nhận file backup tải về/lưu thành công **trước khi** nhấn Terminate. Đừng tin vào bộ nhớ — kiểm tra file backup thực tế.

---

### 14.2 Restore với "Overwrite existing account" — Xóa sạch data hiện tại

**WHM → Backup → Restore a Full Backup/cpmove File → tick "Overwrite existing account"**

Tùy chọn này **xóa toàn bộ dữ liệu hiện tại** của tài khoản đó rồi ghi bản cũ vào. Nếu bản backup đã cũ 3 tháng và website hiện tại đang có bài viết/đơn hàng mới — tất cả sẽ mất.

>  Luôn backup bản hiện tại **trước** khi restore bản cũ hơn. Thứ tự: Backup → Restore, không bao giờ Restore thẳng.

---

### 14.3 phpMyAdmin — DROP TABLE / DROP DATABASE không hỏi lại (hoặc hỏi rất nhẹ)

Trong phpMyAdmin, chọn database → tab **Operations** → **Drop the database** hoặc chọn bảng → **Drop** ở thanh hành động — thao tác này thực hiện ngay, dialog xác nhận rất nhỏ và dễ bỏ qua.

>  Export database ra file `.sql` trước khi làm bất kỳ thay đổi cấu trúc nào. Shortcut an toàn: **Export** → **Quick** → **Go** → lưu file về máy.

---

### 14.4 File Manager — Xóa file/thư mục bỏ qua Trash

Trong File Manager, chọn file → **Delete** → mặc định cPanel **bỏ qua Trash** và xóa vĩnh viễn luôn. Checkbox "Skip the trash" thường được tick sẵn.

>  Trước khi xóa file/thư mục quan trọng: nén lại thành `.zip` và tải về máy, hoặc move sang thư mục tạm trước. Không click Delete mà không có backup.

---

### 14.5 Giải nén file .zip đè lên thư mục hiện có

Khi giải nén file `.zip` vào thư mục `public_html` đang có website, cPanel **ghi đè file trùng tên mà không cảnh báo**. File `index.php`, `wp-config.php`, `.htaccess` bị ghi đè là website sập hoặc mất cấu hình.

>  Luôn backup thư mục hiện tại trước khi giải nén bản mới. Hoặc giải nén vào thư mục tạm → kiểm tra → mới move file vào production.

---

### 14.6 Force HTTPS Redirect bật trước khi có SSL hợp lệ

cPanel → **Domains** → **Force HTTPS Redirect** → bật khi cert chưa active hoặc cert đã hết hạn → website rơi vào **infinite redirect loop** — người dùng không thể truy cập, kể cả admin.

>  Thứ tự bắt buộc: (1) Cài SSL + xác nhận ổ khóa xanh, (2) Bật Force HTTPS.
> Nếu đã bật nhầm: SSH vào server → sửa/xóa `.htaccess` tạm thời, hoặc WHM → Edit DNS Zone → tắt HTTPS redirect ở cấp server.

---

### 14.7 EasyApache 4 Provision — Restart Apache toàn server

**WHM → Software → EasyApache 4 → Provision**

Quá trình Provision build lại Apache/PHP **bắt buộc restart Apache** — toàn bộ website trên server gián đoạn từ vài giây đến vài phút tùy cấu hình.

>  Lên lịch thực hiện vào khung giờ thấp điểm (1:00–4:00 sáng). Thông báo trước cho khách hàng nếu cần. Không Provision giữa giờ cao điểm.

---

### 14.8 Tweak Settings sai cấu hình mail → Mất toàn bộ email đến

**WHM → Server Configuration → Tweak Settings → Mail → "Mail delivery policy"**

Nếu vô tình set **"Reject mail if Exim cannot verify the sender's domain"** trong khi DNS của domain chưa cấu hình đúng → server từ chối nhận email, không bounce, không thông báo người gửi. Email biến mất hoàn toàn.

>  Sau khi thay đổi bất kỳ setting nào liên quan đến mail: test gửi/nhận ngay với cả Gmail và Outlook. Kiểm tra `/var/log/exim_mainlog` xem có dòng REJECT hay DISCARD không.

---

### 14.9 Xóa Private Key SSL trước khi có cert mới

**cPanel → Security → SSL/TLS Certificates → Private Keys → Delete**

Private Key gắn với một cert cụ thể. Xóa Key cũ trong khi cert tương ứng vẫn đang được dùng → website báo lỗi SSL ngay lập tức.

>  Chỉ xóa Private Key khi đã xác nhận cert mới đã được cài và hoạt động. Thứ tự đúng: Cài cert mới và test → Xóa key cũ.

---

### 14.10 Reset mật khẩu cPanel không lưu lại → Mất quyền truy cập

WHM → **Modify an Account** → đổi mật khẩu tài khoản → mật khẩu thay đổi ngay lập tức. Nếu không lưu lại và không gửi cho khách → không ai đăng nhập được vào cPanel nữa.

>  Luôn copy mật khẩu mới vào password manager **trước** khi nhấn Save. Gửi thông tin đăng nhập cho khách ngay sau khi reset.

---

### 14.11 Terminate Account khi backup đang chạy dở

Nếu nhấn Terminate trong lúc quá trình backup chưa hoàn tất → file `.tar.gz` bị corrupt, không restore được.

>  Kiểm tra WHM → Backup → đảm bảo không có backup job nào đang chạy trước khi Terminate.

---

### 14.12 Sửa DNS Zone xóa nhầm bản ghi MX

**cPanel → Zone Editor → xóa nhầm MX record** → toàn bộ email đến của domain bị mất, không bounce về người gửi ngay lập tức (TTL DNS còn hiệu lực).

>  Screenshot hoặc export Zone file trước khi sửa DNS. Sau khi sửa, kiểm tra ngay bằng `dig MX hieucute.id.vn`.

---

## 15. Cấu hình nâng cao qua SSH (cho Admin)

> Các lệnh dưới đây cần chạy với quyền `root` trên server.

---

### 15.1 Check log hệ thống & đăng nhập SSH

**File log:** `/var/log/auth.log` (Ubuntu) | `/var/log/secure` (CentOS)

```bash
# Xem lần đăng nhập gần nhất của tất cả user
lastlog

# Lịch sử đăng nhập thành công (kèm IP)
last -a

# Lọc đăng nhập SSH thất bại — phát hiện brute-force
grep "Failed password" /var/log/auth.log

# Theo dõi realtime ai đang nâng quyền root
tail -f /var/log/auth.log | grep -i "sudo\|su:"
```

>  Nếu thấy 1 IP lặp lại hàng trăm lần trong Failed password: `csf -d <IP> "brute force"`.

---

### 15.2 Check log tài khoản user cụ thể

```bash
# Tiến trình đang chạy của user iamhieu
ps -u iamhieu -f

# Giám sát realtime CPU/RAM của user iamhieu
top -u iamhieu

# Lịch sử lệnh đã gõ
cat /home/iamhieu/.bash_history

# Ép ghi history hiện tại rồi xem 50 dòng gần nhất
history -a && tail -50 /home/iamhieu/.bash_history
```

---

### 15.3 Check log Mail (Exim)

**File log chính:** `/var/log/exim_mainlog`

```bash
# Theo dõi mail log realtime
tail -f /var/log/exim_mainlog

# Lọc mail liên quan đến địa chỉ cụ thể
grep "admin@hieucute.id.vn" /var/log/exim_mainlog | tail -50

# Xem mail đang kẹt trong queue
exim -bp | grep "admin@hieucute.id.vn"

# Tìm mail trong queue theo người gửi
exiqgrep -f admin@hieucute.id.vn

# Xóa toàn bộ mail trong queue (cẩn thận!)
exim -bp | exiqgrep -i | xargs exim -Mrm
```

---

### 15.4 Check log Web (Apache)

**File log:** `/usr/local/apache/logs/` hoặc `/var/log/apache2/`

```bash
# Theo dõi log lỗi domain cụ thể realtime
tail -f /usr/local/apache/logs/hieucute.id.vn-ssl_log

# Xem 100 dòng lỗi gần nhất trong home directory của user
tail -100 /home/iamhieu/logs/hieucute.id.vn/error.log

# Lọc các lỗi HTTP 500/502/503
grep -i "500\|502\|503" /usr/local/apache/logs/hieucute.id.vn-ssl_log | tail -50

# Log Nginx (nếu dùng Nginx reverse proxy)
tail -f /var/log/nginx/error.log
```

---

### 15.5 Check log cPanel & WHM

**File log:** `/usr/local/cpanel/logs/`

```bash
# Theo dõi realtime ai đăng nhập vào cPanel/WHM
tail -f /usr/local/cpanel/logs/login_log

# Lọc lịch sử truy cập của user cụ thể
grep "iamhieu" /usr/local/cpanel/logs/access_log | tail -30

# Log lỗi hệ thống cPanel (không phải lỗi website)
tail -f /usr/local/cpanel/logs/error_log

# Log AutoSSL
tail -f /usr/local/cpanel/logs/autossl_log
```

---

## Bảng vị trí log quan trọng

| Log | Vị trí trong cPanel 134 | Vị trí SSH |
|-----|------------------------|-----------|
| Error log website | **Metrics → Errors** | `~/public_html/error_log` |
| Access log website | **Metrics → Raw Access** | `/usr/local/apache/logs/<domain>_log` |
| Email gửi/nhận | **Email → Track Delivery** | `/var/log/exim_mainlog` |
| SSL/AutoSSL | **Security → SSL/TLS Certificates** | `/usr/local/cpanel/logs/autossl_log` |
| Bandwidth | **Metrics → Bandwidth** | — |
| Đăng nhập SSH | — | `/var/log/auth.log` (Ubuntu) |
| cPanel login | — | `/usr/local/cpanel/logs/login_log` |
| cPanel error | — | `/usr/local/cpanel/logs/error_log` |
| MySQL error | — | `/var/lib/mysql/error.log` |
| Apache error (server) | — | `/var/log/apache2/error_log` |
| WHM AutoSSL detail | WHM → **SSL/TLS → Manage AutoSSL → Logs** | `/usr/local/cpanel/logs/autossl_log` |

---

## References

- [Nhân Hòa – Hướng dẫn cài SSL trên Hosting cPanel](https://wiki.nhanhoa.com/kb/huong-dan-cai-ssl-tren-hosting-cpanel/)
- [Nhân Hòa – Hướng dẫn cài Next.js trên Hosting cPanel](https://wiki.nhanhoa.com/kb/huong-dan-cai-next-js-tren-hosting-cpanel/)
- [Nhân Hòa – Cách tạo Database trên Hosting dùng cPanel](https://wiki.nhanhoa.com/kb/cach-tao-database-tren-hosting-dung-cpanel/)
- [Nhân Hòa – cPanel: Giới hạn thành viên gửi vào Mailing List](https://wiki.nhanhoa.com/kb/cpanel-cau-hinh-gioi-han-thanh-vien-gui-vao-mailing-list/)
- [Nhân Hòa – cPanel: Tính năng Restrict trong quản trị Mail](https://wiki.nhanhoa.com/kb/cpanel-tinh-nang-restrict-trong-trang-quan-tri-mail/)
- [Nhân Hòa – Hướng dẫn tải Backup từ ShareHosting](https://wiki.nhanhoa.com/kb/cpanel-huong-dan-tai-backup-tu-sharehosting/)
- [Nhân Hòa – Hướng dẫn tạo tài khoản FTP trên cPanel](https://wiki.nhanhoa.com/kb/huong-dan-tao-tai-khoan-ftp-tren-hosting-su-dung-cpanel/)
- [Nhân Hòa – Hướng dẫn tạo Cron Job cPanel](https://wiki.nhanhoa.com/kb/cpanel-huong-dan-tao-cronjob-cpanel/)
- [cPanel Knowledge Base](https://docs.cpanel.net/knowledge-base/)
- [WHM Documentation](https://documentation.cpanel.net/)
