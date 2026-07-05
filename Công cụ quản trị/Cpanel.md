# Hướng dẫn sử dụng cPanel (2083) & WHM (2087)

> **Hệ điều hành:** Ubuntu v22.04.5 STANDARD kvm
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
- [2. WHM – Quản trị cấp Server](#2-whm--quản-trị-cấp-server)
- [3. Quản lý Tên miền & Web (cPanel)](#3-quản-lý-tên-miền--web-cpanel)
- [4. Quản lý File](#4-quản-lý-file)
- [5. Quản lý Tài khoản FTP](#5-quản-lý-tài-khoản-ftp)
- [6. Quản lý Cơ sở dữ liệu MySQL](#6-quản-lý-cơ-sở-dữ-liệu-mysql)
- [7. Bảo mật – Kích hoạt SSL](#7-bảo-mật--kích-hoạt-ssl)
- [8. Quản lý Email & Chống Spam](#8-quản-lý-email--chống-spam)
- [9. Mailing List](#9-mailing-list)
- [10. Cron Jobs](#10-cron-jobs)
- [11. Backup & Restore](#11-backup--restore)
- [12. Triển khai Next.js trên cPanel](#12-triển-khai-nextjs-trên-cpanel)
- [13. 5 lỗi phổ biến nhất](#13-5-lỗi-phổ-biến-nhất)
- [14. Thực chiến: Tạo Package → Account → SSL → WordPress](#14-thực-chiến-tạo-package--account--ssl--wordpress)
- [15. Mapping kiến thức DirectAdmin sang cPanel/WHM](#15-mapping-kiến-thức-directadmin-sang-cpanelwhm)

---

## 1. Tổng quan cổng truy cập

cPanel/WHM tách biệt rõ ràng theo cổng — khác hoàn toàn với DirectAdmin dùng chung một cổng 2222 cho mọi cấp:

| Cổng | Dịch vụ | Dành cho | Truy cập thực tế |
|------|---------|----------|-----------------|
| **2087** | WHM (HTTPS) | Root / Reseller — quản trị toàn server | `https://103.159.51.228:2087` |
| **2083** | cPanel (HTTPS) | User — quản lý 1 tài khoản hosting | `https://103.159.51.228:2083` |
| **2096** | Webmail (HTTPS) | Người dùng cuối — chỉ đọc/gửi email | `https://103.159.51.228:2096` |

> DA dùng port 2222 cho tất cả (phân biệt bằng tài khoản đăng nhập). cPanel/WHM tách vật lý theo cổng — Admin/Reseller vào 2087, khách hàng vào 2083, người dùng email vào 2096.


---

## 2. WHM – Quản trị cấp Server

> WHM (Web Host Manager) là panel dành cho **root hoặc reseller**, quản lý toàn bộ server: nhiều tài khoản cPanel, dịch vụ hệ thống, firewall, backup server...

### 2.1 Tạo Package (Gói hosting)

Package định nghĩa giới hạn tài nguyên cho một nhóm tài khoản. Tạo Package **một lần**, dùng lại cho mọi khách cùng gói — tương đương "User Package" bên DirectAdmin.

**Đường dẫn:** WHM → **Packages** → **Add a Package**

**Bước 1:** Điền tên Package — đặt tên có ý nghĩa, không dấu, không khoảng trắng.

**Bước 2:** Cấu hình tài nguyên theo chuẩn gói Doanh Nghiệp Nhân Hòa:

| Trường | Giá trị ví dụ | Ghi chú |
|--------|--------------|---------|
| Package Name | `doanhnghiep_package` | Không dấu, không cách |
| Disk Quota | `6144` MB (6 GB) | Khớp với thực tế trên server (Ảnh 1: Quota 6 GB) |
| Bandwidth | `Unlimited` | Hoặc 50000 MB tùy gói |
| Max Email Accounts | `Unlimited` | |
| Max Mailing Lists | `10` | |
| Max Databases | `6` | Khớp Ảnh 3: Databases 0/6 |
| Max FTP Accounts | `Unlimited` | |
| Max Subdomains | `Unlimited` | Ảnh 3: Subdomains 0/∞ |
| Max Parked Domains (Alias) | `Unlimited` | Ảnh 3: Alias Domains 0/∞ |
| Max Addon Domains | `3` | Ảnh 1 & 3: Addon Domains 0/3 |
| CGI Access | ✅ Bật | |
| Shell Access | ❌ Tắt | Khách thường không cần SSH |
| Feature List | `default` | |

**Bước 3:** Nhấn **Add** → Package xuất hiện trong danh sách.

<img width="779" height="440" alt="image" src="https://github.com/user-attachments/assets/e72393d5-2559-4f29-990d-931c153ea453" />

>  **Lưu ý:** Không thể xóa Package đang được gán cho tài khoản. Muốn xóa phải chuyển toàn bộ account sang Package khác trước.

---

### 2.2 Tạo tài khoản mới (Create a New Account)

**Đường dẫn:** WHM → **Account Functions** → **Create a New Account**

**Bước 1 – Domain Information:**

| Trường | Giá trị ví dụ | Quy tắc bắt buộc |
|--------|--------------|-----------------|
| Domain | `hieucute.id.vn` | Nhập đúng FQDN, không có `http://` hay `/` |
| Username | `iamhieu` | **không đặt là `root`**, đặt `root` sẽ xung đột quyền và gây lỗi nghiêm trọng |
| Password | *(dùng Password Generator)* | bước 2 |
| Email | `admin@hieucute.id.vn` | Email thật để nhận thông báo hệ thống |


>  **Quy tắc đặt Username:**
> - Viết liền, không dấu, không khoảng trắng, chỉ dùng chữ thường và số.
> - Tối đa 16 ký tự.
> - Tránh các tên nhạy cảm: `root`, `admin`, `test`, `mail`, `ftp`.
> - Username này sẽ là tên thư mục home (`/home/iamhieu`) và là prefix của mọi database, FTP account trong cPanel.

**Bước 2 – Tạo mật khẩu mạnh:**

Nhấn nút **Password Generator** → hệ thống tạo mật khẩu ngẫu nhiên đạt **100/100 điểm** → nhấn **Use Password** → copy lưu lại.

**Bước 3 – Chọn Package:**

Tại mục **Package** → chọn `doanhnghiep_package` .

**Bước 4 – Mail Routing Settings:**

Chọn **Automatically Detect Configuration**.

> Tùy chọn này để cPanel tự xác định cấu hình email phù hợp dựa trên DNS hiện tại. Chỉ chọn **Local Mail Exchanger** khi chắc chắn server này xử lý email nội bộ hoàn toàn và MX record đã trỏ về đúng IP.

**Bước 5 – DNS Settings (cực kỳ quan trọng):**

Tích chọn đầy đủ cả 3 mục:

| Tùy chọn | Trạng thái | Lý do |
|----------|-----------|-------|
|  **Enable DKIM** | Bật | Chữ ký số xác thực email — thiếu là email vào Spam |
|  **Enable SPF** | Bật | Xác thực IP được phép gửi mail — thiếu là email vào Spam |
|  **Enable DMARC** | Bật | Chính sách xử lý email giả mạo — bảo vệ domain khỏi bị giả mạo |

>  Nếu bỏ qua bước này, email từ `hieucute.id.vn` sẽ bị Gmail, Outlook đánh dấu Spam ngay từ đầu. Kích hoạt sau khi tạo account phức tạp hơn nhiều so với bật ngay từ lúc tạo.

**Bước 6:** Nhấn **Create** → quan sát log cuối trang.

<img width="643" height="445" alt="image" src="https://github.com/user-attachments/assets/7730dbea-603f-494c-8905-6308434691f1" />

---

### 2.3 Quản lý tài khoản hiện có

**Suspend / Unsuspend** 

WHM → **Account Functions** → **Manage Account Suspension**

>  Ghi rõ lý do suspend — khách sẽ thấy thông báo này khi truy cập website.

<img width="674" height="367" alt="image" src="https://github.com/user-attachments/assets/0c84f28b-ef12-4bd9-b20e-5519853f9bcf" />

**Đổi Package / Nâng cấp tài khoản:**

WHM → **Account Functions** → **Modify an Account** → chọn username → đổi Package → **Save**.

<img width="724" height="434" alt="image" src="https://github.com/user-attachments/assets/65196393-aea0-4f5e-a3ef-d38265920e1b" />

**Terminate (xóa vĩnh viễn):**

WHM → **Account Functions** → **Terminate a Multi-User Account**

> Thao tác này xóa toàn bộ dữ liệu (file, database, email) không thể hoàn tác. Bắt buộc tạo Full Backup trước khi terminate (xem mục 11).

<img width="875" height="421" alt="image" src="https://github.com/user-attachments/assets/cc5d3c91-34a1-4b7e-9733-91300eeec2ce" />

---

### 2.4 Quản lý dịch vụ hệ thống

**Restart dịch vụ:**

WHM → **Restart Services** → chọn dịch vụ (Apache, MySQL, Exim...) → **Restart**.

<img width="518" height="182" alt="image" src="https://github.com/user-attachments/assets/ec338fd7-d8fa-43db-aaea-ccaf6a86d600" />

**Kiểm tra trạng thái dịch vụ:**

WHM → **Server Status** → **Service Status** — hiển thị màu xanh/đỏ cho từng dịch vụ.

<img width="800" height="439" alt="image" src="https://github.com/user-attachments/assets/bf9bae76-aa37-4773-9050-3aecac3bfc01" />

**Giám sát tài nguyên:**

WHM → **System health** → **Process Manager** (tương đương lệnh `top`).

<img width="944" height="434" alt="image" src="https://github.com/user-attachments/assets/fa664dde-ea8d-4cf7-aed3-4ce734ee9d7a" />

---

### 2.5 Quản lý DNS cấp server

WHM → **DNS Functions** → **Edit DNS Zone** — sửa zone của bất kỳ domain nào trên server.

<img width="542" height="430" alt="image" src="https://github.com/user-attachments/assets/a6c11513-3abf-4214-aa42-83517303a4e0" />

---

### 2.6 SSL cấp server

- **Manage AutoSSL** → chọn provider (Let's Encrypt hoặc Sectigo), chạy AutoSSL cho toàn bộ hoặc từng account.
- **Install an SSL Certificate on a Domain** → cài cert cho hostname của chính server.

<img width="948" height="431" alt="image" src="https://github.com/user-attachments/assets/159dedbd-238f-4c1c-b3d2-b0c6dcaaf0ab" />

---



### 2.7 Cấu Hình Backup Configuration
**Đường dẫn truy cập:** `WHM` → `Backup` → `Backup Configuration`

### 2.7.1 Bật Tính Năng Backup
* Chuyển đổi trạng thái **Backup Status** sang **Enable**.  
*  Nếu tùy chọn này bị tắt (`Disable`), toàn bộ các cấu hình chi tiết bên dưới sẽ không có hiệu lực và hệ thống sẽ không thực hiện sao lưu.

### 2.7.2  (Backup Type)

| Loại Backup | Mô tả chi tiết | Ngữ cảnh sử dụng khuyến nghị |
| :--- | :--- | :--- |
| **Compressed** | Nén dữ liệu thành định dạng `.tar.gz`. Giúp tiết kiệm dung lượng ổ cứng tối đa nhưng tiêu tốn nhiều tài nguyên CPU khi nén và giải nén. | Phù hợp với server có ít account, dung lượng ổ đĩa chứa backup hạn chế. |
| **Uncompressed** | Giữ nguyên định dạng file, không nén. Tốc độ backup và restore cực nhanh nhưng chiếm dụng không gian lưu trữ lớn. | Phù hợp với server có nhiều account, yêu cầu tốc độ xử lý cao và ổ đĩa backup còn dư dả. |
| **Incremental (rsync)** | Chỉ sao lưu những phần dữ liệu có sự thay đổi. | Phù hợp với server dữ liệu lớn, cần backup hàng ngày nhằm tiết kiệm tối đa thời gian và băng thông. |

>   Loại *Incremental* mang lại hiệu quả cao nhất về thời gian và dung lượng. Tuy nhiên, nó không xuất ra file `.tar.gz` đơn lẻ để user dễ dàng tải về máy cá nhân. Việc restore bắt buộc phải thực hiện thông qua giao diện quản trị WHM thay vì tải file di chuyển thủ công.

<img width="451" height="337" alt="image" src="https://github.com/user-attachments/assets/611ff5da-f155-435f-8510-7836725fbea2" />

### 2.7.3 Thiết Lập Schedule
Mỗi loại lịch chạy đều có ô **"Retain X"** tương ứng với số lượng bản backup gần nhất được giữ lại trước khi hệ thống tự động xóa bản cũ .

* **Daily (Hàng ngày):** Chọn các ngày cụ thể trong tuần (Ví dụ: Chỉ chạy từ thứ 2 đến thứ 6). *Retention* được tính riêng cho Daily. (Ví dụ: Giữ 7 bản = Lưu trữ xoay vòng 7 ngày gần nhất).
* **Weekly (Hàng tuần):** Chọn một ngày cố định trong tuần để chạy. *Retention* tính riêng cho Weekly.
* **Monthly (Hàng tháng):** Chọn ngày cụ thể trong tháng (Ngày 1, ngày 15...). *Retention* tính riêng cho Monthly.

<img width="502" height="364" alt="image" src="https://github.com/user-attachments/assets/3e4b0593-40a9-4ec7-891b-14d89e8b9013" />

### 2.7.4 Lựa Chọn Dữ Liệu Sao Lưu 

* **Accounts (Toàn bộ tài khoản):**  Đây là thành phần cốt lõi và quan trọng nhất chứa toàn bộ dữ liệu khách hàng.
* **System Files (Cấu hình hệ thống):**  Sao lưu các thư mục hệ thống như `/etc`, cấu hình Apache, Exim... 
* **Suspended Accounts (Tài khoản bị khóa):**  Đảm bảo dữ liệu của các tài khoản đang bị tạm ngưng vẫn được an toàn phòng trường hợp được kích hoạt lại.
* **Bandwidth Data (Số liệu băng thông):** Lưu trữ số liệu thống kê lưu lượng, không quá bắt buộc.

<img width="431" height="371" alt="image" src="https://github.com/user-attachments/assets/6bf67e55-68d7-4fb4-a27d-5e03048681eb" />

### 2.7.5 Cấu Hình Nơi Lưu Trữ Ngoại Vi (Additional Destinations)
>  Tuyệt đối không lưu trữ backup trên cùng một ổ đĩa vật lý/partition với dữ liệu gốc. Nếu máy chủ gặp sự cố phần cứng hỏng ổ đĩa, toàn bộ dữ liệu gốc và dữ liệu backup sẽ mất sạch. Nên áp dụng chiến lược **3-2-1** (3 bản sao, 2 loại phương tiện lưu trữ, 1 bản lưu off-site).

**Đường dẫn:** `WHM` → `Backup` → `Backup Configuration` → `Additional Destinations`

* **Local Directory:** Lưu sang một phân vùng (partition) hoặc ổ đĩa gắn thêm (`mount`) độc lập trên cùng server. (Mức độ an toàn: Thấp).
* **FTP:** Cần khai báo *Host, Username, Password, Remote Directory*. Phổ biến khi doanh nghiệp có cụm server backup riêng.
* **SFTP/SCP:** Tương tự FTP nhưng bảo mật hơn nhờ chạy trên giao thức mã hóa (Port 22, hỗ trợ xác thực bằng Private Key hoặc Password).
* **Amazon S3 / S3-Compatible:** Kết nối qua *Access Key, Secret Key, Bucket name*. Đây là chuẩn Production tối ưu, tách biệt hoàn toàn dữ liệu khỏi hạ tầng vật lý của server gốc.
* **Google Drive / Backblaze / Rsync to remote:** Tùy thuộc vào nhà cung cấp dịch vụ hạ tầng của doanh nghiệp.

<img width="446" height="308" alt="image" src="https://github.com/user-attachments/assets/99a3586b-5b5f-46fc-8d64-ac0fc646d232" />

### 2.7.6 Hệ Thống Cảnh Báo (Notification)
* **Đường dẫn:** `WHM` → `Backup` → `Backup Configuration` → `Notifications`
* **Thao tác:** Điền chính xác địa chỉ email nhận báo cáo.
* *Lưu ý:* Luôn bật tính năng này để nhận báo cáo sau mỗi phiên chạy (Thành công/Thất bại). Tránh tình trạng hệ thống lỗi ngầm nhiều tuần liền mà quản trị viên không hay biết, dẫn đến không có dữ liệu khôi phục khi xảy ra sự cố thực tế.

---

## 2.7.8 Cấu Trúc File cpmove — Hiểu Để Restore Đúng

Sau khi Backup Configuration chạy hoàn tất, hệ thống sẽ sinh ra file có định dạng chuẩn:
`cpmove-abccompany.tar.gz` (Trong đó `abccompany` là username của tài khoản).

Khi tiến hành giải nén cấu trúc file này, bên trong sẽ bao gồm:
* `homedir/`: Chứa toàn bộ mã nguồn website, mã nguồn email (`public_html`, `mail`...).
* `mysql/` hoặc `mysql.sql`: Bản dump toàn bộ cơ sở dữ liệu của tài khoản hoặc từng file `.sql` riêng lẻ cho từng database.
* `meta/`: Thông tin cấu hình hệ thống của tài khoản .
* `userdata/`: Cấu hình Virtual Host của Apache dành riêng cho account đó.
* `homedir_paths.yaml`: Bản đồ ánh xạ đường dẫn thư mục.

>  Đây chính là lý do vì sao một file Full Account Backup dạng `cpmove` không thể khôi phục thông qua giao diện cPanel của user thường. Nó chứa các siêu dữ liệu cấu hình hệ thống (Metadata) cần quyền tối cao (`root`) để ghi đè vào các file cấu hình trục dọc của máy chủ WHM.

---

## 2.8 Hướng Dẫn Thực Chiến Khôi Phục Dữ Liệu (Restore)

### Cách 1: Restore Qua Giao Diện WHM (Phổ biến, trực quan)
**Đường dẫn:** `WHM` → `Backup` → `Restore a Full Backup/cpmove File`

* **Bước 1: Chọn nguồn file (Source)**
    * *Restore from Home Directory:* Chọn nếu file `cpmove-*.tar.gz` đã được định vị sẵn trên server (nằm trong thư mục `/home` do hệ thống tự backup hoặc do bạn upload qua SFTP trước đó).
    * *Upload File:* Upload trực tiếp file từ máy tính cá nhân qua trình duyệt (Chỉ khuyên dùng với file dung lượng nhỏ để tránh lỗi timeout trình duyệt).
* **Bước 2: Chọn tài khoản cần restore**
    * Hệ thống WHM sẽ tự động quét và nhận diện tên `username` dựa trên cấu trúc tên file `cpmove-username.tar.gz`.
* **Bước 3: Chọn các tùy chọn nâng cao**
    * `Restore as Suspended`: Khôi phục xong dữ liệu nhưng giữ tài khoản ở trạng thái khóa. Nên chọn để quản trị viên có thời gian kiểm tra lại file/DB trước khi cho chạy public.
    * `Overwrite existing account`: Ghi đè lên tài khoản nếu nó đang tồn tại sẵn trên server. *(Cực kỳ cẩn trọng vì hành động này sẽ xóa sạch dữ liệu hiện tại của account đó)*.
* **Bước 4:** Nhấn **Submit** → Theo dõi tiến trình qua log hiển thị thời gian thực cho đến khi xuất hiện thông báo thành công.
* **Bước 5:** Nếu có tick chọn ở bước 3, tiến hành kiểm tra mã nguồn, sau đó vào quản lý tài khoản để **Unsuspend** thủ công.

<img width="809" height="368" alt="image" src="https://github.com/user-attachments/assets/703cb11c-06f0-47cf-b627-3e8f258d13e7" />

### Cách 2: Restore Qua Giao Diện Dòng Lệnh SSH (Nhanh, tối ưu cho file lớn)
Áp dụng khi file backup có dung lượng lớn (vài chục đến hàng trăm GB) để tránh các giới hạn hoặc ngắt kết nối của trình duyệt web. File backup cần được di chuyển sẵn vào thư mục `/home`.

1. **Kiểm tra sự tồn tại và tính toàn vẹn của file:**
   ```bash
   ls -lh /home/cpmove-abccompany.tar.gz


---



## 3. Quản lý Tên miền & Web (cPanel)

### 3.1 Trỏ tên miền về VPS

Trước khi làm bất kỳ thao tác nào trên cPanel, DNS của `hieucute.id.vn` phải trỏ đúng về IP VPS.

**Đăng nhập trang quản lý tên miền tại Nhân Hòa** → Zone DNS → kiểm tra/thêm:

| Host | Type | Value | TTL |
|------|------|-------|-----|
| `@` | A | `103.159.51.228` | 300 |
| `www` | CNAME | `hieucute.id.vn.` | 300 |
| `mail` | A | `103.159.51.228` | 300 |

**Kiểm tra propagate:**
```bash
dig A hieucute.id.vn +short
# Kết quả đúng: 103.159.51.228
```
<img width="347" height="116" alt="image" src="https://github.com/user-attachments/assets/b1d511d8-9ea5-4fcf-84d2-3816a73aeffe" />

>  Không tiếp tục bất kỳ bước nào (cài SSL, cài WordPress...) nếu DNS chưa trả về đúng IP. AutoSSL sẽ luôn thất bại nếu DNS sai.

---

### 3.2 Thêm Domain mới

**Đường dẫn:** cPanel → mục **Domains**  → **Domains**

**Bước 1:** Nhấn **Create A New Domain**.

**Bước 2:** Điền thông tin:

| Trường | Giá trị ví dụ | Ghi chú |
|--------|--------------|---------|
| Domain | `shop.hieucute.id.vn` | Nhập FQDN — không có `www`, không có `http://` |
| Document Root | `/home/iamhieu/shop.hieucute.id.vn` | Tự động điền, có thể sửa |
| Share document root | **BỎ TICK** | Xem cảnh báo bên dưới |

>  **CẢNH BÁO — Share document root:**
> - Nếu **tick** ô này → domain mới sẽ hiển thị **y hệt nội dung** domain chính (`hieucute.id.vn`). Đây thường là **nhầm lẫn không mong muốn** và **rất khó hoàn tác** sau khi đã có dữ liệu.
> - **Quy tắc:** Luôn **bỏ tick** trừ khi bạn có chủ đích muốn domain phụ hiển thị cùng nội dung.

**Bước 3:** Nhấn **Submit**.

**Bước 4:** Thêm A record tại DNS Nhân Hòa:
```
shop.hieucute.id.vn.   A   103.159.51.228   TTL 300
```


---

### 3.3 Cài đặt WordPress qua WordPress Management

**Đường dẫn:** cPanel → sidebar trái → **WordPress Management** (hoặc mục **Domains** → **WordPress Management**)

**Bước 1:** Nhấn **Install WordPress** (hoặc **Install** nếu đã vào trang quản lý).

<img width="509" height="199" alt="image" src="https://github.com/user-attachments/assets/d1332952-8cb1-44b4-8e7c-0144a02687b1" />

**Bước 2:** Điền thông tin cài đặt:

| Trường | Giá trị thực tế | Quy tắc |
|--------|----------------|---------|
| Domain | `hieucute.id.vn` | Chọn domain đúng từ dropdown |
| Directory | Để trống | Cài tại root — `https://hieucute.id.vn/` |
| WordPress version | Latest stable | Luôn chọn phiên bản mới nhất |
| Admin username | `admin_hieucute` | **Không dùng `admin`** — dễ bị brute force |
| Admin password | *(Password Generator)* | Tối thiểu 12 ký tự |
| Admin email | `admin@hieucute.id.vn` | Email thật để nhận thông báo WP |
| Site Title | `iamhieu` | - |
| Site Language | `Vietnamese` | |

<img width="624" height="335" alt="image" src="https://github.com/user-attachments/assets/60338c5e-e910-469b-8f45-ec6f88483616" />

**Bước 3:** Nhấn **Install** → chờ 1–2 phút.

**Bước 4:** Truy cập `https://hieucute.id.vn/wp-admin` để đăng nhập.

**Tính năng nổi bật của WordPress Management (WP Toolkit):**

| Tính năng | Mô tả |
|-----------|-------|
| Smart Update | Cập nhật WP/plugin an toàn, tự rollback nếu lỗi |
| Security Check | Quét lỗ hổng bảo mật, gợi ý khắc phục |
| Clone | Tạo bản staging để test trước khi deploy |
| Backup/Restore | Backup từng WP site độc lập |

<img width="935" height="431" alt="image" src="https://github.com/user-attachments/assets/bd434ef0-7ec8-41c3-8c37-8e13ce220a15" />


---

### 3.4 Cài WordPress qua Softaculous (thay thế)

**Đường dẫn:** cPanel → **Software** → **Softaculous Apps Installer** → tìm **WordPress** → **Install Now** → điền thông tin → **Install**.

---

## 4. Quản lý File

### 4.1 Upload file lên public_html

**Cách 1: File Manager (dùng cho file nhỏ, thao tác nhanh)**

**Đường dẫn:** cPanel → mục **Files** → **File Manager**

<img width="959" height="345" alt="image" src="https://github.com/user-attachments/assets/94f4ffee-07e5-4e02-a9e2-f9e2a2270c25" />

**Bước 1:** Trong cây thư mục bên trái, click vào **public_html**.

**Bước 2:** Nhấn **Upload** trên thanh công cụ.

**Bước 3:** Kéo thả file vào vùng upload hoặc nhấn **Select File** — thanh tiến trình hiển thị realtime.

**Bước 4:** Sau khi upload xong, nhấn **Go Back** để quay lại thư mục.

<img width="956" height="395" alt="image" src="https://github.com/user-attachments/assets/0123ef66-4086-4ef1-9254-7d50ff4e6bdb" />

**Cách 2: FTP/SFTP qua FileZilla ( cho project nhiều file)**

```
Host:     hieucute.id.vn  (hoặc 103.159.51.228)
Port:     22 (SFTP) — ưu tiên dùng SFTP
Username: iamhieu
Password: mật khẩu cPanel
```

>  Ưu tiên **SFTP (port 22)** thay FTP (port 21) vì SFTP mã hóa toàn bộ dữ liệu trên đường truyền, không để lộ mật khẩu.

---

### 4.2 Giải nén file

**Qua File Manager:**

**Bước 1:** Upload file `.zip` hoặc `.tar.gz` vào `public_html`.

**Bước 2:** Chuột phải vào file → **Extract** → xác nhận thư mục đích → **Extract File(s)**.

**Qua Terminal (cPanel → Advanced → Terminal):**
```bash
cd ~/public_html
unzip archive.zip
# hoặc
tar -xzvf archive.tar.gz
```

>  **Lỗi hay gặp sau giải nén:** Source code nằm lồng trong `public_html/public_html/` thay vì đúng vị trí. Khắc phục: vào thư mục con → **Select All** → chuột phải → **Move** → đổi đường dẫn về `/home/iamhieu/public_html`.

---

### 4.3 Phân quyền file 

**Qua File Manager:**

Chọn file/thư mục → chuột phải → **Change Permissions** → nhập số quyền → **Change Permissions**.

<img width="935" height="419" alt="image" src="https://github.com/user-attachments/assets/5b3384ae-7d4d-469f-9aaf-b514e891d621" />

**Quyền chuẩn cho WordPress:**

| Đối tượng | Ký hiệu | Số | Lý do |
|-----------|---------|-----|-------|
| Thư mục `public_html` | `rwxr-xr-x` | `755` | Apache cần execute để vào thư mục |
| File `.php`, `.html`, `.css`, `.js` | `rw-r--r--` | `644` | Đọc được, không execute trực tiếp |
| File `wp-config.php` | `rw-------` | `600` | Chứa thông tin DB — chỉ owner đọc được |
| Thư mục `wp-content/uploads` | `rwxrwxr-x` | `775` | WordPress cần ghi file upload |

**Phân quyền hàng loạt qua Terminal:**
```bash
# Đặt 755 cho tất cả thư mục
find ~/public_html -type d -exec chmod 755 {} \;

# Đặt 644 cho tất cả file
find ~/public_html -type f -exec chmod 644 {} \;

# Riêng wp-config.php đặt 600
chmod 600 ~/public_html/wp-config.php
```

>  **Lỗi thường gặp liên quan chmod:**
> - **403 Forbidden** → thư mục đang bị chmod `700` hoặc `777`. Sửa về `755`.
> - **500 Internal Server Error** → file `.php` bị chmod `777` hoặc `000`. Sửa về `644`.
> - **Không upload được ảnh lên WordPress** → `wp-content/uploads` thiếu quyền ghi. Sửa về `775`.

---

## 5. Quản lý Tài khoản FTP

FTP cho phép quản lý file hosting qua client (FileZilla, WinSCP...) mà không cần đăng nhập cPanel. Tất cả gói hosting cPanel hỗ trợ FTP qua port **21** và SFTP qua port **22**.

### 5.1 Tạo tài khoản FTP

**Đường dẫn:** cPanel → **Files** → **FTP Accounts**

**Bước 1:** Điền thông tin trong form phía trên:

| Trường | Giá trị ví dụ | Ghi chú |
|--------|--------------|---------|
| Log In | `ftpdev` | Tên đăng nhập đầy đủ sẽ là `ftpdev@hieucute.id.vn` |
| Password | *(Password Generator)* | Đạt 100/100 điểm |
| Directory | `/home/iamhieu/public_html` | Giới hạn truy cập trong thư mục này |
| Quota | `500 MB` | Hoặc Unlimited |

**Bước 2:** Nhấn **Create FTP Account**.

**Bước 3:** Mở FileZilla → kết nối:
```
Host:     ftp.hieucute.id.vn  (hoặc 103.159.51.228)
Username: ftpdev@hieucute.id.vn
Password: (mật khẩu vừa tạo)
Port:     21 (FTP) hoặc 22 (SFTP — ưu tiên)
```

### 5.2 Quản lý và xóa tài khoản FTP

- cPanel → **FTP Accounts** → danh sách hiển thị bên dưới form tạo.
- **Change Password:** đổi mật khẩu FTP account.
- **Change Quota:** thay đổi giới hạn dung lượng.
- **Delete:** xóa account (chọn có xóa thư mục liên quan hay không).

>  **Bảo mật FTP:**
> - Tài khoản FTP chính (dùng thông tin cPanel) không thể xóa — có full quyền toàn bộ home directory.
> - Khi cần chia sẻ quyền cho bên thứ ba (developer, designer), tạo FTP account riêng với **Directory giới hạn** vào đúng thư mục cần thiết.
> - Không bật **Anonymous FTP** trên môi trường production.

---

## 6. Quản lý Cơ sở dữ liệu MySQL

### 6.1 Tạo Database, User và gán quyền

**Đường dẫn:** cPanel → mục **Databases** (Ảnh 1 & 3) → **Database Wizard**

#### Cách 1: Database Wizard 

**Bước 1 — Tạo Database:**

Nhập tên database, ví dụ `wpdb` → nhấn **Next Step**.

**Bước 2 — Tạo Database User:**

Nhập Username `wpuser` → nhập Password  → nhấn **Create User**.

**Bước 3 — Gán quyền:**

Tích chọn **ALL PRIVILEGES** → nhấn **Next Step**.

**Bước 4 — Lưu lại thông tin kết nối (dùng để điền vào wp-config.php):**

```
DB_NAME:     iamhieu_wpdb
DB_USER:     iamhieu_wpuser
DB_PASSWORD: .%3JN1g5t$KYLY*W
DB_HOST:     localhost
```
<img width="793" height="130" alt="image" src="https://github.com/user-attachments/assets/61fc80d9-f565-4824-a20a-947f515c2b96" />

#### Cách 2: MySQL Databases 

**Đường dẫn:** cPanel → **Databases** → **Manage My Databases** 

1. **Tạo Database:** nhập tên → **Create Database**.
2. **Tạo User:** kéo xuống **MySQL Users** → nhập Username + Password(f8(=6$([(l$!o0)5) → **Create User**. 
3. **Gán quyền:** mục **Add User To Database** → chọn User và Database → **Add** → tích **ALL PRIVILEGES** → **Make Changes**.

<img width="509" height="176" alt="image" src="https://github.com/user-attachments/assets/8b8fba59-3b54-44fe-a3ad-8ad3e7c01285" />

#### Kiểm tra qua phpMyAdmin

cPanel → **Databases** → **phpMyAdmin**  → chọn database ở panel trái để xác nhận đã tạo thành công.

<img width="959" height="224" alt="image" src="https://github.com/user-attachments/assets/dfc6a274-09e0-4ca3-9717-0190716d99c8" />


---

## 7. Bảo mật – Kích hoạt SSL

### 7.1 AutoSSL (Let's Encrypt)

AutoSSL cấp SSL miễn phí và tự động gia hạn, không cần can thiệp thủ công.

**Đường dẫn:** cPanel → mục **Security**  → **SSL/TLS Certificates**

**Bước 1:** cPanel → **Security** → **SSL/TLS Certificates** → tab **Domains**.

**Bước 2:** Danh sách domain hiển thị trạng thái SSL hiện tại.

**Bước 3:** Tick chọn `hieucute.id.vn` và `www.hieucute.id.vn`.

**Bước 4:** Nhấn **Run AutoSSL** → chờ 1–3 phút.

**Bước 5:** Trạng thái chuyển từ  Self-signed sang ✅ Let's Encrypt.

**Bước 6:** Kiểm tra tại `https://hieucute.id.vn` — biểu tượng ổ khóa xanh xuất hiện.

---

### 7.2 Cài SSL có sẵn (từ ZeroSSL, Sectigo...)

**Đường dẫn:** cPanel → **Security** → **SSL/TLS Certificates** → **Manage SSL Certificates** → **Install an SSL Certificate**

**Bước 1:** Chọn domain `hieucute.id.vn`.

**Bước 2:** Paste nội dung 3 file chứng chỉ:

| Ô nhập | File | Nội dung bắt đầu bằng |
|--------|------|-----------------------|
| Certificate (CRT) | `certificate.crt` | `-----BEGIN CERTIFICATE-----` |
| Private Key (KEY) | `server.key` | `-----BEGIN PRIVATE KEY-----` |
| CA Bundle (CABUNDLE) | `ca_bundle.crt` | `-----BEGIN CERTIFICATE-----` |

**Bước 3:** Nhấn **Install Certificate**.

---

### 7.3 Gỡ SSL cũ trước khi chạy lại AutoSSL (khi AutoSSL thất bại)

Khi AutoSSL fail liên tục, nguyên nhân thường là cert cũ còn xung đột. Xóa sạch trước:

**Bước 1:** cPanel → **SSL/TLS Certificates** → **Private Keys** → xóa key cũ của domain.

**Bước 2:** **SSL/TLS Certificates** → **Certificates** → **Uninstall** cert cũ → **Delete Key**.

**Bước 3:** cPanel → **File Manager** → `public_html` → xóa thư mục `.well-known` (nếu có).

**Bước 4:** Quay lại **SSL/TLS Certificates** → tick tất cả domain → **Run AutoSSL**.

**Bước 5:** Chờ 5–10 phút → kiểm tra lại.

---

### 7.4 Force HTTPS Redirect

**Qua giao diện (cPanel 134):**

cPanel → **Domains** → chọn `hieucute.id.vn` → bật toggle **Force HTTPS Redirect** → **Save**.

**Qua `.htaccess` (thủ công):**
```apache
RewriteEngine On
RewriteCond %{HTTPS} !=on
RewriteRule ^ https://%{HTTP_HOST}%{REQUEST_URI} [R=301,L]
```


---

### 7.5 Lỗi thường gặp với SSL

| Lỗi | Nguyên nhân | Khắc phục |
|-----|------------|-----------|
| AutoSSL thất bại | DNS chưa trỏ đúng hoặc port 80 bị block | Kiểm tra DNS, mở port 80 trên firewall |
| `CAA record prevents issuance` | DNS có CAA record không cho Let's Encrypt | Thêm `hieucute.id.vn. CAA 0 issue "letsencrypt.org"` |
| Mixed Content sau HTTPS | Resource load qua `http://` | WP: đổi địa chỉ sang https, cài plugin Really Simple SSL |
| Self-signed cert vẫn hiện | AutoSSL chưa chạy hoặc đang pending | Run AutoSSL, kiểm tra log WHM → AutoSSL → Logs |

---

## 8. Quản lý Email & Chống Spam

### 8.1 Tạo tài khoản Email theo tên miền

**Đường dẫn:** cPanel → mục **Email**  → **Email Accounts**

**Bước 1:** Nhấn **Create (+)** ở góc phải trên.

**Bước 2:** Điền thông tin:

| Trường | Giá trị | Ghi chú |
|--------|---------|---------|
| Username | `info` | Địa chỉ đầy đủ: `info@hieucute.id.vn` |
| Domain | `hieucute.id.vn` | Chọn từ dropdown |
| Password | *(Password Generator)* | Đạt 100/100 điểm |
| Storage Space | `1024 MB` | Hoặc Unlimited nếu gói cho phép |

<img width="926" height="421" alt="image" src="https://github.com/user-attachments/assets/95af89f3-9f88-4409-92c1-6e30067a0c36" />

**Bước 3:** Nhấn **Create**.

**Bước 4:** Truy cập Webmail qua:
- `https://hieucute.id.vn/webmail`
- `https://103.159.51.228:2096`
- Hoặc trong danh sách Email Accounts → nhấn **Check Email**.

>  Khi đăng nhập Webmail (cổng 2096 hoặc /webmail) phải nhập đầy đủ địa chỉ email: `info@hieucute.id.vn`.

<img width="945" height="415" alt="image" src="https://github.com/user-attachments/assets/43ee0675-d939-4778-8a32-a5c32b9f066b" />

---


### 8.2 Cấu hình SPF, DKIM, DMARC

**Đường dẫn:** cPanel → **Email**  → **Email Deliverability**

Trang này hiển thị trạng thái SPF, DKIM, DMARC của từng domain và cho phép tự động sửa bằng nút **Repair**.

#### SPF

Bản ghi SPF chuẩn cho `hieucute.id.vn` (server IP `103.159.51.228`):

| Host | Type | Value |
|------|------|-------|
| `@` | TXT | `v=spf1 ip4:103.159.51.228 ~all` |

> 💡 `~all` = SoftFail. Sau khi ổn định đổi sang `-all` để bảo mật tốt hơn.

#### DKIM

**Bước 1:** Email Deliverability → chọn `hieucute.id.vn` → nhấn **Enable** hoặc **Repair** nếu chưa bật.

**Bước 2:** cPanel tự tạo cặp key và hiển thị bản ghi DNS cần thêm:
```
default._domainkey.hieucute.id.vn   TXT   "v=DKIM1; k=rsa; p=MIIBIjAN..."
```

**Bước 3:** Nếu DNS quản lý bên ngoài cPanel : copy bản ghi → vào DNS Zone → thêm TXT record.

**Kiểm tra:**
```bash
dig TXT default._domainkey.hieucute.id.vn
```
<img width="613" height="301" alt="image" src="https://github.com/user-attachments/assets/b9bc345d-0925-4465-bd05-f76fab337999" />

#### DMARC – Triển khai theo lộ trình 3 giai đoạn

**Thêm vào Zone Editor:** cPanel → **Domains**  → **Zone Editor** → `hieucute.id.vn` → **Manage** → **Add Record** → Type: TXT.

| Giai đoạn | Thời điểm | Host | Value |
|-----------|----------|------|-------|
| **1 – Giám sát** | Mới cài, chưa chắc | `_dmarc` | `v=DMARC1; p=none; rua=mailto:admin@hieucute.id.vn` |
| **2 – Kiểm dịch** | Sau 1–2 tuần xem báo cáo | `_dmarc` | `v=DMARC1; p=quarantine; pct=100; rua=mailto:admin@hieucute.id.vn` |
| **3 – Từ chối** | Khi đã ổn định hoàn toàn | `_dmarc` | `v=DMARC1; p=reject; pct=100; rua=mailto:admin@hieucute.id.vn` |

**Kiểm tra toàn bộ email authentication:**
```bash
dig TXT hieucute.id.vn | grep spf
dig TXT default._domainkey.hieucute.id.vn
dig TXT _dmarc.hieucute.id.vn
```

Kiểm tra online: [https://mxtoolbox.com/emailhealth/](https://mxtoolbox.com/emailhealth/)

---

## 9. Mailing List

### 9.1 Tạo Mailing List

**Đường dẫn:** cPanel → **Email**  → **Mailing Lists**

**Bước 1:** Điền thông tin:

| Trường | Ví dụ | Ghi chú |
|--------|-------|---------|
| List Name | `newsletter` | Địa chỉ: `newsletter@hieucute.id.vn` |
| Domain | `hieucute.id.vn` | |
| Password | *(Password Generator)* | Dùng để đăng nhập giao diện quản lý Mailman |

**Bước 2:** Nhấn **Add Mailing List**.

<img width="796" height="142" alt="image" src="https://github.com/user-attachments/assets/613138d2-8b66-4a5a-a850-90151ca61478" />

---

### 9.2 Giới hạn thành viên gửi vào Mailing List

Mặc định cPanel cho phép **tất cả thành viên** gửi mail vào list. Để chỉ cho phép một số email cụ thể:

**Bước 1:** cPanel → **Mailing Lists** → nhấn **Manage** cạnh list cần cấu hình.

**Bước 2:** Hệ thống chuyển sang giao diện **Mailman**.

**Bước 3:** Vào **Privacy options** → **Sender filters**.

**Bước 4:** Tại mục **"Restrict posting privilege to list members"**:
- Chọn **Yes** → chỉ thành viên trong list mới được gửi.
- Hoặc thêm email cụ thể vào **"List of non-member addresses whose postings should be automatically accepted"**.

**Bước 5:** Nhấn **Submit Your Changes**.

>  Email từ địa chỉ không được liệt kê sẽ bị giữ lại chờ duyệt hoặc từ chối — tùy cấu hình **"Action to take for postings from non-members"**.

<img width="943" height="355" alt="image" src="https://github.com/user-attachments/assets/376c1ebc-e8d9-4614-9981-fb29f1505137" />

---

## 10. Cron Jobs

### 10.1 Cron Job là gì

Cron Job là tác vụ chạy tự động theo lịch trên server — không cần thao tác thủ công. Dùng cho: tự động backup, gửi email định kỳ, dọn dẹp file log, gia hạn SSL...

**Đường dẫn:** cPanel → mục **Advanced**  → **Cron Jobs**

### 10.2 Tạo Cron Job

**Bước 1:** Mục **Add New Cron Job** → điền thời gian chạy:

| Trường | Ý nghĩa | Phạm vi |
|--------|---------|---------|
| Minute | Phút | 0–59 |
| Hour | Giờ | 0–23 |
| Day | Ngày trong tháng | 1–31 |
| Month | Tháng | 1–12 |
| Weekday | Thứ trong tuần | 0–7 (0 và 7 đều là Chủ nhật) |

`*` = mọi giá trị.

**Bước 2:** Nhập lệnh vào ô **Command** — dùng đường dẫn **tuyệt đối**.

**Bước 3:** Nhấn **Add New Cron Job**.

### 10.3 Cú pháp cron phổ biến

| Cú pháp | Ý nghĩa |
|---------|---------|
| `* * * * *` | Mỗi phút |
| `0 * * * *` | Mỗi giờ (phút 0) |
| `0 2 * * *` | Mỗi ngày lúc 2:00 sáng |
| `0 2 * * 0` | Mỗi Chủ nhật lúc 2:00 sáng |
| `0 2 1 * *` | Ngày 1 mỗi tháng lúc 2:00 sáng |
| `*/5 * * * *` | Mỗi 5 phút |

### 10.4 Ví dụ thực tế

**WordPress cron**
```bash
*/15 * * * * wget -q -O /dev/null "https://hieucute.id.vn/wp-cron.php?doing_wp_cron"
```

**Backup database hàng ngày lúc 2:00 sáng:**
```bash
0 2 * * * mysqldump -u iamhieu_wpuser -p'password' iamhieu_wpdb | gzip > /home/iamhieu/backups/db_$(date +\%Y-\%m-\%d).sql.gz
```

**Xóa file backup cũ hơn 7 ngày:**
```bash
0 3 * * * find /home/iamhieu/backups -type f -mtime +7 -delete
```

**Script backup tự động hoàn chỉnh:**
```bash
#!/bin/bash
# Lưu tại /home/iamhieu/backup.sh
# chmod +x /home/iamhieu/backup.sh

BACKUP_DIR="/home/iamhieu/backups/$(date +%Y-%m-%d)"
mkdir -p "$BACKUP_DIR"

# Backup database
mysqldump -u iamhieu_wpuser -p'password' iamhieu_wpdb \
  | gzip -9 > "$BACKUP_DIR/db_hieucute.sql.gz"

# Backup source code
zip -r "$BACKUP_DIR/source_hieucute.zip" /home/iamhieu/public_html/ -q

# Xóa backup cũ hơn 3 ngày
find /home/iamhieu/backups -type d -mtime +3 -exec rm -rf {} + 2>/dev/null

echo "Backup hoàn tất: $BACKUP_DIR"
```

Cron chạy hàng ngày lúc 1:00 sáng:
```
0 1 * * * /home/iamhieu/backup.sh
```

>  **Lưu ý khi dùng Cron Job:**
> - Luôn dùng **đường dẫn tuyệt đối** (ví dụ `/usr/bin/php` thay vì `php`).
> - Để nhận thông báo lỗi qua email: thêm `MAILTO=admin@hieucute.id.vn` ở đầu crontab.
> - Test script thủ công trước khi đặt cron.

---

## 11. Backup & Restore

### 11.1 Tạo Full Backup (cấp User)

**Đường dẫn:** cPanel → **Files**  → **Backup Wizard**

**Bước 1:** Nhấn **Back Up**.

**Bước 2:** Chọn loại backup:

| Loại | Nội dung | Dùng khi |
|------|----------|---------|
| **Full Account Backup** | Toàn bộ: file, DB, email, DNS, cấu hình | Di chuyển hosting, backup tổng thể |
| **Home Directory** | Chỉ file trong `/home/iamhieu/` | Backup code nhanh |
| **MySQL Databases** | Chỉ database | Trước khi cập nhật WordPress/plugin |

**Bước 3:** Chọn **Full Account Backup** → chọn đích lưu:

| Đích | Mô tả |
|------|-------|
| Home Directory | Lưu tại `/home/iamhieu/` trên server |
| Remote FTP Server | Gửi sang FTP server backup |
| Secure Copy (SCP) | Gửi qua SSH an toàn |

**Bước 4:** Điền email nhận thông báo hoàn tất → **Generate Backup**.

**Bước 5:** Tải file `.tar.gz` về máy:

File Manager → tìm file `.tar.gz` trong home directory → chuột phải → **Download**.

---

### 11.2 Backup nhanh từng phần

**Backup database:**
cPanel → **Files** → **Backup** → **Download a MySQL Database Backup** → chọn database → tải về.

**Backup thư mục Home:**
cPanel → **Files** → **Backup** → **Download a Home Directory Backup** → tải về.

---

### 11.3 Restore

#### Restore Source Code

**Bước 1:** cPanel → **Backup Wizard** → **Restore** → **Home Directory** → chọn file `.tar.gz` → **Upload**.

**Bước 2:** Vào **File Manager** sau khi hệ thống giải nén:
- Chuột phải file backup → **Extract** → giải nén ra folder mới.
- Vào folder → `HomeDir` → `public_html` → **Select All** → chuột phải → **Move**.
- Đổi đường dẫn đích về `/home/iamhieu/public_html` → **Move Files**.

#### Restore MySQL Database

**Cách 1:** cPanel → **Backup Wizard** → **Restore** → **MySQL Databases** → chọn file `.sql.gz` → **Upload**.

**Cách 2:** phpMyAdmin → chọn database `iamhieu_wpdb` → tab **Import** → chọn file SQL → **Go**.

>  **Lưu ý quan trọng:**
> - **Restore ghi đè dữ liệu hiện tại** — luôn tạo backup mới nhất trước khi restore bản cũ.
> - Database đích phải **tồn tại sẵn** trước khi import file SQL vào.
> - **Full Account Backup** (`.tar.gz` tổng hợp) chỉ restore được qua **WHM → Restore a Full Backup/cpmove File** — không restore được qua cPanel user thông thường.

---

## 12. Triển khai Next.js trên cPanel

cPanel hỗ trợ Node.js (bao gồm Next.js) thông qua **Setup Node.js App** (Phusion Passenger).

### 12.1 Yêu cầu

- cPanel có tính năng **Setup Node.js App** trong mục **Software**.
- Node.js ≥ 18 cho Next.js 14+.

### 12.2 Tạo Node.js Application

**Đường dẫn:** cPanel → **Software** → **Setup Node.js App** → **Create Application**

| Trường | Giá trị | Ghi chú |
|--------|---------|---------|
| Node.js version | `18.x` hoặc `20.x` | Chọn phiên bản phù hợp Next.js |
| Application mode | `Production` | |
| Application root | `nextjs_app` | Tương đối so với home directory |
| Application URL | `hieucute.id.vn` | Domain chạy ứng dụng |
| Application startup file | `app.js` | File khởi động — tạo ở bước sau |

Nhấn **Create**.

### 12.3 Cài đặt Next.js qua Terminal

**Đường dẫn:** cPanel → **Advanced**  → **Terminal**

```bash
# Vào thư mục ứng dụng
cd ~/nextjs_app

# Xóa file mặc định
rm -rf *

# Tạo project Next.js mới
npx create-next-app@latest . --yes

# Build project
npm run build
```

### 12.4 Tạo file khởi động app.js

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
            console.error('Error:', req.url, err);
            res.statusCode = 500;
            res.end('internal server error');
        }
    }).listen(port, (err) => {
        if (err) throw err;
        console.log(`> Ready on http://${hostname}:${port}`);
    });
});
```

### 12.5 Khởi động ứng dụng

cPanel → **Setup Node.js App** → tìm app `hieucute.id.vn` → **Stop App** → **Start App**.

Truy cập `https://hieucute.id.vn` để kiểm tra.

>  **Lưu ý:**
> - Mỗi lần thay đổi code: Stop → Start lại app.
> - Biến môi trường (API keys, DB connection string...): điền vào ô **Environment variables** trong Setup Node.js App — không hardcode vào file.
> - Next.js dạng Static Export không cần Node.js runtime — build xong copy thư mục `out/` vào `public_html/` là đủ.
> - Nếu không có Terminal: liên hệ Nhân Hòa để bật tính năng.

---

## 13. 5 lỗi phổ biến nhất

### Lỗi 1 – "500 Internal Server Error"

**Nguyên nhân phổ biến:**
- `.htaccess` sai cú pháp
- File `.php` bị chmod sai (777 hoặc 000)
- PHP version không tương thích với code

**Kiểm tra log:**
- cPanel → **Metrics**  → **Errors**
- File Manager → `public_html/error_log`

**Khắc phục:**
```bash
# Đổi tên .htaccess để loại trừ nguyên nhân
mv ~/public_html/.htaccess ~/public_html/.htaccess_bak

# Sửa quyền hàng loạt
find ~/public_html -type f -name "*.php" -exec chmod 644 {} \;
find ~/public_html -type d -exec chmod 755 {} \;
```

cPanel → **Software** → **MultiPHP Manager** → đổi PHP version cho `hieucute.id.vn`.

---

### Lỗi 2 – Email gửi vào Spam hoặc bị từ chối

**Nguyên nhân phổ biến:**
- Thiếu SPF, DKIM, DMARC (xem mục 8.3)
- IP `103.159.51.228` bị blacklist
- Thiếu PTR record (Reverse DNS)

**Kiểm tra:**
- cPanel → **Email** → **Email Deliverability** → xem trạng thái từng record
- cPanel → **Email** → **Track Delivery** → nhập địa chỉ → xem chi tiết

**Khắc phục:**
1. Email Deliverability → nhấn **Repair** để tự sửa SPF/DKIM.
2. Kiểm tra blacklist: [https://mxtoolbox.com/blacklists.aspx](https://mxtoolbox.com/blacklists.aspx)
3. Liên hệ Nhân Hòa tạo PTR record: `103.159.51.228` → `mail.hieucute.id.vn`.

---

### Lỗi 3 – AutoSSL thất bại

**Nguyên nhân phổ biến:**
- DNS chưa trỏ `hieucute.id.vn` về `103.159.51.228`
- Port 80 bị block bởi firewall
- Còn cert cũ xung đột (đặc biệt với self-signed như trong Ảnh 1)

**Kiểm tra:**
```bash
dig A hieucute.id.vn +short        # Phải trả về 103.159.51.228
curl -I http://hieucute.id.vn       # Server phải phản hồi qua port 80
dig CAA hieucute.id.vn              # Kiểm tra CAA record có chặn Let's Encrypt không
```

**Khắc phục:**
- Làm quy trình gỡ cert cũ (mục 7.3) → Run AutoSSL lại.
- Nếu vẫn fail: kiểm tra CSF/iptables có block port 80 không.

---

### Lỗi 4 – WordPress "Error establishing a database connection"

**Nguyên nhân phổ biến:**
- Sai thông tin DB trong `wp-config.php` — thường do **quên prefix `iamhieu_`**.

**Kiểm tra:**
- phpMyAdmin → đăng nhập bằng DB user/password để xác nhận thông tin đúng.

**Khắc phục:**

File Manager → `public_html/wp-config.php` → **Edit**:
```php
define( 'DB_NAME',     'iamhieu_wpdb' );    // Phải có prefix "iamhieu_"
define( 'DB_USER',     'iamhieu_wpuser' );  // Phải có prefix "iamhieu_"
define( 'DB_PASSWORD', 'correct_password' );
define( 'DB_HOST',     'localhost' );
```

Nếu quên mật khẩu DB: cPanel → **Manage My Databases** → **Current Users** → **Change Password**.

---

### Lỗi 5 – Upload file lỗi "413 Request Entity Too Large" hoặc timeout

**Nguyên nhân:** Vượt giới hạn PHP `upload_max_filesize` hoặc `post_max_size`.

**Kiểm tra:** cPanel → **Metrics** → **Errors** → tìm dòng `413`.

**Khắc phục:**

**Cách 1: MultiPHP INI Editor (Ảnh 3 → Software → MultiPHP INI Editor)**

cPanel → **Software** → **MultiPHP INI Editor** → chọn `hieucute.id.vn` → sửa:

| Directive | Giá trị khuyến nghị |
|-----------|---------------------|
| `upload_max_filesize` | `128M` |
| `post_max_size` | `128M` |
| `max_execution_time` | `300` |
| `memory_limit` | `256M` |

Nhấn **Apply**.

**Cách 2: File `.htaccess`:**
```apache
php_value upload_max_filesize 128M
php_value post_max_size 128M
php_value max_execution_time 300
php_value memory_limit 256M
```

---

## 14. Thực chiến: Tạo Package → Account → SSL → WordPress

> **Kịch bản:** Khách hàng "Công ty ABC" mua gói Hosting Doanh Nghiệp, domain `abc-company.vn`. Server IP `103.159.51.228`.

### 14.1 Tạo Package trong WHM

WHM → **Packages** → **Add a Package** → điền theo bảng ở mục 2.1 → **Add**.

Package `doanhnghiep_package` sẵn sàng cho mọi khách cùng gói.

### 14.2 Tạo Account cho khách

WHM → **Account Functions** → **Create a New Account**

| Trường | Giá trị | Quy tắc |
|--------|---------|---------|
| Domain | `abc-company.vn` | |
| Username | `abccompany` | Không phải `root`, không dấu, viết liền |
| Password | *(Password Generator — 100/100)* | Copy lưu lại |
| Email | Email thật của khách | |
| Package | `doanhnghiep_package` | |
| Mail Routing | **Automatically Detect Configuration** | |
| Enable DKIM | ✅ | |
| Enable SPF | ✅ | |
| Enable DMARC | ✅ | |

Nhấn **Create** → kiểm tra log không có dòng đỏ → nhấn **Go to cPanel**.

### 14.3 Xác minh DNS trước khi cài SSL (bắt buộc)

```bash
dig A abc-company.vn +short
```
Phải trả về `103.159.51.228`. Nếu chưa — **dừng lại, chờ DNS propagate**.

### 14.4 Cài SSL

Đăng nhập cPanel của account vừa tạo.

cPanel → **Security** → **SSL/TLS Certificates** → tick domain → **Run AutoSSL** → chờ ✅.

Sau đó: cPanel → **Domains** → chọn domain → bật **Force HTTPS Redirect**.

>  Thứ tự: **SSL trước → Force HTTPS sau**.

### 14.5 Cài WordPress

cPanel → **WordPress Management** → **Install WordPress** → điền thông tin (username không phải `admin`) → **Install**.

Kiểm tra `https://abc-company.vn/wp-admin`.

### 14.6 Checklist bàn giao khách

| Hạng mục | Trạng thái |
|----------|-----------|
| DNS trỏ đúng IP | ☐ |
| SSL AutoSSL thành công (ổ khóa xanh) | ☐ |
| Force HTTPS Redirect đã bật | ☐ |
| DKIM / SPF / DMARC đã bật | ☐ |
| WordPress cài xong, đăng nhập `/wp-admin` được | ☐ |
| Đổi permalink WP sang "Post name" | ☐ |
| Xóa theme / plugin mặc định không dùng | ☐ |
| Kiểm tra Mixed Content | ☐ |
| Gửi thông tin cPanel + WP admin cho khách | ☐ |
| Đặt lịch backup tự động (Cron Job) | ☐ |

---

## 15. Mapping kiến thức DirectAdmin sang cPanel/WHM

### 15.1 Hierarchy (phân cấp quản trị)

| DirectAdmin | cPanel/WHM | Điểm khác biệt |
|-------------|-----------|----------------|
| Admin (toàn quyền server) | Root trong **WHM** | Tương đương |
| Reseller (quản lý nhiều User) | Reseller account trong **WHM** | WHM: Reseller dùng chung giao diện WHM (ẩn bớt menu). DA: Reseller có UI riêng biệt |
| User (chủ 1 website) | **cPanel** account | Tương đương 1-1 |
| User Package | **Package** trong WHM | Tương đương 1-1 |
| Giao diện Admin (port 2222) | **WHM** (port 2087) | DA dùng 1 port cho tất cả; cPanel/WHM **tách port vật lý** |
| Giao diện User (port 2222) | **cPanel** (port 2083) | |

> 🔑 **Khác biệt quan trọng nhất:** DirectAdmin dùng **1 cổng (2222)** cho cả Admin/Reseller/User. cPanel/WHM **tách hoàn toàn theo cổng**: 2087 cho quản trị, 2083 cho khách hàng, 2096 cho webmail.

### 15.2 Mapping thao tác thường dùng

| Việc cần làm | DirectAdmin | cPanel/WHM |
|--------------|-------------|------------|
| Tạo gói hosting | Admin → **Package Manager** | WHM → **Packages → Add a Package** |
| Tạo tài khoản khách | Admin → **Create User** | WHM → **Create a New Account** |
| Sửa giới hạn tài nguyên | Admin → **User Manager → Edit** | WHM → **Modify an Account** |
| Khóa tài khoản | Admin → **Suspend/Unsuspend Users** | WHM → **Suspend/Unsuspend an Account** |
| Xóa tài khoản | Admin → **Delete Users** | WHM → **Terminate a Multi-User Account** |
| Cài SSL 1 domain | User → **SSL Certificates** | cPanel → **SSL/TLS Certificates** |
| Cài WordPress | User → Softaculous/Installatron | cPanel → **WordPress Management** |
| Quản lý DNS 1 domain | User → **DNS Management** | cPanel → **Zone Editor** |
| Quản lý DNS toàn server | Admin → **Admin Level DNS** | WHM → **DNS Functions → Edit DNS Zone** |
| Backup toàn tài khoản | User → **Backup/Restore** | cPanel → **Backup Wizard** |
| Restore Full Backup | Admin → **Admin Backup/Transfer** | WHM → **Restore a Full Backup/cpmove File** |
| Restart dịch vụ hệ thống | Admin → **Service Monitor** | WHM → **Restart Services** |
| Quản lý Firewall | Admin → CSF | WHM → CSF (giống hệt) |

### 15.3 Kiến thức dùng lại được 100%

| Kiến thức | Áp dụng lại |
|-----------|------------|
| DNS record types (A, CNAME, MX, TXT) | Giống hệt — chỉ khác giao diện nhập |
| SPF / DKIM / DMARC nguyên lý | Giống hệt |
| Let's Encrypt / AutoSSL nguyên lý | Giống hệt |
| CSF Firewall cấu hình | Giống hệt |
| Cron job syntax (`* * * * *`) | Giống hệt |
| chmod 755/644/600 | Giống hệt |

### 15.4 Cần học lại thật sự

| Chủ đề | Lý do |
|--------|-------|
| Cấu trúc file Apache VirtualHost | DA dùng `/etc/httpd/conf/extra/httpd-vhosts.conf`; cPanel dùng `/etc/apache2/conf.d/userdata/` với hệ thống "userdata" riêng |
| Feature List trong WHM | Khái niệm này không có trong DA — cho phép bật/tắt cả cụm tính năng UI cho từng Package |
| WP Toolkit / WordPress Management | Giao diện và luồng thao tác khác Installatron/Softaculous |

---

## Bảng vị trí log quan trọng

| Log | Vị trí trong cPanel 134 |
|-----|------------------------|
| Error log website | **Metrics → Errors** hoặc `~/public_html/error_log` |
| Access log | **Metrics → Raw Access** |
| Email gửi/nhận | **Email → Track Delivery** |
| Trạng thái SSL | **Security → SSL/TLS Certificates** |
| Tài nguyên server | **Metrics → Bandwidth / Visitors** |
| Apache error (SSH) | `/var/log/apache2/error_log` |
| MySQL error (SSH) | `/var/lib/mysql/error.log` |
| cPanel system log (SSH) | `/usr/local/cpanel/logs/` |
| WHM AutoSSL log | WHM → **SSL/TLS → Manage AutoSSL → Logs** |

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
