# DirectAdmin 

---

## Mục lục

- [1. Tổng quan DirectAdmin](#1-tổng-quan-directadmin)
- [2. Kiến trúc 3 cấp độ](#2-kiến-trúc-3-cấp-độ)
- [3. Giao diện Admin Level](#3-giao-diện-admin-level)
- [4. Giao diện Reseller Level](#4-giao-diện-reseller-level)
- [5. Giao diện User Level](#5-giao-diện-user-level)
- [6. Quản lý DNS](#6-quản-lý-dns)
- [7. Quản lý Email](#7-quản-lý-email)
- [8. Cài đặt SSL](#8-cài-đặt-ssl)
- [9. Các tính năng nâng cao](#9-các-tính-năng-nâng-cao)

---

## 1. Tổng quan DirectAdmin

**DirectAdmin (DA)** là web hosting control panel chạy trên Linux, cho phép quản lý server, domain, email, database, DNS... qua giao diện web.

- **Cổng truy cập mặc định:** `http://server_ip:2222`
- **Hệ điều hành hỗ trợ:** AlmaLinux, CentOS, CloudLinux, Ubuntu

---

## 2. Kiến trúc 3 cấp độ

```
Admin (toàn quyền server)
    └── Reseller (đại lý, quản lý nhiều user)
            └── User (quản lý domain/hosting của mình)
```

| Cấp độ | Quyền hạn | Truy cập |
|--------|-----------|----------|
| **Admin** | Toàn quyền server, tạo reseller/user, quản lý dịch vụ | `ip:2222` với tài khoản admin |
| **Reseller** | Tạo/quản lý user, tạo package, phân bổ tài nguyên | `ip:2222` với tài khoản reseller |
| **User** | Quản lý domain, email, FTP, database, SSL của mình | `ip:2222` với tài khoản user |

---

## 3. Giao diện Admin Level
 Cấp cao nhất. Có quyền kiểm soát toàn bộ máy chủ, cài đặt cấu hình hệ thống, quản lý địa chỉ IP, xem nhật ký truy cập và tạo/phân quyền cho các tài khoản Admin khác, Reseller hoặc User.  
 
### 3.1 Server Management

| Chức năng | Mô tả |
|-----------|-------|
| **Create Administrator** | Tạo tài khoản admin mới |
| **List Administrators** | Xem danh sách tất cả admin |
| **Change Passwords** | Đổi mật khẩu user/reseller/admin |
| **Manage Tickets** | Quản lý ticket hỗ trợ từ user |
| **Create Reseller** | Tạo tài khoản đại lý |
| **List Resellers** | Xem và quản lý danh sách reseller |
| **Manage Reseller Packages** | Tạo/sửa gói tài nguyên cho reseller |
| **Show All Users** | Xem toàn bộ user trên server |

### 3.2 Admin Tools

| Chức năng | Mô tả |
|-----------|-------|
| **IP Management** | Thêm/xóa/phân bổ IP cho user |
| **DNS Administration** | Quản lý DNS zone toàn server |
| **Admin Backup/Transfer** | Backup hoặc chuyển dữ liệu user sang server khác |
| **Multi Server Setup** | Cấu hình nhiều server hoạt động cùng nhau |
| **Mail Queue Administration** | Xem/xóa hàng đợi email đang chờ gửi |
| **Move Users between Resellers** | Chuyển user từ reseller này sang reseller khác |
| **System Information** | Xem thông tin CPU, RAM, disk, uptime |
| **Service Monitor** | Kiểm tra trạng thái Apache, MySQL, FTP... |
| **System Backup** | Backup toàn bộ hệ thống |
| **Log Viewer** | Xem log Apache, email, error... |
| **File Editor** | Chỉnh sửa file cấu hình trực tiếp trên web |
| **Process Monitor** | Xem các tiến trình đang chạy (tương tự lệnh `top`) |

<img width="527" height="432" alt="image" src="https://github.com/user-attachments/assets/2118bf2d-06a3-4dd1-9959-da1bca58dad6" />

#### 3.2.1 Chi tiết Backup & Log ở cấp Admin

| Chức năng | Mô tả chi tiết |
|-----------|----------------|
| **Admin Backup/Transfer** | Backup riêng lẻ theo từng user hoặc theo reseller, lưu local hoặc chuyển thẳng sang server khác qua SSH key |
| **System Backup** | Backup toàn bộ server theo lịch (cron), hỗ trợ Local / FTP / rsync / Amazon S3 |
| **Log Viewer** | Xem tổng hợp log toàn server: Apache error log, DA system log, Exim mail log |

**Lệnh backup thủ công qua CLI (khi cần chạy tay hoặc debug cron backup lỗi):**
```bash
# Chạy tác vụ backup đang nằm trong hàng đợi (dataskq xử lý job admin_backup)
/usr/local/directadmin/dataskq d

# Vị trí lưu backup mặc định của admin
/home/admin/admin_backups/

# Xem log của chính DirectAdmin (lỗi panel, lỗi task queue)
tail -f /var/log/directadmin/error.log
tail -f /var/log/directadmin/system.log
```

### 3.3 Extra Features

| Chức năng | Mô tả |
|-----------|-------|
| **Complete Usage Statistics** | Thống kê băng thông, dung lượng toàn server |
| **Custom HTTPD Configurations** | Thêm cấu hình Apache tùy chỉnh cho từng domain |
| **PHP Configuration** | Cấu hình phiên bản PHP, extension, php.ini |
| **Brute Force Monitor** | Theo dõi và chặn IP tấn công brute force |
| **ConfigServer Security & Firewall** | Quản lý firewall CSF — chặn/mở port, IP |
| **Administrator Settings** | Cài đặt chung của DA |
| **Licensing / Updates** | Kiểm tra license DA và cập nhật phiên bản |
| **Plugin Manager** | Cài/gỡ plugin mở rộng cho DA |
| **All User Cron Jobs** | Xem tất cả cron job của mọi user |
| **CustomBuild 2.0** | Cài/nâng cấp phần mềm server (Apache, PHP, MySQL...) |

---

## 4. Giao diện Reseller Level
Cấp đại lý. Được Admin phân bổ một lượng tài nguyên nhất định, sau đó tự tạo các gói lưu trữ để quản lý và bán lại cho các khách hàng.

<img width="543" height="439" alt="image" src="https://github.com/user-attachments/assets/6eb8e7ef-1d60-452f-8ff2-bbbe0178b7bc" />

### 4.1 Tạo Reseller Package

Trước khi tạo reseller, cần tạo package để gán cho họ.

**Admin Level → Manage Reseller Packages → Add Package**

#### Giới hạn tài nguyên

| Cấu hình | Ý nghĩa |
|----------|---------|
| **Bandwidth (MB)** | Băng thông tối đa/tháng cho toàn bộ user con |
| **Disk Space (MB)** | Dung lượng ổ đĩa cấp phát |
| **Inodes** | Số lượng file/thư mục tối đa |
| **Domains** | Số domain chính có thể tạo |
| **Sub-Domains** | Số subdomain được phép |
| **User Accounts** | Số tài khoản user có thể tạo |
| **IPs** | Số địa chỉ IP riêng |
| **Email Accounts** | Tổng số tài khoản email |
| **Forwarders** | Số địa chỉ email chuyển tiếp |
| **Mailing Lists** | Số danh sách gửi thư |
| **Autoresponders** | Số email trả lời tự động |
| **MySQL Databases** | Số database MySQL |
| **Domain Pointers** | Số domain trỏ về website chính |
| **FTP Accounts** | Số tài khoản FTP |

#### Cấu hình tính năng & quyền truy cập

| Tính năng | Ý nghĩa |
|-----------|---------|
| **CGI Access** | Cho phép chạy CGI script |
| **PHP Access** | Cho phép chạy PHP |
| **SpamAssassin** | Bộ lọc thư rác |
| **SSL Access** | Cho phép cài chứng chỉ SSL |
| **SSH Access** | Truy cập dòng lệnh qua SSH |
| **Overselling** | Cho phép bán vượt giới hạn tài nguyên |
| **Cron Jobs** | Cho phép thiết lập tác vụ định kỳ |
| **System Info** | Cho phép xem thông tin hệ thống |
| **Login Keys** | Tạo khóa đăng nhập không cần mật khẩu |
| **DNS Control** | Cho phép chỉnh sửa bản ghi DNS |
| **Share Server IP** | Dùng chung IP với server |
| **Anonymous FTP** | FTP không cần tài khoản (nên tắt) |
| **Catch-All Email** | Nhận tất cả email kể cả địa chỉ không tồn tại |
| **Personal DNS (3 IPs)** | DNS riêng với 3 địa chỉ IP |

### 4.2 Tạo Reseller

**Admin Level → Create Reseller**

Thông tin cần điền:
- **Username:** Tên đăng nhập
- **E-Mail:** Email quản trị
- **Password:** Mật khẩu
- **Domain:** Tên miền chính
- **Use Reseller Package:** Chọn gói đã tạo
- **Account IP:** Shared
- **Send Email Notification:** Gửi thông báo khi tạo

---

## 5. Giao diện User Level
Cấp độ người dùng cuối. Có quyền thấp nhất, do Admin hoặc Reseller tạo ra. Người dùng không thể tạo thêm tài khoản khác mà chỉ có thể thao tác quản trị website của mình như tạo database, upload mã nguồn, quản lý email và domain.  
 
<img width="479" height="437" alt="image" src="https://github.com/user-attachments/assets/69cc8058-2663-4e35-92af-fcf8db318f08" />

### 5.1 Tạo User Package

Reseller đăng nhập → **Manage User Packages → Add Package**

#### Ví dụ cấu hình gói user cơ bản

| Tài nguyên | Giá trị |
|-----------|---------|
| Bandwidth | 1000 MB/tháng |
| Disk Space | 100 MB |
| Domains | 1 |
| Sub-Domains | 10 |
| Email Accounts | 10 |
| MySQL Databases | 5 |
| FTP Accounts | 1 |
| PHP Access | Enabled |
| SSL Access | Enabled |
| SpamAssassin | Enabled |
| Cron Jobs | Enabled |
| DNS Control | Enabled |
| Suspend At Limit | Enabled (tự khóa nếu vượt giới hạn) |

### 5.2 Tạo User

**Reseller Level → Add New User**

Thông tin cần điền:
- **Username:** Tên đăng nhập
- **E-Mail:** Email quản trị
- **Password:** Mật khẩu
- **Domain:** Tên miền chính
- **User Package:** Chọn gói tài nguyên
- **IP Address:** Shared (dùng chung IP server)

### 5.3 Domain Setup – Những lưu ý quan trọng

**User Level → Domain Setup → Modify domain.com**

| Trường | Ý nghĩa | Lưu ý khi hỗ trợ |
|--------|---------|-------------------|
| **Bandwidth / Disk Space** | Giới hạn riêng cho domain này | Nếu tick **"Same as Main Account"** thì domain dùng chung hạn mức tổng của cả tài khoản; bỏ tick để giới hạn riêng — hữu ích khi user có nhiều domain và muốn chia nhỏ quota |
| **Secure SSL** | Bật khả năng chạy HTTPS cho domain | **Phải tick trước** thì mục *private_html setup* bên dưới mới xuất hiện và SSL Certificates mới cài được lên domain. Nếu user báo "không thấy chỗ cài SSL", kiểm tra ngay ô này |
| **PHP Access** | Bật/tắt chạy PHP trên domain | Tắt sẽ khiến toàn bộ file `.php` bị trình duyệt tải về thay vì thực thi — nguyên nhân phổ biến khi web "tải file lạ" thay vì hiển thị |
| **Force Redirect** | Chuyển hướng cố định về 1 dạng domain: *No redirection* / `www.domain.com` / `domain.com` | Nên chọn 1 dạng cố định (thường là non-www) để tránh trùng lặp nội dung  và tránh lỗi mixed-content khi kết hợp với SSL |
| **private_html setup**  | Chọn cách phục vụ nội dung HTTPS | • **Use a directory named private_html:** HTTP và HTTPS phục vụ 2 bộ nội dung khác nhau <br>• **Use a symbolic link :** HTTP và HTTPS cùng trỏ về `public_html`, dùng chung 1 bộ file — phù hợp với hầu hết website thông thường<br>• **Force SSL with https redirect:** tick thêm ô này để ép toàn bộ traffic HTTP tự chuyển sang HTTPS ngay ở mức domain (cách này đơn giản hơn, thay thế được đoạn `RewriteRule` thủ công ở mục 8.3) |
| **PHP Version Selector** | Chọn version PHP + Handler cho domain | Handler ảnh hưởng hiệu năng & phân quyền file:<br>• **suPHP** (ví dụ `PHP 7.4 suphp`): chạy PHP với quyền của chính user, an toàn hơn nhưng **chậm hơn**, phù hợp shared hosting nhỏ<br>• **mod_php:** nhanh nhưng chạy chung quyền Apache, kém an toàn hơn khi nhiều user share server<br>• **PHP-FPM:** khuyến nghị cho site lượng truy cập cao, cân bằng tốt giữa tốc độ và cô lập tiến trình |

> Khi ticket báo "web lỗi 500 sau khi đổi PHP version" → kiểm tra lại đúng mục này trước, vì đổi Handler hoặc version có thể phá vỡ code cũ.

<img width="481" height="436" alt="image" src="https://github.com/user-attachments/assets/842d7f3f-38c4-40e7-a518-ac40fcd9fe85" />

### 5.4 FTP Management

**User Level → FTP Management → Create FTP account**

| Trường | Mô tả |
|--------|-------|
| **FTP Username** | Tên đăng nhập, hệ thống tự thêm hậu tố `@domain.com` |
| **Enter Password / Re-Enter Password** | Mật khẩu FTP |

**4 tùy chọn thư mục gốc khi tạo FTP**

| Tùy chọn | Thư mục gốc thực tế | Khi nào dùng |
|----------|----------------------|--------------|
| **Domain** | Thư mục home của domain, **cao hơn 1 cấp so với `public_html`** (tức thấy được cả `public_html`, `private_html`, `logs`, `backups`...) | Dùng cho chính chủ website hoặc người cần toàn quyền quản lý domain. Đây là quyền **rộng nhất** |
| **Ftp** | Thư mục `public_ftp` riêng, **tách biệt hoàn toàn với `public_html`** | Dùng khi chỉ cần chia sẻ file qua lại (ví dụ gửi file cho khách, nhận file upload) mà không muốn động vào mã nguồn website |
| **User** | Thư mục trùng tên user, nằm **bên trong `public_html`** (ví dụ `public_html/tenuser/`) | Dùng khi muốn cấp cho 1 người chỉ truy cập đúng 1 thư mục con trong website  |
| **Custom** | Đường dẫn tự nhập, ví dụ `/home/iamhieu` | Dùng khi cần giới hạn FTP vào **đúng 1 thư mục cụ thể bất kỳ** trên hệ thống file của account — linh hoạt nhất, nên dùng khi cấp quyền cho bên thứ ba |

> ⚠️ Lưu ý khi tư vấn khách: nếu khách chỉ cần "gửi/nhận file" thì hướng dẫn chọn **Ftp** hoặc **Custom** trỏ vào thư mục riêng, **không nên** mặc định chọn **Domain** vì sẽ lộ toàn bộ mã nguồn, log, và cả các domain con khác nếu có.

**Quản lý sau khi tạo:**

| Chức năng | Mô tả |
|-----------|-------|
| **FTP Accounts List** | Xem/sửa/xóa các tài khoản FTP hiện có |
| **Quota** | Giới hạn dung lượng riêng cho từng FTP account |
| **Anonymous FTP** | Cho phép truy cập không cần tài khoản — **nên tắt** vì rủi ro bảo mật |

**Thông tin kết nối FTP:**
```
Host: server_ip hoặc domain.com
Port: 21 (FTP) / 22 (SFTP qua SSH nếu bật)
Username: tenftp@domain.com (FTP account phụ) hoặc username DA (FTP chính)
```

> Với hosting dùng chung IP, khuyến nghị dùng **SFTP (port 22)** thay vì FTP thường (21) vì FTP truyền dữ liệu/mật khẩu không mã hóa.

### 5.5 Site Summary / Statistics / Logs

**User Level → Site Summary / Statistics / Logs**

| Chức năng | Mô tả |
|-----------|-------|
| **Bandwidth Usage** | Băng thông đã dùng theo tháng, biểu đồ theo ngày |
| **Disk Usage** | Dung lượng đã dùng: web, email, database, backup |
| **Awstats / Webalizer** | Công cụ thống kê truy cập (visitor, page views, referrer) |
| **Access Log** | Log truy cập web, xem trực tiếp hoặc tải về |
| **Error Log** | Log lỗi Apache/Nginx của domain |

<img width="436" height="440" alt="image" src="https://github.com/user-attachments/assets/6cb3e7da-235f-4387-b45a-cd5a0ca74daa" />

**Đường dẫn log thực tế trên server (Admin có thể xem qua SSH):**
```
/var/log/httpd/domains/domain.com.log         # access log
/var/log/httpd/domains/domain.com.error.log   # error log
/var/log/httpd/domains/domain.com.bytes       # bandwidth log
```

**Lệnh kiểm tra nhanh khi hỗ trợ ticket:**
```bash
# Xem 100 dòng log lỗi mới nhất của domain
tail -n 100 /var/log/httpd/domains/domain.com.error.log

# Theo dõi log real-time (khi user báo lỗi 500)
tail -f /var/log/httpd/domains/domain.com.error.log

# Tìm IP gây lỗi nhiều nhất trong access log
awk '{print $1}' /var/log/httpd/domains/domain.com.log | sort | uniq -c | sort -nr | head -10
```

### 5.6 Backup & Restore

#### Create Backup
 
#### Bước 1 – Who (Chọn đối tượng backup)
 
| Tùy chọn | Mô tả |
|----------|-------|
| **All Users** | Backup toàn bộ user trên server |
| **All Users Except Selected Users** | Backup tất cả trừ các user được chọn |
| **Selected Users** | Chỉ backup các user được chọn  |
| **Skip Suspended** | Bỏ qua các tài khoản đang bị khóa |
 
---
 
#### Bước 2 – When (Thời điểm backup)
 
| Tùy chọn | Mô tả |
|----------|-------|
| **Now** | Chạy backup ngay lập tức |
| **Cron Schedule** | Đặt lịch tự động theo thời gian |
 
Cấu hình Cron Schedule:
 
| Trường | Giá trị | Ý nghĩa |
|--------|---------|---------|
| Minute | 0–59 | Phút |
| Hour | 0–23 | Giờ |
| Day of Month | `*` hoặc 1–31 | Ngày trong tháng (`*` = mỗi ngày) |
| Month | `*` hoặc 1–12 | Tháng |
| Day of Week | `*` hoặc 0–7 | Thứ trong tuần (`*` = mỗi thứ) |
 
> Ví dụ: `Minute=0, Hour=2, */*/*` → backup lúc 2:00 sáng mỗi ngày.
 
---
 
#### Bước 3 – Where (Nơi lưu backup)
 
| Tùy chọn | Mô tả |
|----------|-------|
| **Local** | Lưu tại `/home/admin/admin_backups` trên server |
| **FTP** | Gửi sang server FTP khác |
| **Append to path** | Thêm hậu tố vào thư mục đích (`Nothing` / `Username` / `Domain`...) |
 
Nếu chọn **FTP**, cần điền thêm:
 
| Trường | Mô tả |
|--------|-------|
| IP | Địa chỉ IP server FTP đích |
| Username | Tài khoản FTP |
| Password | Mật khẩu FTP |
| Remote Path | Thư mục lưu trên server FTP (mặc định `/`) |
| Port | Cổng FTP (mặc định `21`) |
| Secure FTP | Bật/tắt FTPS |
 
---
 
#### Bước 4 – What (Nội dung backup)
 
| Tùy chọn | Mô tả |
|----------|-------|
| **All Data** | Backup toàn bộ tất cả hạng mục |
| **Selected Data** | Chọn từng hạng mục cụ thể |
 
Các hạng mục có thể chọn riêng:
 
| Hạng mục | Nội dung |
|----------|---------|
| **Domains Directory** | Toàn bộ file web (`public_html`) |
| **Subdomain Lists** | Danh sách subdomain đã tạo |
| **FTP Accounts** | Tài khoản FTP |
| **FTP Settings** | Cấu hình FTP |
| **Database Settings** | Cấu hình database|
| **Database Data** | Dữ liệu MySQL bên trong database |
| **Deleted Trash Data** | Dữ liệu đã xóa còn trong thùng rác |
| **E-Mail Accounts** | Danh sách tài khoản email |
| **E-Mail Data** | Nội dung hộp thư (email đã nhận/gửi) |
| **E-Mail Settings** | Cấu hình email  |
| **Vacation Messages** | Thư trả lời tự động khi vắng mặt |
| **Autoresponders** | Cấu hình autoresponder |
| **Mailing Lists** | Danh sách gửi thư |
| **Forwarders** | Cấu hình chuyển tiếp email |
 
---
 
#### Bước 5 – Submit
 
Nhấn **Submit** để bắt đầu backup.
 
File được lưu tại `/home/admin/admin_backups/` dưới dạng `username.tar.gz`.
 
 <img width="535" height="391" alt="image" src="https://github.com/user-attachments/assets/0540d5af-0486-4283-a07e-3bfde8492926" />

---

### Restore Backup
 
Phần **Restore Backup** nằm ngay bên dưới trên cùng trang, gồm 3 bước:
 
#### Bước 1 – From Where
 
Chọn nguồn file backup:
 
| Tùy chọn | Mô tả |
|----------|-------|
| **Local** | File đang có sẵn trên server (`/home/admin/admin_backups/`) |
| **FTP** | Lấy file từ server FTP khác |
 
#### Bước 2 – Select IP
 
Chọn địa chỉ IP sẽ gán cho dữ liệu sau khi restore.
 
#### Bước 3 – Select File(s)
 
Chọn file `.tar.gz` cần restore → nhấn **Submit** để bắt đầu.
 
> ⚠️ **Lưu ý:** Restore sẽ **ghi đè** dữ liệu hiện tại của user/domain đó.  
> Luôn tạo backup bản hiện tại trước khi restore bản cũ.
 
---

### 5.7 Login History

**User Level → Login History**

| Chức năng | Mô tả |
|-----------|-------|
| **Xem lịch sử đăng nhập** | IP, thời gian, thành công/thất bại vào DA panel |
| **Phát hiện bất thường** | Dùng để kiểm tra khi nghi ngờ tài khoản bị dò mật khẩu |

<img width="545" height="384" alt="image" src="https://github.com/user-attachments/assets/b9b3df2e-aa6a-4b48-a966-e656d73757ce" />

---

## 6. Quản lý DNS

### 6.1 Các loại bản ghi DNS cần biết

| Loại | Mục đích | Ví dụ |
|------|----------|-------|
| **A** | Trỏ domain/subdomain về IP | `@ → 103.170.123.213` |
| **CNAME** | Tạo alias trỏ về domain khác | `www → hieucute.id.vn.` |
| **MX** | Chỉ định mail server | `@ → mail.hieucute.id.vn.` priority 10 |
| **TXT** | SPF, DKIM, DMARC, xác minh domain | `v=spf1 ip4:x.x.x.x ~all` |
| **NS** | Chỉ định nameserver | `@ → ns1.example.com.` |
| **PTR** | Reverse DNS (IP → domain) | Cấu hình ở nhà cung cấp IP |

và các bản ghi khác DKIM, DMARC

### 6.2 Bộ bản ghi DNS đầy đủ cho mail

Ví dụ domain `hieucute.id.vn`, server IP `103.170.123.213`:

#### Bước 1 – Bản ghi cơ bản

| STT | Host | Type | Value | TTL |
|-----|------|------|-------|-----|
| 1 | `@` | A | `103.170.123.213` | 300 |
| 2 | `mail` | A | `103.170.123.213` | 300 |
| 3 | `www` | CNAME | `hieucute.id.vn.` | 300 |
| 4 | `@` | MX | `mail.hieucute.id.vn.` | 300 | Priority 10 |

> ⚠️ MX phải trỏ vào **A record** (`mail.hieucute.id.vn.`), **không trỏ vào CNAME**.

#### Bước 2 – SPF (chống giả mạo email)

| Host | Type | Value |
|------|------|-------|
| `@` | TXT | `v=spf1 ip4:103.170.123.213 ~all` |

#### Bước 3 – DKIM (chữ ký số email)

| Host | Type | Value |
|------|------|-------|
| `default._domainkey` | TXT | `v=DKIM1; k=rsa; p=<public_key_từ_server>` |

> Trong DirectAdmin: vào Email Manager → DKIM is enabled → copy key → thêm vào DNS.

#### Bước 4 – DMARC (chính sách xử lý mail giả mạo)

| Host | Type | Value |
|------|------|-------|
| `_dmarc` | TXT | `v=DMARC1; p=none; rua=mailto:admin@hieucute.id.vn` |

<img width="938" height="424" alt="image" src="https://github.com/user-attachments/assets/4ea3115d-78b0-4edf-bd41-8af2a47ef530" />

### 6.3 Lộ trình triển khai DMARC

```
Giai đoạn 1 (mới cài):    p=none      → chỉ theo dõi, không chặn
Giai đoạn 2 (sau 1-2 tuần): p=quarantine → mail nghi ngờ vào Spam
Giai đoạn 3 (ổn định):    p=reject    → từ chối hoàn toàn mail giả mạo
```

---

## 7. Quản lý Email

### 7.1 Tạo tài khoản email

**User Level → E-Mail Manager → Create Mail Account**

- Nhập tên (ví dụ: `iamhieu`) → tự động tạo `iamhieu@hieucute.id.vn`
- Đặt mật khẩu và quota

<img width="560" height="206" alt="image" src="https://github.com/user-attachments/assets/654964e0-bd5d-4e26-9a45-2c7478c564f7" />

### 7.2 Đăng nhập Webmail

**Lưu ý quan trọng:** Phải nhập **đầy đủ email**, không chỉ tên:

| Trường | Đúng | Sai |
|--------|------|-----|
| Username | `iamhieu@hieucute.id.vn` | `iamhieu` |

Địa chỉ truy cập webmail:
```
http://mail.hieucute.id.vn/webmail
http://server_ip/webmail
```

Hoặc trong DirectAdmin panel → nhấn nút **Webmail** ở menu trên → tự động đăng nhập.

### 7.3 Các tính năng Email trong DirectAdmin

| Tính năng | Mô tả |
|-----------|-------|
| **Email Accounts** | Tạo/xóa/quản lý hộp thư |
| **Forwarders** | Chuyển tiếp email sang địa chỉ khác |
| **Autoresponders** | Email trả lời tự động khi vắng mặt |
| **Mailing Lists** | Danh sách gửi thư hàng loạt |
| **SpamAssassin** | Lọc thư rác tự động |
| **DKIM** | Chữ ký số cho email (bật/tắt trong Email Manager) |
| **Catch-All** | Nhận tất cả email dù địa chỉ không tồn tại |
| **Mail Queue** | Xem hàng đợi email đang chờ gửi |

<img width="378" height="103" alt="image" src="https://github.com/user-attachments/assets/f341d978-c045-47dc-b66a-bc22242bf69e" />

### 7.4 Bật Let's Encrypt + DKIM trong DirectAdmin

**Bật Let's Encrypt:**
```bash
echo "letsencrypt=1" >> /usr/local/directadmin/conf/directadmin.conf
echo "enable_ssl_sni=1" >> /usr/local/directadmin/conf/directadmin.conf
/etc/init.d/directadmin restart
```

Trong giao diện: SSL Certificates → Free & automatic certificate from Let's Encrypt → Save.

<img width="353" height="348" alt="image" src="https://github.com/user-attachments/assets/4cdff3e0-2267-435b-a19b-62927dcebd3f" />

---

## 8. Cài đặt SSL

### 8.1 Các tùy chọn SSL trong DirectAdmin

Vào **User Level → SSL Certificates**:

| Tùy chọn | Mô tả |
|----------|-------|
| Use the server's certificate | Dùng SSL mặc định của server |
| Create a self-signed certificate | Tạo cert tự ký (test nội bộ) |
| Create a certificate request | Tạo CSR để xin cert từ CA |
| Free & automatic (Let's Encrypt) | Miễn phí, tự gia hạn |
| Paste a pre-generated certificate | Dán cert đã có sẵn từ CA khác |



### 8.2 Cài SSL có sẵn (từ ZeroSSL, Sectigo...)

1. SSL Certificates → **Paste a pre-generated Certificate and key**
2. Paste nội dung Private Key + Certificate → Save
3. Click **"Click Here to paste a CA Root Certificate"** → paste CA Bundle
4. Quay lại Domain Setup → domain → **Use a symbolic link** → Save

<img width="353" height="419" alt="image" src="https://github.com/user-attachments/assets/084e84c1-3e39-425a-bce7-4c8516a50849" />

### 8.3 Cấu hình HTTP Redirect (HTTP → HTTPS)

**Cách 1 – Trong DirectAdmin:**
Domain Setup → chọn domain → tích **Force SSL with HTTP redirect** → Save

**Cách 2 – Trong Custom HTTPD Configurations:**
```apache
RewriteEngine On
RewriteCond %{HTTPS} !=on
RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [R=301,L]
```

### 8.4 Cấu hình HSTS

Admin Level → **Extra Features → Custom HTTPD Configurations** → chọn domain → Custom1:

```apache
Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"
```
<img width="624" height="368" alt="image" src="https://github.com/user-attachments/assets/77a487fa-2e75-48cf-848a-d134b8b6ec46" />

### 8.5 Cấu hình OCSP Stapling

Trong **Custom HTTPD Configurations** → Http.conf Customization:
```apache
SSLUseStapling on
SSLStaplingResponderTimeout 5
SSLStaplingReturnResponderErrors off
SSLStaplingFakeTryLater off
SSLStaplingStandardCacheTimeout 86400
```

Trong **Custom1**:
```apache
SSLStaplingCache "shmcb:logs/ssl_stapling(32768)"
```

### 8.6 Kiểm tra SSL

```bash
# Kiểm tra cert đang dùng
openssl s_client -connect hieucute.id.vn:443 -status

# Kiểm tra OCSP Stapling
openssl s_client -connect hieucute.id.vn:443 -status | findstr OCSP
```

Kiểm tra online:
- [https://www.ssllabs.com/ssltest/](https://www.ssllabs.com/ssltest/)
- [https://www.sslshopper.com/ssl-checker.html](https://www.sslshopper.com/ssl-checker.html)

---

## 9. Các tính năng nâng cao

### 9.1 CustomBuild 2.0

Công cụ cài đặt và nâng cấp phần mềm server trong DirectAdmin.

```bash
cd /usr/local/directadmin/custombuild

# Kiểm tra phiên bản
./build version

# Cập nhật danh sách
./build update

# Cài/cập nhật Let's Encrypt
./build letsencrypt

# Áp dụng lại cấu hình
./build rewrite_confs
```

> Nếu badge CustomBuild hiển thị số → vào cập nhật ngay để vá lỗi bảo mật.

### 9.2 ConfigServer Security & Firewall (CSF)

Firewall tích hợp trong DirectAdmin, cho phép:
- Chặn/mở port
- Chặn/mở IP
- Theo dõi đăng nhập thất bại
- Tự động chặn IP tấn công brute force

### 9.3 Brute Force Monitor

Tự động phát hiện và chặn IP đăng nhập sai nhiều lần vào:
- SSH
- FTP
- cPanel/DirectAdmin panel
- Webmail

### 9.4 Cron Jobs

Thiết lập tác vụ chạy tự động theo lịch.

Ví dụ tự động gia hạn Let's Encrypt:
```bash
0 12 * * * /usr/bin/certbot renew --quiet
```

---
