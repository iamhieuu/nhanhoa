<img width="1266" height="649" alt="image" src="https://github.com/user-attachments/assets/dccf9124-44ef-4df5-a68e-746eb887fba5" /># Hướng dẫn sử dụng Plesk Obsidian (Admin & Customer)

> **Hệ điều hành:** Linux (Ubuntu/CentOS/AlmaLinux) hoặc Windows Server
> **Phiên bản:** Plesk Obsidian (bản mới nhất, cập nhật liên tục theo chu kỳ)
> **Truy cập Plesk (Admin/Reseller/Customer dùng chung 1 cổng):** `https://103.179.190.194:8443` hoặc `https://<domain>:8443`
> **Truy cập Webmail:** thường qua `https://webmail.<domain>` hoặc link **Webmail** ngay trong menu Mail của từng domain (không có cổng cố định riêng như cPanel 2096)

---

## Mục lục

- [1. Tổng quan cổng truy cập](#1-tổng-quan-cổng-truy-cập)
- [2. Server Management – Quản trị cấp Server (Admin/Service Provider)](#2-server-management--quản-trị-cấp-server-adminservice-provider)
  - [2.1 Tạo Service Plan (Gói hosting)](#21-tạo-service-plan-gói-hosting)
  - [2.2 Tạo tài khoản mới (Customer + Subscription)](#22-tạo-tài-khoản-mới-customer--subscription)
  - [2.3 Quản lý tài khoản hiện có](#23-quản-lý-tài-khoản-hiện-có)
  - [2.4 Permissions – Kiểm soát quyền/tính năng cho từng gói](#24-permissions--kiểm-soát-quyềntính-năng-cho-từng-gói)
  - [2.5 Reseller – Tạo và quản lý đại lý](#25-reseller--tạo-và-quản-lý-đại-lý)
  - [2.6 Quản lý dịch vụ hệ thống](#26-quản-lý-dịch-vụ-hệ-thống)
  - [2.7 Quản lý IP](#27-quản-lý-ip)
  - [2.8 DNS cấp server](#28-dns-cấp-server)
  - [2.9 SSL cấp server & Let's Encrypt](#29-ssl-cấp-server--lets-encrypt)
  - [2.10 Security – Firewall, Fail2Ban, ModSecurity](#210-security--firewall-fail2ban-modsecurity)
  - [2.11 PHP Manager – Cấu hình PHP toàn server](#211-php-manager--cấu-hình-php-toàn-server)
  - [2.12 Tools & Settings – Tinh chỉnh hành vi server](#212-tools--settings--tinh-chỉnh-hành-vi-server)
  - [2.13 Database Server Configuration](#213-database-server-configuration)
  - [2.14 Cấu hình Backup (Backup Manager cấp server)](#214-cấu-hình-backup-backup-manager-cấp-server)
  - [2.15 Restore Full Backup](#215-restore-full-backup)
- [3. Quản lý Tên miền & Web (Subscription)](#3-quản-lý-tên-miền--web-subscription)
- [4. Quản lý File](#4-quản-lý-file)
- [5. Quản lý Tài khoản FTP](#5-quản-lý-tài-khoản-ftp)
- [6. Quản lý Cơ sở dữ liệu](#6-quản-lý-cơ-sở-dữ-liệu)
- [7. Bảo mật – Kích hoạt SSL](#7-bảo-mật--kích-hoạt-ssl)
- [8. Quản lý Email & Chống Spam](#8-quản-lý-email--chống-spam)
- [9. Mailing List](#9-mailing-list)
- [10. Scheduled Tasks (Cron)](#10-scheduled-tasks-cron)
- [11. Backup & Restore (cấp Subscription)](#11-backup--restore-cấp-subscription)
- [12. Triển khai Next.js trên Plesk](#12-triển-khai-nextjs-trên-plesk)
- [13. 5 lỗi phổ biến nhất](#13-5-lỗi-phổ-biến-nhất)
- [14. Những lưu ý "chết người" – Dễ mất file/data](#14-những-lưu-ý-chết-người--dễ-mất-filedata)
- [15. Cấu hình nâng cao qua SSH (cho Admin)](#15-cấu-hình-nâng-cao-qua-ssh-cho-admin)
  - [15.1 Check log hệ thống & đăng nhập SSH](#151-check-log-hệ-thống--đăng-nhập-ssh)
  - [15.2 Check log tài khoản user cụ thể](#152-check-log-tài-khoản-user-cụ-thể)
  - [15.3 Check log Mail (Postfix/Qmail)](#153-check-log-mail-postfixqmail)
  - [15.4 Check log Web (Nginx/Apache)](#154-check-log-web-nginxapache)
  - [15.5 Check log Plesk Panel](#155-check-log-plesk-panel)
- [Bảng vị trí log quan trọng](#bảng-vị-trí-log-quan-trọng)
- [References](#references)

---

## 1. Tổng quan cổng truy cập

Khác với cPanel/WHM tách biệt theo cổng riêng cho từng cấp (2087/2083/2096), **Plesk dùng chung 1 cổng 8443** cho mọi cấp quản trị — sự khác biệt nằm ở **tài khoản đăng nhập**, không phải cổng:

| Cổng | Dịch vụ | Dành cho | Địa chỉ thực tế |
|------|---------|----------|----------------|
| **8443** | Plesk Panel | Admin — quản trị toàn server | `https://103.179.190.194:8443` (login `admin`) |
| **8443** | Plesk Panel | Reseller — quản lý khách hàng được cấp | `https://103.179.190.194:8443` (login reseller) |
| **8443** | Plesk Panel | Customer — quản lý 1 subscription | `https://103.179.190.194:8443` hoặc `https://<domain>:8443` (login customer) |
| — | Webmail | Người dùng cuối — đọc/gửi email | `https://webmail.<domain>` hoặc link **Webmail** trong mục Mail |

> Vì dùng chung cổng, Plesk phân quyền hoàn toàn dựa trên **loại tài khoản đăng nhập** (Administrator / Reseller / Customer) — đăng nhập bằng tài khoản nào, giao diện và phạm vi thao tác sẽ tự động giới hạn tương ứng.

---

## 2. Server Management – Quản trị cấp Server (Admin/Service Provider)

> Chỉ Admin và Reseller trong phạm vi được cấp mới truy cập được. Quản lý toàn bộ server: nhiều Subscription, dịch vụ hệ thống, firewall, cấu hình PHP/Web server, backup...
<img width="1886" height="937" alt="image" src="https://github.com/user-attachments/assets/54cda20c-bb0f-41d6-80bd-2b5df7cdc2a5" />

---

### 2.1 Tạo Service Plan (Gói hosting)

Service Plan định nghĩa giới hạn tài nguyên cho nhóm tài khoản — tương đương **Package** bên cPanel/WHM.

**Đường dẫn:** Plesk (Admin) → **Service Plans** → **Hosting Plan** → **Add a Plan**

**Bước 1:** Điền **Service Plan Name** — không dấu, không khoảng trắng, có ý nghĩa (ví dụ `hieu_doanhnghiep`).

**Bước 2:** Cấu hình tài nguyên tại tab **Resources**:

| Trường | Giá trị ví dụ | Ghi chú |
|--------|--------------|---------|
| Service Plan Name | `hieu_doanhnghiep  ` | Không dấu, không cách |
| Disk space | `6144 MB` | 6 GB — khớp thực tế trên server |
| Traffic | `Unlimited` | Hoặc giới hạn theo GB/tháng |
| Mailboxes | `Unlimited` | |
| Mailing lists | `10` | |
| MySQL/MariaDB databases | `6` | |
| Additional FTP accounts | `Unlimited` | |
| Subdomains | `Unlimited` | |
| Domain aliases | `Unlimited` | |
| Domains (addon) | `3` | |

<img width="1850" height="916" alt="image" src="https://github.com/user-attachments/assets/075d9ff9-b82d-43b7-b16b-a52fad86c877" />

**Bước 3:** Tab **Permissions** — bật/tắt các quyền tương ứng.

<img width="1857" height="925" alt="image" src="https://github.com/user-attachments/assets/d461a990-3124-4083-af2d-14cc40aa91e3" />

**Bước 4:** Nhấn **OK**/**Save** để lưu Service Plan.

> Không thể xóa Service Plan đang được gán cho Subscription. Muốn xóa phải **Change Plan** toàn bộ Subscription đang dùng sang plan khác trước.

---

### 2.2 Tạo tài khoản mới (Customer + Subscription)

**Đường dẫn:** Plesk (Admin) → **Hosting Services** → **Customers** → **Add Customer**

**Bước 1 – Thông tin liên hệ:**

| Trường | Giá trị ví dụ | Ghi chú |
|--------|--------------|---------|
| Contact name | `Nguyễn Thanh Hiếu` | Tên hiển thị |
| Email | `hieunt@nhanhoa.com.vn` | Email nhận thông báo hệ thống |
| Username | `iamhieu` | **TUYỆT ĐỐI không đặt là `admin`, `root`** |
| Password | *Z4wJ6f@!xKemmzv0* | Xem cảnh báo bên dưới |

> **Quy tắc đặt Username:**
> - Không dấu, không khoảng trắng.
> - Tránh trùng với các tên hệ thống: `admin`, `root`, `test`, `mail`, `ftp`.
> - Username gắn liền với tài khoản đăng nhập Plesk của khách hàng.

**Bước 2 – Mật khẩu mạnh:** Nhấn biểu tượng con mắt/nút **Generate** cạnh ô Password → hệ thống sinh mật khẩu ngẫu nhiên đủ mạnh → copy lưu lại ngay.

**Bước 3 – Tạo Subscription đi kèm** (ngay trong form tạo Customer, mục *Create a subscription for the customer*):

| Trường | Giá trị ví dụ | Ghi chú |
|--------|--------------|---------|
| Domain name | `hieucute.id.vn` | FQDN, không `http://` |
| Service plan | `hieu_doanhnghiep` | Chọn plan đã tạo |
| IP address | Chọn IP mặc định server | Có thể đổi sang Dedicated sau |

<img width="638" height="920" alt="image" src="https://github.com/user-attachments/assets/7f501e2f-6166-4743-bcc4-cfbcedbbd2c3" />

**Bước 4 – Activate account by email:** tick nếu muốn khách tự kích hoạt tài khoản qua email, hoặc để trống nếu Admin kích hoạt thủ công ngay.

**Bước 5:** Nhấn **OK** → theo dõi thông báo.

---

### 2.3 Quản lý tài khoản hiện có

#### Suspend / Unsuspend (Change Status)

**Đường dẫn:** Plesk (Admin) → **Hosting Services** → **Customers** → chọn customer → **Change Status** → **Suspend/Active**

<img width="1371" height="326" alt="image" src="https://github.com/user-attachments/assets/191755a7-93b8-46ed-82a8-cc0c94cbce96" />

> Khi Suspend, website và mail của khách hàng ngừng hoạt động ngay lập tức, khách hàng sẽ thấy trang thông báo tạm ngưng dịch vụ.

#### Đổi Service Plan / Nâng cấp tài khoản

**Đường dẫn:** Plesk (Admin) → **Hosting Services** → **Subscriptions** → tick subscription → **Change Plan** → chọn plan mới → **OK**

<img width="1143" height="368" alt="image" src="https://github.com/user-attachments/assets/29869daf-fdf6-4952-a2b2-2b0e6b286012" />

> Khi nâng Plan (tăng tài nguyên), thay đổi có hiệu lực **ngay lập tức** — không cần restart dịch vụ.

#### Xem thông tin tổng hợp tất cả tài khoản

**Đường dẫn:** Plesk (Admin) → **Hosting Services** → **Subscriptions** (hoặc **Customers**)

<img width="1654" height="405" alt="image" src="https://github.com/user-attachments/assets/b4018dd2-951c-44cd-8a0e-c74a34ef85ee" />

Bảng hiển thị: domain, tên khách hàng, Service Plan, ngày thiết lập, trạng thái (Active/Suspended). Hữu ích khi cần kiểm tra nhanh server đang có bao nhiêu subscription, subscription nào đang dùng nhiều tài nguyên nhất (xem thêm ở **Server Management → Info and Statistics**).

<img width="1607" height="504" alt="image" src="https://github.com/user-attachments/assets/b37f70c7-a45f-4e34-bcba-c3ca8d3c2b26" />

#### Remove (xóa vĩnh viễn)

**Đường dẫn:** Plesk (Admin) → **Hosting Services** → **Customers** → chọn customer → **Remove**

<img width="1643" height="390" alt="image" src="https://github.com/user-attachments/assets/cf2a80b0-1e74-4866-b8b4-0774e00ff6fd" />

> Xóa toàn bộ file, database, email, DNS của mọi Subscription thuộc customer này. Không có thùng rác, không undo. Bắt buộc tạo Backup đầy đủ trước.

---

### 2.4 Permissions – Kiểm soát quyền/tính năng cho từng gói

Permissions là khái niệm tương đương **Feature List** bên cPanel/WHM — bật/tắt quyền thao tác của khách hàng trong phạm vi Subscription (khác Feature List ở chỗ Plesk gắn Permissions **trực tiếp vào Service Plan**, không tạo file riêng biệt).

**Đường dẫn:** Plesk (Admin) → **Hosting Services** → **Service Plans** → chọn plan → tab **Permissions**

| Nhóm quyền | Bật/tắt điển hình |
|---|---|
| DNS Zone Management | Tuỳ nhu cầu — tắt nếu không muốn khách sửa DNS |
| Hosting Settings Management | Nên bật (SSL, PHP version...) |
| Spam Filter / Antivirus Management | Nên bật |
| Backup and restoration (server/remote storage) | Nên bật |
| Web Statistics Management | Luôn bật |
| WP Toolkit / Laravel Toolkit access | Tắt nếu gói cơ bản không hỗ trợ |
| Remote database access | Tắt với gói phổ thông |
| Additional FTP accounts | Luôn bật |

<img width="1652" height="836" alt="image" src="https://github.com/user-attachments/assets/b4d9c2db-47df-4f90-a8e2-f6a7298cc3e3" />

> Permissions ảnh hưởng đến **những gì khách hàng thấy và thao tác được trong giao diện Plesk của họ** — không ảnh hưởng tài nguyên thực tế.

---

### 2.5 Reseller – Tạo và quản lý đại lý

Reseller có thể tạo và quản lý Customer dưới quyền mình, với giới hạn tài nguyên do Admin cấp qua **Reseller Plan**.

#### Tạo Reseller Plan

**Đường dẫn:** Plesk (Admin) → **Hosting Services** → **Service Plans** → tab **Reseller Plans** → **Add a Plan**

Cấu hình tương tự Hosting Plan nhưng bổ sung:
- **Resources**: số lượng Customer tối đa được phép tạo.
  - **Permissions**: quyền quản trị Reseller (tạo customer, oversell tài nguyên...).
- **IP Addresses**: cấp IP dùng chung hoặc số lượng IP riêng để Reseller phân phối lại.

<img width="1106" height="849" alt="image" src="https://github.com/user-attachments/assets/4765aee3-dd0d-4b49-824f-e68e520ed75a" />

#### Tạo tài khoản Reseller

**Đường dẫn:** Plesk (Admin) → **Hosting Services** → **Resellers** → **Add Reseller** → gán Reseller Plan vừa tạo → **OK**

<img width="717" height="833" alt="image" src="https://github.com/user-attachments/assets/181d1cdc-e89a-4e42-9298-11b655db30a1" />

#### Xem tài khoản do Reseller quản lý

**Đường dẫn:** Plesk (Admin) → chọn Reseller trong danh sách **Resellers** → xem danh sách Customer trực thuộc.

> Reseller trong Plesk dùng chung giao diện Plesk chỉ thấy phạm vi trong quyền được cấp

---

### 2.6 Quản lý dịch vụ hệ thống

#### Restart dịch vụ

**Đường dẫn:** Plesk (Admin) → **Tools & Settings** → **Server Management** → **Services Management**

Danh sách dịch vụ có thể Stop/Restart:

| Dịch vụ | Khi nào cần restart |
|---------|---------------------|
| **Web Server (Apache/Nginx)** | Sau khi thay đổi cấu hình vhost, PHP handler |
| **MySQL/MariaDB** | Sau khi sửa cấu hình DB, khi service bị treo |
| **Mail Server** | Sau khi sửa cấu hình mail, khi mail queue bị kẹt |
| **FTP Server** | Sau khi thay đổi cấu hình FTP |
| **Plesk (sw-engine/plesk service)** | Khi giao diện Plesk không phản hồi |
| **DNS Server** | Sau khi sửa zone DNS quan trọng |

<img width="1867" height="935" alt="image" src="https://github.com/user-attachments/assets/6c92ef02-397f-48eb-a50e-340d176e1d75" />

> Restart Web Server làm **gián đoạn toàn bộ website** trên server trong vài giây. Nên thực hiện ngoài giờ cao điểm.

#### Kiểm tra trạng thái dịch vụ

Cùng giao diện **Services Management** hiển thị cột **State** (running/stopped) và **Startup type** cho từng dịch vụ.

#### Giám sát tài nguyên realtime

**Đường dẫn:** Plesk (Admin) → **Server Management** → **Monitoring** 

<img width="1630" height="583" alt="image" src="https://github.com/user-attachments/assets/f301221b-bb00-4853-b67a-2b297eff5e52" />

Tương đương lệnh `top` — hiển thị biểu đồ CPU/RAM/Disk/Network theo thời gian thực, theo từng dịch vụ (Web/Mail/MySQL/Plesk). Hữu ích khi server chạy chậm bất thường để xác định thành phần thủ phạm.

---

### 2.7 Quản lý IP

**Đường dẫn:** Plesk (Admin) → **Tools & Settings** → **Tools & Resources** → **IP Addresses**

<img width="1646" height="406" alt="image" src="https://github.com/user-attachments/assets/d18b1bcf-5441-426f-a139-d11431b931dc" />

#### Thêm IP mới vào server

**IP Addresses** → **Add IP Address** → nhập địa chỉ IP và subnet mask → **OK**.

<img width="864" height="428" alt="image" src="https://github.com/user-attachments/assets/e03b7943-d299-4229-adf8-ccbf14b7a8bb" />

> IP mới cần được nhà cung cấp route về server trước. Thêm trong Plesk chỉ là bước cấu hình phần mềm.

#### Gán IP riêng cho Subscription

Plesk (Admin) → **Hosting Services** → chọn Subscription → **Hosting Settings** → mục **IP addresses** → đổi từ Shared sang Dedicated IP → **OK**.

> Dedicated IP cần thiết khi: cài SSL không hỗ trợ SNI, chạy ứng dụng cần IP riêng, hoặc khách yêu cầu bảo mật cao hơn.

---

### 2.8 DNS cấp server

#### Sửa Zone DNS bất kỳ domain

**Đường dẫn:** Plesk (Admin) → **Hosting Services** → chọn Subscription → domain → **DNS Settings**

Admin có thể sửa zone của mọi domain trên server mà không cần đăng nhập vào tài khoản khách hàng.

<img width="1885" height="919" alt="image" src="https://github.com/user-attachments/assets/0e3e59c3-25a6-4512-a84e-6b92f0f693af" />

#### Cấu hình mẫu DNS mặc định cho toàn server

**Đường dẫn:** Plesk (Admin) → **Tools & Settings** → **DNS Settings**

<img width="1863" height="928" alt="image" src="https://github.com/user-attachments/assets/c2467697-c70e-4451-8e9c-dc7ec99272b7" />

Cấu hình **Zone Records Template**, **Zone Settings Template**, **Zone Transfers** áp dụng làm mặc định cho mọi domain mới được tạo trên server.

---

### 2.9 SSL cấp server & Let's Encrypt

#### Cấu hình chứng chỉ bảo vệ chính panel Plesk

**Đường dẫn:** Plesk (Admin) → **Tools & Settings** → **SSL/TLS Certificates**

<img width="1620" height="837" alt="image" src="https://github.com/user-attachments/assets/d79c3235-9017-4061-9da1-35160eb9515e" />

- **Let's Encrypt** — miễn phí, tự động, thời hạn 90 ngày, tự gia hạn.
- **Upload certificate** — dùng chứng chỉ thương mại đã mua.

Bước 1: Nhấn vào nút + Let's Encrypt màu xám nằm ở ngay phía dưới tiêu đề "Certificates currently in use for securing Plesk server".  

Bước 2: Hệ thống sẽ hiện ra một form yêu cầu điền Domain/Hostname của server (Ví dụ: server.hieucute.id.vn).  
<img width="711" height="351" alt="image" src="https://github.com/user-attachments/assets/52700ea2-a631-44d7-9626-2d1f4ca028c4" />

Bước 3: Nhấn Install.  

Bước 4: Sau khi cài xong, quay lại:  

Tại dòng Certificate for securing mail, bấm vào nút [Change].  

Chọn chứng chỉ Let's Encrypt vừa tạo từ danh sách thả xuống để bảo mật hoàn toàn cho cổng gửi/nhận Mail.  
<img width="639" height="261" alt="image" src="https://github.com/user-attachments/assets/f34f9190-1fca-4611-9a54-1280bda6aa9c" />



#### Cài SSL cho từng domain hàng loạt

Chọn domain trong danh sách **Subscriptions** và Manage in Customer Panel

<img width="1850" height="890" alt="image" src="https://github.com/user-attachments/assets/c9bd96c5-1929-4d33-be18-10c5f9a2aa23" />

<img width="887" height="863" alt="image" src="https://github.com/user-attachments/assets/c9c82247-e3f4-42ab-b70d-fde890e8e759" />

#### Kiểm tra tình trạng SSL toàn server

**Đường dẫn:** Plesk (Admin) → **Home** — mục **SSL/TLS Certificates**, hiển thị danh sách site có/thiếu chứng chỉ SSL hợp lệ.

<img width="1869" height="858" alt="image" src="https://github.com/user-attachments/assets/3f53299f-7aab-41f9-ad56-191012a60f7b" />

#### Cài SSL có trả phí 
Trong trang “SSL/TLS Certificates“, nhấp vào nút “Manage” rồi sau đó nhấp vào nút “Add SSL/TLS Certificate” để bắt đầu quá trình cài đặt SSL.  
<img width="1870" height="913" alt="image" src="https://github.com/user-attachments/assets/278c6256-bde5-4eb2-8902-e0ae9c3b5189" />
<img width="1871" height="944" alt="image" src="https://github.com/user-attachments/assets/fc906661-d1ef-4639-8d42-16576927d8ca" />

Nhập các thông tin cần thiết 
<img width="1787" height="922" alt="image" src="https://github.com/user-attachments/assets/220a65ac-d00a-45f2-9bb5-72668ab2db60" />

Upload các file key hoặc dán dạng text ở bên dưới, sau đó ấn upload
<img width="1197" height="501" alt="image" src="https://github.com/user-attachments/assets/bcdaeb8f-0551-4fd9-bea9-9db5488e9a14" />

---

### 2.10 Security – Firewall, Fail2Ban, ModSecurity

**Đường dẫn:** Plesk (Admin) → **Tools & Settings** → nhóm **Security**
<img width="351" height="328" alt="image" src="https://github.com/user-attachments/assets/5f2b908b-d7c4-43cb-aef1-dacf02bbaa9e" />

#### Firewall

**Firewall** → bật **Firewall Protection**, cấu hình **Firewall Rules** theo port/protocol/IP, kích hoạt **Panic Mode** khi cần chặn khẩn cấp toàn bộ lưu lượng.
<img width="1173" height="835" alt="image" src="https://github.com/user-attachments/assets/cb25651a-2124-4eff-9811-cb8fd0dd8742" />

#### Fail2Ban (chống brute-force)

**Đường dẫn:** Plesk (Admin) → **Tools & Settings** → **IP Address Banning (Fail2Ban)**

<img width="949" height="479" alt="image" src="https://github.com/user-attachments/assets/6173dea3-4a17-438a-b6a1-fe61bc6c8ad7" />

Tự động chặn IP đăng nhập sai nhiều lần vào Plesk/SSH/FTP/Mail. Cấu hình:
- Số lần thất bại tối đa trước khi chặn: `5`
- Thời gian theo dõi: `10 phút`
- Thời gian chặn IP: `10 phút` – `1 giờ` (tuỳ mức độ)

```bash
# Xem danh sách IP đang bị Fail2Ban chặn (SSH)
fail2ban-client status ssh

# Gỡ chặn 1 IP cụ thể
fail2ban-client set sshd unbanip 1.2.3.4
```
<img width="604" height="130" alt="image" src="https://github.com/user-attachments/assets/3b6a13b4-7478-454f-9836-463b93fb698e" />
Trong đây có list IP đã bị chặn, list IP được cho phép, logs và jails là các bộ lọc tự động theo dõi, nếu ai đó gõ sai mật khẩu SSH, Mail, Apache hay Plesk Panel quá số lần quy định, họ sẽ bị tống vào các "nhà tù" tương ứng này  

#### Web Application Firewall (ModSecurity)

**Đường dẫn:** Plesk (Admin) → **Tools & Settings** → **Web Application Firewall (ModSecurity)**
<img width="1605" height="824" alt="image" src="https://github.com/user-attachments/assets/698014d9-a92a-4c29-9540-1ab65ae9b3ad" />

- Off: Server không kiểm tra gì cả. Hacker gửi code độc, SQL Injection hay lệnh phá hoại lên, website nhận hết. Website chạy rất nhẹ nhưng cực kỳ nguy hiểm.   

- Detection only (Chỉ phát hiện/Ghi log): Khi có một cuộc tấn công, ModSecurity vẫn cho phép kẻ tấn công truy cập vào website, nhưng nó sẽ âm thầm ghi lại toàn bộ hành vi, IP, loại tấn công đó vào file log. Chế độ này cực kỳ hữu ích khi bạn mới cài một website lạ, bật lên để xem website đó có bị xung đột với tường lửa hay không mà không làm gián đoạn người dùng.    

- On (Bật hoàn toàn): Khi phát hiện request có dấu hiệu độc hại, ModSecurity sẽ chặn đứng lập tức và trả về lỗi 403 Forbidden cho người truy cập, đồng thời ghi log lại. Website của bạn sẽ cực kỳ an toàn.  
 

<img width="950" height="754" alt="image" src="https://github.com/user-attachments/assets/b752f273-2686-431f-be48-8c4a4694d082" />
- OWASP (Miễn phí): Bộ luật tiêu chuẩn quốc tế, cực kỳ nghiêm ngặt. Nó chặn hack rất tốt nhưng vì quá chặt nên rất dễ chặn nhầm cả người dùng thật.   

- Comodo Free: Bộ luật rất tốt dành cho các website chạy mã nguồn phổ thông như WordPress, Joomla... Nó được tối ưu hóa để cân bằng giữa bảo mật và hạn chế việc chặn nhầm người dùng.   

- tomic (Trả phí): Bộ luật cao cấp, cập nhật liên tục hàng ngày để chống lại các lỗ hổng bảo mật mới xuất hiện.   

Nếu khách hàng lỗi do modsecurity chặn, ta check ID ở log chứa tên miền của khách hàng, sau đó quay lại trang ModSecurity audit log dán ID vào ô Switch off security rules  

<img width="1266" height="649" alt="image" src="https://github.com/user-attachments/assets/f21fff93-3c7e-4175-843a-7827ef94b490" />

---

### 2.11 PHP Manager – Cấu hình PHP toàn server
Quản lý các PHP Handler cài sẵn cho toàn server.

**Đường dẫn:** Plesk (Admin) → **Tools & Settings** → **General Settings** → **PHP Settings**

<img width="1655" height="644" alt="image" src="https://github.com/user-attachments/assets/4c50592e-63fa-435f-8983-30c0e125c36a" />

#### Cài thêm PHP version

Trên Linux, cài thêm phiên bản PHP qua package manager của hệ điều hành hoặc Plesk Installer, sau đó handler mới sẽ tự xuất hiện trong danh sách **PHP Settings**.

#### Đổi phiên bản PHP cho từng domain

**Đường dẫn:** Plesk → chọn domain → **PHP Settings** → chọn version + Handler type (FPM/CGI/FastCGI) → **OK**.

<img width="1878" height="909" alt="image" src="https://github.com/user-attachments/assets/911bcc9a-2d7d-4a3e-b3a0-4a3f36b598ae" />

> Đổi PHP version áp dụng gần như ngay lập tức cho domain đó, không cần restart toàn bộ Web Server như khi build lại Apache trong EasyApache.

---

### 2.12 Tools & Settings – Tinh chỉnh hành vi server

**Đường dẫn:** Plesk (Admin) → **Tools & Settings** → **General Settings**

Tập trung hàng loạt cài đặt chi tiết cho toàn server. Các setting quan trọng nhất:

| Nhóm | Setting | Khuyến nghị |
|-----|---------|------------|
| **Mail** | Limit outgoing messages (per domain/hour) | `50–200` (chống spam) |
| **Mail** | Spam Filter (SpamAssassin) | Bật |
| **System** | PHP `upload_max_filesize`/`post_max_size` | `128M` |
| **System** | Website Preview | Bật quick preview nội bộ |
| **Security** | Secure FTP (FTPS) | Chỉ cho phép kết nối bảo mật |
| **Security** | Password Strength | Mạnh trở lên |
| **DNS** | Recursive query (Service-wide) | Deny for all requests |

> Thay đổi tại đây ảnh hưởng **toàn bộ server**. Đọc kỹ mô tả của từng setting trước khi thay đổi.

---

### 2.13 Database Server Configuration

**Đường dẫn:** Plesk (Admin) → **Tools & Settings** → **Applications & Databases** (Database servers)

<img width="1644" height="347" alt="image" src="https://github.com/user-attachments/assets/400b9738-5fc0-4475-a078-91cf3d0e1724" />

#### Quản lý Database Server

Tại giao diện này, ấn + Add Database Server, điền IP của VPS cơ sở dữ liệu từ xa và tài khoản root.
<img width="656" height="641" alt="image" src="https://github.com/user-attachments/assets/b06b46e8-3ea4-43b8-86a3-03a284810b78" />

Plesk sẽ kết nối và quản lý. Khi khách hàng tạo database mới, họ có quyền chọn lưu data ở server phụ đó để giảm tải hoàn toàn cho server web chính.  

Ta có thể login vào database của khách hàng để sửa bất cứ lúc nào

<img width="656" height="641" alt="image" src="https://github.com/user-attachments/assets/409bbb52-e2bd-4aab-9f3c-18fcc177d8c7" />


#### Tinh chỉnh cấu hình DB qua SSH 

Plesk không cung cấp giao diện chỉnh `my.cnf` chi tiết nên thao tác này thực hiện trực tiếp qua SSH:

```bash
nano /etc/mysql/mariadb.conf.d/50-server.cnf
# Chỉnh innodb_buffer_pool_size, max_connections...
systemctl restart mariadb
```
<img width="1274" height="760" alt="image" src="https://github.com/user-attachments/assets/e401ac12-e100-41e3-b0c0-8975933d4ae4" />

> Sửa cấu hình DB sẽ **restart MySQL/MariaDB** — toàn bộ kết nối database bị ngắt trong vài giây, mọi website WordPress/Laravel trên server báo lỗi kết nối DB tạm thời.

---

### 2.14 Cấu hình Backup (Backup Manager cấp server)

**Đường dẫn:** Plesk (Admin) → **Tools & Settings** → **Tools & Resources** → **Backup Manager** (cấp server), hoặc từng Subscription có Backup Manager riêng.
<img width="1380" height="466" alt="image" src="https://github.com/user-attachments/assets/d64d4eae-712d-40f8-aced-e36eb8d34ef7" />

#### Loại Backup

| Loại | Mô tả | Khi nào dùng |
|------|-------|-------------|
| **Full backup** | Toàn bộ config, mail, file, database | Backup định kỳ toàn diện |
| **Incremental** | Chỉ sao lưu phần thay đổi | Server dữ liệu lớn, backup hàng ngày |

<img width="783" height="796" alt="image" src="https://github.com/user-attachments/assets/7738e862-16ba-4466-a285-563df1654c82" />

#### Lịch chạy (Schedule)

Cấu hình: kích hoạt task, thời gian thực thi, loại backup, số lượng bản full backup được giữ lại (retain).

<img width="950" height="825" alt="image" src="https://github.com/user-attachments/assets/a7f6a07e-4dbd-4132-b201-2076d7229e94" />

#### Lựa chọn dữ liệu sao lưu

| Hạng mục | Quan trọng? |
|----------|------------|
| **Cấu hình (Configuration)** | Cốt lõi |
| **Mail** | Nên bật |
| **Tệp người dùng (Files)** | Cốt lõi |
| **Cơ sở dữ liệu** | Cốt lõi |

#### Lưu trữ ngoại vi (Remote Storage – FTPS)

> **TUYỆT ĐỐI không lưu backup trên cùng ổ đĩa với dữ liệu gốc.** Áp dụng chiến lược **3-2-1**: 3 bản sao, 2 loại phương tiện, 1 bản off-site.

Cấu hình: **Add Remote Storage Server** → nhập Host/Username/Password/Port FTPS → **Save and Validate**.
  <img width="635" height="500" alt="image" src="https://github.com/user-attachments/assets/4154acee-33bc-48a1-8cca-b68d1a4d2555" />

---

### 2.15 Restore Full Backup

File backup Plesk có định dạng `.tar` (không nén như `cpmove` của cPanel nhưng cấu trúc tương tự — gồm cấu hình, dữ liệu file, dump database, metadata subscription).

#### Cách 1: Restore qua giao diện Plesk

**Đường dẫn:** Plesk → **Backup Manager** (server hoặc subscription) → **Upload Files** hoặc chọn bản backup có sẵn → bấm vào thẳng link backup đấy → **Restore** → chọn nội dung cần khôi phục (Configuration/Mail/Files/Database) → **OK**.

<img width="1445" height="834" alt="image" src="https://github.com/user-attachments/assets/e13bd0dd-0437-43bd-9a33-1b7c04503e48" />

#### Cách 2: Restore qua SSH (file lớn)

```bash
# Kiểm tra file backup tồn tại
ls -lh /var/lib/psa/dumps/

# Restore bằng công cụ pleskbackup / pleskrestore
/usr/local/psa/bin/pleskrestore --restore /path/to/backup.tar -level subscription -subscription-name hieucute.id.vn
```
<img width="982" height="634" alt="image" src="https://github.com/user-attachments/assets/65b9a04e-4475-4a2a-a957-f6bd0796cc2d" />

> Restore ghi đè dữ liệu hiện tại của Subscription đích. Luôn kiểm tra bản backup hiện tại **trước khi** restore một bản cũ hơn.

---


## 3. Quản lý Tên miền & Web (Subscription)

### 3.1 Trỏ tên miền về VPS

Trước khi thao tác, DNS của domain phải trỏ đúng về IP server Plesk.

| Host | Type | Value | TTL |
|------|------|-------|-----|
| `@` | A | `103.179.190.194` | 300 |
| `www` | CNAME | `hieucute.id.vn.` | 300 |
| `mail` | A | `103.179.190.194` | 300 |

```bash
dig A hieucute.id.vn +short
# Kết quả đúng: 103.179.190.194
```

<img width="982" height="55" alt="image" src="https://github.com/user-attachments/assets/d103b5dc-989d-4f0a-96f4-d6a6f16324f5" />


### 3.2 Thêm Domain mới

**Đường dẫn:** Plesk → **Websites & Domains** → **Add Domain**

| Trường | Giá trị | Ghi chú |
|--------|---------|---------|
| Domain name | `shop.hieucute.id.vn` | FQDN, không `www` hay `http://` |
| Document root | `/shop.hieucute.id.vn` (mặc định) | Có thể sửa |
| Hosting type | Website | Web Hosting/Forwarding/No Hosting |

Sau khi tạo → thêm bản ghi A tại DNS trỏ đúng subdomain nếu cần.

### 3.3 Cài đặt WordPress qua WP Toolkit

**Đường dẫn:** Plesk → sidebar → **WordPress** → **Install**

| Trường | Giá trị | Quy tắc |
|--------|---------|---------|
| Installation path | domain gốc hoặc thư mục con | - |
| WordPress version | Latest stable | Luôn phiên bản mới nhất |
| Admin username | `admin_abc` | **Không dùng `admin`** |
| Admin password | *(nút Generate)* | Tối thiểu 12 ký tự |
| Admin email | `hieunt@nhanhoa.com.vn` | Email thật |

Nhấn **Install** → chờ 1–2 phút → truy cập `https://hieucute.id.vn/wp-admin`.

---

## 4. Quản lý File

### 4.1 Upload file lên Document Root

**Cách 1: File Manager** — Plesk → **Files** (trong Subscription) → điều hướng thư mục → **Upload Files**.

**Cách 2: SFTP qua FileZilla (khuyến nghị)**
```
Host: 103.179.190.194  |  Port: 22  |  Username: (system user của domain)  |  Password: mật khẩu FTP/hệ thống
```

### 4.2 Giải nén file

Chọn file `.zip`/`.tar.gz` trong File Manager → **Extract Files** → xác nhận thư mục đích.

> Sau giải nén hay bị lồng thư mục (`httpdocs/httpdocs/`). Khắc phục: Select All → **Move** về đúng thư mục Document Root.

### 4.3 Phân quyền file (Chmod)

| Đối tượng | Số | Lý do |
|-----------|-----|-------|
| Thư mục Document Root | `755` | Web server cần execute |
| File `.php`, `.html` | `644` | Đọc được, không execute |
| File `wp-config.php` | `600` | Chỉ owner đọc |
| Thư mục `wp-content/uploads` | `775` | WordPress cần ghi |

```bash
find /var/www/vhosts/hieucute.id.vn/httpdocs -type d -exec chmod 755 {} \;
find /var/www/vhosts/hieucute.id.vn/httpdocs -type f -exec chmod 644 {} \;
chmod 600 /var/www/vhosts/hieucute.id.vn/httpdocs/wp-config.php
```

---

## 5. Quản lý Tài khoản FTP

**Đường dẫn:** Plesk → **Websites & Domains** → chọn domain → **FTP Access** (hoặc **Hosting & DNS → FTP**)

| Trường | Giá trị | Ghi chú |
|--------|---------|---------|
| FTP account name | `ftpdev` | |
| Home directory | thư mục con cần giới hạn | Giới hạn phạm vi truy cập |
| Password | *(nút Generate)* | |
| Hard disk quota | `500 MB` hoặc Unlimited | |

Kết nối FileZilla:
```
Host: 103.179.190.194  |  Port: 22 (SFTP)  |  Username: ftpdev
```

> Ưu tiên SFTP (port 22). Kiểm tra chính sách **Secure FTP (FTPS)** tại mục Security (2.10) nếu bắt buộc kết nối mã hoá.

---

## 6. Quản lý Cơ sở dữ liệu

**Đường dẫn:** Plesk → **Databases** (trong Subscription) → **Add Database**

**Bước 1:** Nhập tên database `wpdb`.

**Bước 2:** Chọn Database server (MySQL/MariaDB/PostgreSQL).

**Bước 3:** Tại tab **Users**, nhấn **Add Database User** → tạo `wpuser` + mật khẩu (nút Generate).

**Bước 4:** Lưu thông tin kết nối:
```
DB_NAME:     wpdb
DB_USER:     wpuser
DB_PASSWORD: (mật khẩu vừa tạo)
DB_HOST:     localhost
```

**Kết nối phpMyAdmin:** nhấn nút **phpMyAdmin** ngay cạnh database trong danh sách để quản trị trực quan (Export/Import Dump, Check & Repair...).

---

## 7. Bảo mật – Kích hoạt SSL

### 7.1 Let's Encrypt

**Đường dẫn:** Plesk → **Websites & Domains** → chọn domain → **SSL/TLS Certificates** → **Install a free basic certificate provided by Let's Encrypt** → chọn phạm vi bảo vệ (domain chính, www, webmail, mail) → **Get it free**.

Chờ 1–3 phút → trạng thái Secured.

### 7.2 Cài SSL có sẵn 

Plesk → **SSL/TLS Certificates** → **Upload the certificate here** → chọn file `.pem`/private key/CA bundle → **Upload**.

### 7.3 Gỡ SSL cũ khi Let's Encrypt thất bại

1. **SSL/TLS Certificates** → **Manage** → chọn cert lỗi → **Remove**.
2. **File Manager** → kiểm tra/xoá thư mục `.well-known` tồn đọng trong Document Root nếu có.
3. Thử lại **Get it free**, chờ 5–10 phút.

### 7.4 Redirect HTTP to HTTPS

Plesk → **Hosting Settings** của domain → tick **Redirect visitors from HTTP to HTTPS**.

> **Thứ tự bắt buộc:** Cài SSL xong, xác nhận cert hợp lệ **trước** khi bật Redirect. Bật trước khi cert active sẽ gây redirect loop — website sập hoàn toàn.

---

## 8. Quản lý Email & Chống Spam

### 8.1 Tạo tài khoản Email

**Đường dẫn:** Plesk → **Mail** → **Create Email Address**

| Trường | Giá trị | Ghi chú |
|--------|---------|---------|
| Email address | `a` | Địa chỉ: `a@hieucute.id.vn` |
| Password | *Pp~sv2127* | |
| Mailbox size | `1024 MB` hoặc Unlimited | |

<img width="1645" height="850" alt="image" src="https://github.com/user-attachments/assets/e86ce859-41d6-4432-a0b4-3012936f11d0" />

Webmail: truy cập link **Webmail** ngay trong danh sách tài khoản mail — đăng nhập bằng `a@hieucute.id.vn`.

<img width="1886" height="932" alt="image" src="https://github.com/user-attachments/assets/9aba12ea-33cc-465b-af93-6f9fd751bfe8" />

### 8.2 Giới hạn email gửi đi (Limit Outgoing Messages)

Plesk → **Mail** → **Mail Settings** 
<img width="606" height="199" alt="image" src="https://github.com/user-attachments/assets/bed93404-d689-4866-9e5d-44b15e142aab" />

### 8.3 SPF, DKIM, DMARC

**Đường dẫn:** Plesk → **Mail** → **Mail Settings** → bật **Use DKIM**; cấu hình SPF/DMARC qua **DNS Settings** (thêm bản ghi TXT thủ công).

| Record | Bản ghi mẫu |
|--------|-------------|
| SPF | `v=spf1 ip4:103.179.190.194 ~all` |
| DKIM | Plesk tự sinh khi bật Use DKIM — copy public key vào DNS nếu quản lý DNS ngoài |
| DMARC (giai đoạn 1) | `v=DMARC1; p=none; rua=mailto:hieunt@nhanhoa.com.vn` |
| DMARC (giai đoạn 3 – nghiêm ngặt) | `v=DMARC1; p=reject; pct=100; rua=mailto:hieunt@nhanhoa.com.vn` |

```bash
dig TXT hieucute.id.vn | grep spf
dig TXT default._domainkey.hieucute.id.vn
dig TXT _dmarc.hieucute.id.vn
```

---

## 9. Mailing List

**Đường dẫn:** Plesk → **Mail** → **Mailing Lists** → **Create Mailing List**

Nhập địa chỉ mailing list, email admin quản lý, danh sách subscribers → **OK**.

---

## 10. Scheduled Tasks (Cron)

**Đường dẫn:** Plesk → **Websites & Domains** → chọn domain → **Scheduled Tasks**

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
0 2 * * * mysqldump -u wpuser -p'password' wpdb | gzip > /var/www/vhosts/hieucute.id.vn/backups/db_$(date +\%Y-\%m-\%d).sql.gz

# Xóa backup cũ hơn 7 ngày
0 3 * * * find /var/www/vhosts/hieucute.id.vn/backups -type f -mtime +7 -delete
```

> Luôn dùng đường dẫn **tuyệt đối** trong lệnh cron. Cấu hình email nhận thông báo lỗi ngay trong giao diện Scheduled Tasks.

---

## 11. Backup & Restore (cấp Subscription)

### 11.1 Full Backup

**Đường dẫn:** Plesk → Subscription → **Backup Manager** → **Back Up** → chọn nội dung (Configuration/Mail/Files/Database) → chọn nơi lưu → **OK**.

Tải về: **Backup Manager** → chọn bản backup → **Download**.

### 11.2 Backup từng phần

Tương tự mục 11.1 nhưng bỏ tick các hạng mục không cần (ví dụ chỉ backup Database, hoặc chỉ Files).

### 11.3 Restore

Plesk → **Backup Manager** → chọn bản backup → **Restore** → chọn lại đúng hạng mục cần khôi phục (Configuration/Mail/Files/Database) → **OK**.

> Restore ghi đè dữ liệu hiện tại của mục được chọn. Full Backup cấp server chỉ restore được qua Admin (xem mục 2.15).

---

## 12. Triển khai Next.js trên Plesk

**Đường dẫn:** Plesk → **Websites & Domains** → chọn domain → **Node.js**

| Trường | Giá trị |
|--------|---------|
| Node.js version | `18.x` hoặc `20.x` |
| Application mode | `Production` |
| Document root | thư mục chứa mã nguồn Next.js |
| Application startup file | `server.js` |

Cài qua **Run Nodejs Command** (terminal tích hợp trong Plesk) hoặc SSH:
```bash
cd /var/www/vhosts/hieucute.id.vn/httpdocs && rm -rf *
npx create-next-app@latest . --yes
npm run build
```

File `server.js`:
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

Sau khi tạo xong: nhấn **Enable Node.js** → **Restart App**.

---

## 13. 5 lỗi phổ biến nhất

### Lỗi 1 – 500 Internal Server Error

**Log:** Plesk → **Websites & Domains** → **Logs** | SSH: `/var/www/vhosts/system/<domain>/logs/error_log`

```bash
mv /var/www/vhosts/hieucute.id.vn/httpdocs/.htaccess /var/www/vhosts/hieucute.id.vn/httpdocs/.htaccess_bak
find /var/www/vhosts/hieucute.id.vn/httpdocs -type f -name "*.php" -exec chmod 644 {} \;
find /var/www/vhosts/hieucute.id.vn/httpdocs -type d -exec chmod 755 {} \;
```

Đổi PHP version: Plesk → domain → **PHP Settings**.

### Lỗi 2 – Email vào Spam / bị từ chối

**Log:** Plesk → **Mail** → **Mail Settings** hoặc theo dõi `/var/log/maillog`

1. Kiểm tra lại DKIM/SPF/DMARC (mục 8.3).
2. Kiểm tra blacklist: [mxtoolbox.com/blacklists.aspx](https://mxtoolbox.com/blacklists.aspx)
3. Liên hệ nhà cung cấp tạo PTR record: IP server → `mail.hieucute.id.vn`.

### Lỗi 3 – Let's Encrypt thất bại

```bash
dig A hieucute.id.vn +short          # Phải trả về đúng IP server
curl -I http://hieucute.id.vn         # Port 80 phải phản hồi
dig CAA hieucute.id.vn                 # Kiểm tra bản ghi CAA (nếu có, phải cho phép Let's Encrypt)
```

### Lỗi 4 – WordPress "Error establishing a database connection"

File Manager → `wp-config.php` → **Edit**:
```php
define( 'DB_NAME',     'wpdb' );
define( 'DB_USER',     'wpuser' );
define( 'DB_PASSWORD', 'correct_password' );
define( 'DB_HOST',     'localhost' );
```

### Lỗi 5 – Upload file 413 / Timeout

Plesk → domain → **PHP Settings** → sửa:

| Directive | Giá trị |
|-----------|---------|
| `upload_max_filesize` | `128M` |
| `post_max_size` | `128M` |
| `max_execution_time` | `300` |
| `memory_limit` | `256M` |

---

## 14. Những lưu ý "chết người" – Dễ mất file/data

> Tổng hợp các thao tác **không thể hoàn tác** hoặc **gây mất dữ liệu không cảnh báo rõ ràng** trong Plesk. Đọc kỹ trước khi thực hiện.

### 14.1 Remove Customer trong Admin — Xóa vĩnh viễn, không hỏi lại nhiều

**Plesk (Admin) → Hosting Services → Customers → Remove**

Xóa toàn bộ file website, database, email, DNS zone của mọi Subscription thuộc customer. **Không có thùng rác, không undo.**

> **Bắt buộc:** Tạo Full Backup (mục 2.14) và xác nhận file backup tồn tại thực tế **trước khi** Remove.

### 14.2 Restore ghi đè Subscription hiện tại

**Backup Manager → Restore → chọn ghi đè dữ liệu hiện có**

Nếu bản backup đã cũ, dữ liệu mới hơn (bài viết, đơn hàng) trên site hiện tại sẽ mất khi restore đè lên.

> Luôn backup bản hiện tại **trước** khi restore bản cũ hơn.

### 14.3 phpMyAdmin — Drop Database/Table không hỏi lại kỹ

Trong phpMyAdmin (mở từ Plesk Databases), tab **Operations** → **Drop the database** — thực hiện gần như ngay lập tức.

> Luôn **Export** ra file `.sql` trước khi thay đổi cấu trúc database.

### 14.4 File Manager — Xóa file bỏ qua Recycle Bin

Plesk File Manager có thể cấu hình xóa vĩnh viễn ngay (tuỳ theo cấu hình Recycle Bin của server).

> Nén (`.zip`) và tải file quan trọng về máy trước khi xóa.

### 14.5 Giải nén file .zip đè lên thư mục hiện có

Giải nén đè trực tiếp vào Document Root đang có website — Plesk ghi đè file trùng tên (`index.php`, `wp-config.php`, `.htaccess`) mà không luôn cảnh báo rõ ràng.

> Backup thư mục hiện tại trước khi giải nén bản mới, hoặc giải nén vào thư mục tạm rồi mới move vào production.

### 14.6 Redirect HTTP → HTTPS bật trước khi có SSL hợp lệ

Bật **Redirect visitors from HTTP to HTTPS** khi cert chưa active hoặc đã hết hạn → website rơi vào **infinite redirect loop**.

> Thứ tự bắt buộc: (1) Cài SSL + xác nhận hoạt động, (2) mới bật Redirect.

### 14.7 Đổi PHP Handler toàn server — Restart Web Server

Đổi PHP handler/version áp dụng cho nhiều domain cùng lúc có thể **yêu cầu restart Web Server**, gây gián đoạn tạm thời toàn bộ site trên máy chủ.

> Thực hiện vào khung giờ thấp điểm (1:00–4:00 sáng).

### 14.8 Sai cấu hình Mail Settings → Mất toàn bộ email đến

Cấu hình sai chính sách nhận email (ví dụ chọn **Reject** cho địa chỉ không tồn tại trong khi DNS domain đang trỏ sai) → email bị từ chối âm thầm, không bounce.

> Sau khi đổi bất kỳ Mail Setting nào: test gửi/nhận ngay với Gmail/Outlook, kiểm tra `/var/log/maillog`.

### 14.9 Xóa SSL Private Key trước khi có cert mới

**SSL/TLS Certificates → Manage → Remove** cert cũ trong khi vẫn đang được website sử dụng → lỗi SSL ngay lập tức.

> Thứ tự đúng: cài cert mới và test hoạt động → mới xoá cert/key cũ.

### 14.10 Reset mật khẩu Customer không lưu lại

Plesk (Admin) → **Customers** → đổi thông tin đăng nhập → hiệu lực ngay lập tức. Không lưu/gửi lại cho khách → mất quyền truy cập.

> Luôn copy mật khẩu mới vào password manager **trước** khi rời trang, gửi ngay cho khách hàng.

### 14.11 Remove Subscription khi Backup đang chạy dở

Xóa Subscription trong lúc backup job đang chạy → file backup dở dang bị lỗi, không restore được.

> Kiểm tra **Backup Manager** đảm bảo không có job đang chạy trước khi Remove/Terminate.

### 14.12 Sửa DNS Zone xóa nhầm bản ghi MX

**DNS Settings → Records → xóa nhầm MX** → toàn bộ email đến của domain bị mất, không bounce ngay (do TTL DNS còn hiệu lực).

> Ghi lại/export cấu hình Zone trước khi sửa DNS. Sau khi sửa, kiểm tra ngay bằng `dig MX hieucute.id.vn`.

---

## 15. Cấu hình nâng cao qua SSH (cho Admin)

> Các lệnh dưới đây cần chạy với quyền `root` trên server.

### 15.1 Check log hệ thống & đăng nhập SSH

**File log:** `/var/log/auth.log` (Ubuntu) | `/var/log/secure` (CentOS/AlmaLinux)

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

> Nếu thấy 1 IP lặp lại hàng trăm lần trong Failed password: `fail2ban-client set sshd banip <IP>` hoặc chặn thủ công qua Firewall (mục 2.10).

### 15.2 Check log tài khoản user cụ thể

```bash
# Tiến trình đang chạy của system user gắn với domain hieucute.id.vn
ps -u abc_vn -f

# Giám sát realtime CPU/RAM
top -u abc_vn

# Lịch sử lệnh đã gõ (nếu user có shell access)
cat /var/www/vhosts/hieucute.id.vn/.bash_history
```

### 15.3 Check log Mail (Postfix/Qmail)

**File log chính:** `/var/log/maillog` (CentOS/AlmaLinux) hoặc `/var/log/mail.log` (Ubuntu)

```bash
# Theo dõi mail log realtime
tail -f /var/log/maillog

# Lọc mail liên quan đến địa chỉ cụ thể
grep "hieunt@nhanhoa.com.vn" /var/log/maillog | tail -50

# Xem mail đang kẹt trong queue (Postfix)
mailq

# Xóa toàn bộ mail trong queue (cẩn thận!)
postsuper -d ALL
```

### 15.4 Check log Web (Nginx/Apache)

**File log:** `/var/www/vhosts/system/<domain>/logs/`

```bash
# Theo dõi log lỗi domain cụ thể realtime
tail -f /var/www/vhosts/system/hieucute.id.vn/logs/error_log

# Xem 100 dòng access log gần nhất
tail -100 /var/www/vhosts/system/hieucute.id.vn/logs/access_log

# Lọc các lỗi HTTP 500/502/503
grep -i "500\|502\|503" /var/www/vhosts/system/hieucute.id.vn/logs/access_log | tail -50

# Log Nginx cấp server (reverse proxy phía trước Apache)
tail -f /var/log/nginx/error.log
```

### 15.5 Check log Plesk Panel

**File log:** `/var/log/plesk/`

```bash
# Theo dõi hoạt động chính panel Plesk (thao tác quản trị)
tail -f /var/log/plesk/panel.log

# Log đăng nhập vào Plesk
grep -i "login" /var/log/plesk/panel.log | tail -30

# Log các tác vụ nền (task manager) của Plesk
tail -f /var/log/plesk/tasks.log
```

---

## Bảng vị trí log quan trọng

| Log | Vị trí trong Plesk | Vị trí SSH |
|-----|------------------------|-----------|
| Error log website | **Websites & Domains → Logs** | `/var/www/vhosts/system/<domain>/logs/error_log` |
| Access log website | **Websites & Domains → Logs** | `/var/www/vhosts/system/<domain>/logs/access_log` |
| Email gửi/nhận | **Mail → Mail Settings** | `/var/log/maillog` |
| SSL/Let's Encrypt | **Websites & Domains → SSL/TLS Certificates** | `/var/log/plesk/letsencrypt.log` |
| Traffic/Bandwidth | **Statistics** | — |
| Đăng nhập SSH | — | `/var/log/auth.log` (Ubuntu) / `/var/log/secure` (CentOS) |
| Plesk login | — | `/var/log/plesk/panel.log` |
| Plesk lỗi hệ thống | — | `/var/log/plesk/panel.log` |
| MySQL/MariaDB error | — | `/var/log/mysql/error.log` |
| Web server (server-wide) | — | `/var/log/nginx/error.log` hoặc `/var/log/apache2/error.log` |
| Fail2Ban | — | `/var/log/fail2ban.log` |

---

## References

- [Nhân Hòa – Hướng dẫn tạo và quản lý Database với Plesk](https://wiki.nhanhoa.com/kb/huong-dan-tao-va-quan-ly-database-voi-plesk/)
- [Nhân Hòa – Hướng dẫn tạo bản backup Plesk](https://wiki.nhanhoa.com/kb/huong-dan-tao-ban-backup-plesk/)
- [Nhân Hòa – Hướng dẫn cài đặt chứng chỉ SSL trên Plesk dễ dàng](https://wiki.nhanhoa.com/kb/huong-dan-cai-dat-ssl-tren-plesk-de-dang/)
- [Nhân Hòa – Hướng dẫn tạo tên miền vào hosting Plesk chỉ với 2 bước](https://wiki.nhanhoa.com/kb/huong-dan-tao-ten-mien-vao-hosting-plesk-de-dang-chi-voi-2-buoc/)
- [Nhân Hòa – Hướng dẫn thao tác xử lý log file trên Plesk](https://wiki.nhanhoa.com/kb/plesk-huong-dan-thao-tac-xu-ly-log-file/)
- [Nhân Hòa – Thay đổi và cài đặt các phiên bản PHP trên Plesk](https://wiki.nhanhoa.com/kb/phien-ban-php-tren-plesk/)
- [Nhân Hòa – Chuyên mục Plesk (tổng hợp bài viết)](https://wiki.nhanhoa.com/chuyen-muc/plesk/)
- [Plesk Official Documentation](https://docs.plesk.com/en-US/obsidian/)
- [Plesk Support Center](https://support.plesk.com/hc/en-us)
