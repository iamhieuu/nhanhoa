# Triển khai ứng dụng mail server MDaemon

## Mục lục

- [1. Tổng quan về MDaemon](#1-tổng-quan-về-mdaemon)
- [2. Cài đặt email server MDaemon trên Windows Server 2022](#2-cài-đặt-email-server-mdaemon-trên-windows-server-2022)
  - [2.1. Bước 1: Cài DNS các bản ghi A, MX, PTR](#21-bước-1-cài-dns-các-bản-ghi-a-mx-ptr)
  - [2.2. Bước 2: Chuẩn bị Windows Server](#22-bước-2-chuẩn-bị-windows-server)
    - [Cài IP tĩnh cho Windows Server](#cài-ip-tĩnh-cho-windows-server)
    - [Đổi hostname](#đổi-hostname)
    - [Tắt Firewall tạm thời để test](#tắt-firewall-tạm-thời-để-test)
    - [Gỡ IIS tránh xung đột port 80/443](#gỡ-iis-tránh-xung-đột-port-80443)
  - [2.3. Bước 3: Tải và cài MDaemon](#23-bước-3-tải-và-cài-mdaemon)
- [3. Truy cập Admin và End-user](#3-truy-cập-admin-và-end-user)
- [4. Các port cần thiết trên email server MDaemon](#4-các-port-cần-thiết-trên-email-server-mdaemon)
- [5. Khởi tạo domain, user, group, alias, mailing lists](#5-khởi-tạo-domain-user-group-alias-mailing-lists)
- [6. Thiết lập chính sách mật khẩu account email](#6-thiết-lập-chính-sách-mật-khẩu-account-email)
- [7. Thiết lập chữ ký email](#7-thiết-lập-chữ-ký-email)
  - [7.1. Cho từng cá nhân](#71-cho-từng-cá-nhân)
  - [7.2. Cho cả domain](#72-cho-cả-domain)
- [8. Thiết lập forward email](#8-thiết-lập-forward-email)
- [9. Tìm hiểu về Content Filter](#9-tìm-hiểu-về-content-filter)
  - [9.1. Spam Filter](#91-spam-filter)
  - [9.2. Antivirus](#92-antivirus)
  - [9.3. Attachment Filters](#93-attachment-filters)
  - [9.4. Message Filters](#94-message-filters)
- [10. Phân quyền tài khoản thành Admin của domain](#10-phân-quyền-tài-khoản-thành-admin-của-domain)
- [11. Đổi mật khẩu account Admin Global, Admin domain](#11-đổi-mật-khẩu-account-admin-global-admin-domain)
- [12. Kiểm tra log gửi/nhận email](#12-kiểm-tra-log-gửinhận-email)
  - [12.1. Cách 1: Xem qua giao diện Logs](#121-cách-1-xem-qua-giao-diện-logs)
  - [12.2. Cách 2: Xem trạng thái hàng chờ](#122-cách-2-xem-trạng-thái-hàng-chờ)
- [13. Dynamic Screening trong Security của MDaemon](#13-dynamic-screening-trong-security-của-mdaemon)
- [14. Backup và Restore](#14-backup-và-restore)
  - [14.1. Tạo file thực thi (.bat)](#141-tạo-file-thực-thi-bat)
  - [14.2. Thiết lập Task Scheduler](#142-thiết-lập-task-scheduler)
  - [14.3. Cấu hình nâng cao (qua GUI Task Scheduler)](#143-cấu-hình-nâng-cao-qua-gui-task-scheduler)

---

## 1. Tổng quan về MDaemon

**MDaemon** là một phần mềm mail server (SMTP/POP3/IMAP server) chạy trên nền **Windows Server**, phổ biến với doanh nghiệp vừa và nhỏ vì cài đặt và quản trị dễ dàng qua giao diện web, hỗ trợ đầy đủ các tính năng: chống spam, antivirus, webmail (WorldClient), quản lý domain/user/group, và các chính sách bảo mật nâng cao.

So với Zimbra (chạy trên Linux), MDaemon phù hợp với các hạ tầng đã sẵn dùng Windows Server / Active Directory.

---

## 2. Cài đặt email server MDaemon trên Windows Server 2022

### 2.1. Bước 1: Cài DNS các bản ghi A, MX, PTR

**Định nghĩa các loại bản ghi DNS liên quan đến mail server:**

| Loại bản ghi | Ý nghĩa |
|---|---|
| **A** | Trỏ tên miền/hostname (VD: `mail.domain.com`) sang một địa chỉ IPv4 cụ thể. |
| **MX** (Mail Exchanger) | Khai báo server nào chịu trách nhiệm nhận mail cho một domain. Có độ ưu tiên (priority) — số càng nhỏ càng được ưu tiên gửi đến trước. |
| **PTR** (Pointer / Reverse DNS) | Bản ghi ngược, ánh xạ từ IP về lại hostname. Rất quan trọng với mail server — nhiều hệ thống nhận mail (Gmail, Outlook...) sẽ từ chối hoặc đánh spam nếu server gửi đi không có PTR hợp lệ khớp với hostname khai báo trong SMTP banner. |

<img width="465" height="215" alt="image" src="https://github.com/user-attachments/assets/7fe9005e-35cf-4563-ab2a-6919091fe42a" />

> Kiểm tra bản ghi DNS bằng PowerShell:
> ```powershell
> Resolve-DnsName mail.domain.com -Type A
> Resolve-DnsName domain.com -Type MX
> Resolve-DnsName 103.159.51.228 -Type PTR
> ```

---

### 2.2. Bước 2: Chuẩn bị Windows Server

#### Cài IP tĩnh cho Windows Server

<img width="396" height="383" alt="image" src="https://github.com/user-attachments/assets/9aaad67b-f4cb-4d70-a2df-841110a9f826" />

Ngoài thao tác qua GUI (Network Connections), có thể cấu hình IP tĩnh bằng PowerShell:

```powershell
New-NetIPAddress -InterfaceAlias "Ethernet0" -IPAddress 192.168.254.10 -PrefixLength 24 -DefaultGateway 192.168.254.1
Set-DnsClientServerAddress -InterfaceAlias "Ethernet0" -ServerAddresses 8.8.8.8,1.1.1.1
```

**Giải thích:**
- `-InterfaceAlias`: tên card mạng (xem bằng lệnh `Get-NetAdapter`).
- `-PrefixLength 24`: tương đương subnet mask `255.255.255.0`.
- `-DefaultGateway`: địa chỉ gateway ra internet.
- `Set-DnsClientServerAddress`: gán DNS server cho card mạng đó.

#### Đổi hostname

```powershell
Rename-Computer -NewName "MAILSERVER" -Restart
```

<img width="190" height="121" alt="image" src="https://github.com/user-attachments/assets/2fc5e87c-a463-4560-8e5b-f50a705b2244" />

**Giải thích flag:**
- `-NewName`: tên hostname mới muốn đặt.
- `-Restart`: tự động khởi động lại máy ngay sau khi đổi tên (bắt buộc để hostname có hiệu lực).

#### Tắt Firewall tạm thời để test

<img width="601" height="368" alt="image" src="https://github.com/user-attachments/assets/906cc6fa-e40c-4a33-8c82-03f0ae771c26" />

Tương đương bằng PowerShell (chỉ nên dùng trong môi trường lab, **không khuyến khích ở production**):

```powershell
Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled False
```

> **Khuyến nghị production:** thay vì tắt hẳn firewall, chỉ nên mở đúng các port MDaemon cần dùng (xem bảng port ở mục 4):
> ```powershell
> New-NetFirewallRule -DisplayName "MDaemon SMTP" -Direction Inbound -Protocol TCP -LocalPort 25 -Action Allow
> New-NetFirewallRule -DisplayName "MDaemon IMAP" -Direction Inbound -Protocol TCP -LocalPort 143 -Action Allow
> New-NetFirewallRule -DisplayName "MDaemon WorldClient" -Direction Inbound -Protocol TCP -LocalPort 3000 -Action Allow
> ```

#### Gỡ IIS tránh xung đột port 80/443

```powershell
Disable-WindowsOptionalFeature -Online -FeatureName IIS-WebServerRole
```

**Giải thích:**
- `-Online`: áp dụng thay đổi ngay trên hệ điều hành đang chạy (không phải trên file image offline).
- `-FeatureName`: tên feature cần tắt — `IIS-WebServerRole` là vai trò Web Server của IIS, cần gỡ vì MDaemon WorldClient (port 443/HTTPS) có thể xung đột nếu IIS cũng đang lắng nghe các port đó.

> Kiểm tra port đang bị chiếm dụng trước khi gỡ:
> ```powershell
> Get-NetTCPConnection -LocalPort 80,443 | Select-Object LocalAddress,LocalPort,OwningProcess
> ```

---

### 2.3. Bước 3: Tải và cài MDaemon

* Truy cập trang [mdaemon.com](https://mdaemon.com/pages/downloads-mdaemon-mail-server-free-trial) tải bản mới nhất (.exe).

<img width="642" height="374" alt="image" src="https://github.com/user-attachments/assets/2822eeeb-6f56-4dfe-a32e-9748360b31a0" />

* Chạy file cài đặt dưới quyền Admin (**Run as administrator**).

<img width="441" height="260" alt="image" src="https://github.com/user-attachments/assets/5f1016fb-0f55-4202-a1f6-f34c84d31c2b" />
<img width="346" height="226" alt="image" src="https://github.com/user-attachments/assets/4cb0e190-5d55-4673-919a-4882b3c40fdf" />

* Cấu hình domain và hostname.

<img width="346" height="230" alt="image" src="https://github.com/user-attachments/assets/60e59b6a-265f-4751-b02b-53cd0070ff1a" />

* Tạo tài khoản Admin đầu tiên.

<img width="341" height="225" alt="image" src="https://github.com/user-attachments/assets/959b63f2-b452-4fdf-bf8c-393087605ccd" />

Cài đặt thành công:

<img width="641" height="374" alt="image" src="https://github.com/user-attachments/assets/975c9679-6eca-4ecb-ab2e-d169d30a6e34" />

> Kiểm tra service MDaemon đã chạy bằng PowerShell:
> ```powershell
> Get-Service -Name "MDaemon*"
> Restart-Service -Name "MDaemon" -Force   # nếu cần khởi động lại dịch vụ
> ```

---

## 3. Truy cập Admin và End-user

Trong MDaemon, có 3 cổng truy cập chính qua trình duyệt:

| Cổng truy cập | Đối tượng | Link mặc định | Định nghĩa |
|---|---|---|---|
| **WebAdmin** | Quản trị viên | `http://IP-Server:1000` | Giao diện quản trị hệ thống, cấu hình toàn bộ server (domain, user, filter, bảo mật...). |
| **WorldClient** | Người dùng cuối | `http://IP-Server:3000` | Giao diện Webmail để đọc/gửi thư, tương tự Outlook Web Access. |
| **Remote Administration** | Domain Admin | Tương tự WebAdmin | Chức năng gần giống WebAdmin nhưng giới hạn quyền, chỉ dành cho user được cấp quyền quản lý riêng một domain nhỏ (không đụng được cấu hình toàn server). |

---

## 4. Các port cần thiết trên email server MDaemon

| Dịch vụ | Port | Ghi chú |
| :--- | :---: | :--- |
| **SMTP** | 25 | Gửi/Nhận mail giữa các server |
| **POP3** | 110 | Tải mail về máy khách |
| **IMAP** | 143 | Đồng bộ mail giữa server và thiết bị |
| **WorldClient (HTTP)** | 3000 | Giao diện Webmail dành cho người dùng cuối |
| **WebAdmin (HTTP)** | 1000 | Giao diện quản trị hệ thống qua trình duyệt |
| **SMTP SSL/TLS** | 465 / 587 | Gửi mail bảo mật |
| **IMAP SSL** | 993 | Đồng bộ mail bảo mật qua giao thức IMAP |
| **POP3 SSL** | 995 | Tải mail bảo mật qua giao thức POP3 |
| **WorldClient (HTTPS)** | 443 | Truy cập Webmail qua kết nối bảo mật SSL |

> Test nhanh một port có đang mở/lắng nghe trên server bằng PowerShell:
> ```powershell
> Test-NetConnection -ComputerName localhost -Port 25
> ```

---

## 5. Khởi tạo domain, user, group, alias, mailing lists

**Định nghĩa nhanh:**
- **Domain**: tên miền mail server sẽ quản lý (VD: `domain.com`) — một MDaemon server có thể quản lý nhiều domain cùng lúc (multi-domain).
- **User**: tài khoản email của từng cá nhân.
- **Group**: tập hợp nhiều user, dùng để phân quyền hoặc gửi mail hàng loạt theo nhóm.
- **Alias**: một địa chỉ email "bí danh" trỏ về một hộp thư (mailbox) có sẵn — không tốn thêm dung lượng lưu trữ riêng.
- **Mailing List**: danh sách gửi mail hàng loạt — khi gửi đến 1 địa chỉ list, mail được nhân bản gửi tới tất cả thành viên đăng ký.

* Domain

<img width="497" height="319" alt="image" src="https://github.com/user-attachments/assets/206ccffc-3258-48f1-977d-74431c7ba16f" />

* User

<img width="429" height="317" alt="image" src="https://github.com/user-attachments/assets/16996c3c-809e-4fc3-a6f0-ab916985a360" />

* Group

<img width="490" height="323" alt="image" src="https://github.com/user-attachments/assets/4d0d1702-3b6b-41ae-9573-94e30bd52560" />

* Alias

<img width="634" height="374" alt="image" src="https://github.com/user-attachments/assets/c175fb18-0a4a-4b8b-bae8-d96ad0e18dde" />

* Mailing Lists

<img width="401" height="316" alt="image" src="https://github.com/user-attachments/assets/02347358-dc46-41fa-95a5-59dda5ffd59c" />

---

## 6. Thiết lập chính sách mật khẩu account email

* Vào **Setup → Account Settings → Passwords**.

<img width="845" height="414" alt="image" src="https://github.com/user-attachments/assets/e5ef51a6-7fe5-4cb6-b6a6-8545d41b9be7" />

Chính sách sẽ thiết lập độ mạnh của mật khẩu, hạn sử dụng (password expiry), và cảnh báo gửi đến người dùng trước khi hết hạn.

---

## 7. Thiết lập chữ ký email

### 7.1. Cho từng cá nhân

* Vào **Account Manager → Edit người dùng → Signature**.

<img width="580" height="336" alt="image" src="https://github.com/user-attachments/assets/60fe698c-66ac-463d-83bc-b66960d8913e" />

### 7.2. Cho cả domain

* Vào **Domain → chọn domain cần thiết lập → Default Signature**.

<img width="761" height="367" alt="image" src="https://github.com/user-attachments/assets/a9937930-97eb-43e2-86d1-48e1e4ac33db" />

---

## 8. Thiết lập forward email

* Vào **Account Manager → Edit người dùng → Forwarding**.

<img width="593" height="337" alt="image" src="https://github.com/user-attachments/assets/c24360d9-793f-45d2-8686-bf280d99a572" />

Nếu muốn giữ lại một bản sao tại hộp thư gốc, tích chọn ô **Retain a copy of forwarded mail**.

---

## 9. Tìm hiểu về Content Filter

**Content Filter** là tập hợp các bộ lọc kiểm tra nội dung/tệp đính kèm của email trước khi cho vào hộp thư người dùng, gồm 4 thành phần chính:

### 9.1. Spam Filter

Là bộ phận đánh giá xem một email có phải là quảng cáo hoặc lừa đảo (spam/phishing) hay không, thường dựa trên điểm số (spam score) tính từ nội dung, tiêu đề, IP nguồn gửi...

<img width="697" height="339" alt="image" src="https://github.com/user-attachments/assets/aef57d97-6d88-420b-8f50-ae9888e1b17a" />

### 9.2. Antivirus

Là tấm khiên bảo vệ server khỏi các phần mềm độc hại (Malware/Virus) đính kèm trong thư.

<img width="771" height="341" alt="image" src="https://github.com/user-attachments/assets/f7f11d20-2fb8-4eb1-bb74-24c4c8610340" />

Nếu phát hiện virus, hệ thống sẽ tự động xóa file đính kèm, xóa toàn bộ thư, hoặc cô lập (**Quarantine**) để Admin kiểm tra.

### 9.3. Attachment Filters

Là bộ lọc dựa trên loại tệp (đuôi file) thay vì nội dung file, để ngăn người dùng vô tình tải và chạy các file thực thi có thể chứa mã độc — ngay cả khi virus đó "mới" đến mức Antivirus chưa kịp nhận diện.

<img width="818" height="344" alt="image" src="https://github.com/user-attachments/assets/5cb6e514-b515-47ff-ac20-ce2def8e9b73" />

### 9.4. Message Filters

Là bộ lọc linh hoạt nhất, cho phép Admin tự định nghĩa quy tắc theo nhu cầu doanh nghiệp.

Ví dụ: nếu trong nội dung thư có chữ "Lương tháng 13" thì tự động chuyển thư đó vào hộp thư của Sếp.

<img width="806" height="365" alt="image" src="https://github.com/user-attachments/assets/1c724917-92d9-4a74-a25a-358256148af7" />

---

## 10. Phân quyền tài khoản thành Admin của domain

**Domain Admin** là tài khoản chỉ có quyền quản lý các user trong một tên miền cụ thể, không được can thiệp vào cấu hình hệ thống server (khác với Global Admin ở mục 11).

* Vào **Account Manager → Administrative Roles → Domain Administrator**.

<img width="784" height="342" alt="image" src="https://github.com/user-attachments/assets/009bb0bc-f6ae-4ee6-8fdd-b8e87a1228c4" />

---

## 11. Đổi mật khẩu account Admin Global, Admin domain

**Global Admin** là tài khoản có quyền chỉnh sửa mọi thứ trên toàn server (tất cả domain, toàn bộ cấu hình hệ thống).

* Vào **Account Manager → Edit → Account Detail → sửa Password**.

<img width="569" height="353" alt="image" src="https://github.com/user-attachments/assets/65c453e7-6814-4bc6-86c0-845c1bdf57b2" />

* Admin domain: là tập con quyền hạn của Global Admin (đã giải thích ở mục 10).

---

## 12. Kiểm tra log gửi/nhận email

### 12.1. Cách 1: Xem qua giao diện Logs

<img width="822" height="368" alt="image" src="https://github.com/user-attachments/assets/2a835a40-7a01-4640-b7bb-eaec6d678963" />

* **SMTP-In**: ghi lại toàn bộ quá trình các server khác gửi mail đến server của bạn (nhận mail).
* **SMTP-Out**: ghi lại quá trình server của bạn gửi mail đi nơi khác (gửi mail).

<img width="811" height="343" alt="image" src="https://github.com/user-attachments/assets/93445e0a-197a-4c04-af7b-00d7d1259ed0" />
<img width="817" height="327" alt="image" src="https://github.com/user-attachments/assets/ba1783f8-935e-4d9a-bb32-3adaccca100a" />

Các dòng chữ màu đen/xanh là bình thường, dòng chữ màu đỏ thường là báo lỗi.

### 12.2. Cách 2: Xem trạng thái hàng chờ

* Vào **Messages and Queues**:
  * **Remote Queue**: nơi chứa các email đang chờ để gửi ra ngoài internet.
  * **Local Queue**: nơi chứa các email gửi nội bộ giữa các user trong công ty bạn.

<img width="817" height="370" alt="image" src="https://github.com/user-attachments/assets/400567ab-d216-4347-827a-0f4eec2df0e7" />

Nếu thấy email nằm ở đây quá lâu, có thể chuột phải chọn **Freeze** (tạm giữ) hoặc **Re-queue** (gửi lại).

**Một số mã lỗi log "đặc biệt chú ý" trong MDaemon:**

| Mã lỗi | Ý nghĩa |
|---|---|
| `250 OK` | Email đã được gửi/nhận thành công. |
| `550 User unknown` | Gửi trượt vì địa chỉ email người nhận không tồn tại. |
| `421 Service unavailable` | Server đối phương đang bận hoặc từ chối kết nối. |
| `554 Transaction failed` | Thư bị chặn do dính Content Filter hoặc Spam Filter. |

---

## 13. Dynamic Screening trong Security của MDaemon

**Dynamic Screening** là một trong những tính năng bảo mật "thông minh" nhất của MDaemon. Đây là bộ lọc theo dõi các kết nối đến server theo thời gian thực — nếu một địa chỉ IP có hành vi "xấu" (thử mật khẩu sai nhiều lần, gửi quá nhiều mail rác...), Dynamic Screening sẽ tự động chặn IP đó trong một khoảng thời gian nhất định.

<img width="826" height="342" alt="image" src="https://github.com/user-attachments/assets/020a731d-7994-401a-af64-02b48b643f7b" />

| Tính năng | Mô tả |
|---|---|
| **Authentication Failure Blocking** | Chặn nếu 1 IP đăng nhập sai mật khẩu **X** lần trong **Y** phút. |
| **Account Blocking Options** | Tự động "đóng băng" một tài khoản nếu có quá nhiều lần đăng nhập thất bại. |
| **Dynamic Allow/Block List** | Danh sách các IP tin tưởng / không tin tưởng. |

---

## 14. Backup và Restore

### 14.1. Tạo file thực thi (.bat)

<img width="362" height="161" alt="image" src="https://github.com/user-attachments/assets/e1839d24-2f8c-46f1-b7a1-54df4eab52b1" />

Ví dụ nội dung file backup đơn giản dùng `robocopy` (công cụ copy mạnh mẽ, có thể copy incremental, giữ nguyên permission, retry khi lỗi):

```batch
@echo off
set BACKUP_DIR=D:\Backup\MDaemon\%date:~-4%%date:~3,2%%date:~0,2%
robocopy "C:\MDaemon" "%BACKUP_DIR%" /MIR /R:3 /W:5 /LOG:"%BACKUP_DIR%\backup.log"
```

**Giải thích flag của `robocopy`:**

| Flag | Ý nghĩa |
|---|---|
| `/MIR` | Mirror — đồng bộ chính xác thư mục nguồn sang đích (xóa ở đích những gì không còn ở nguồn). |
| `/R:3` | Số lần thử lại (retry) nếu copy 1 file bị lỗi — ở đây là 3 lần. |
| `/W:5` | Thời gian chờ (giây) giữa mỗi lần retry — ở đây là 5 giây. |
| `/LOG:` | Ghi log quá trình backup ra file, thay vì chỉ in ra màn hình. |

### 14.2. Thiết lập Task Scheduler

<img width="492" height="351" alt="image" src="https://github.com/user-attachments/assets/47af66f4-5b2e-496a-9ef2-88ee86351012" />

Tương đương tạo Task bằng PowerShell/CLI (thay vì thao tác GUI):

```powershell
$action = New-ScheduledTaskAction -Execute "C:\Scripts\mdaemon_backup.bat"
$trigger = New-ScheduledTaskTrigger -Daily -At 2:00AM
Register-ScheduledTask -TaskName "MDaemon Backup" -Action $action -Trigger $trigger -RunLevel Highest -User "SYSTEM"
```

**Giải thích:**
- `New-ScheduledTaskAction -Execute`: chỉ định file/script sẽ được chạy.
- `New-ScheduledTaskTrigger -Daily -At 2:00AM`: chạy hàng ngày lúc 2h sáng.
- `-RunLevel Highest`: tương đương tick **Run with highest privileges** trên GUI — chạy với quyền cao nhất (Admin).
- `-User "SYSTEM"`: chạy bằng tài khoản hệ thống, không phụ thuộc user nào đang đăng nhập — tương đương **Run whether user is logged on or not**.

### 14.3. Cấu hình nâng cao (qua GUI Task Scheduler)

* Chọn **Run whether user is logged on or not** — để task tự chạy ngay cả khi bạn đã log out khỏi server.
* Tích chọn **Run with highest privileges** — chạy với quyền Admin cao nhất để có quyền copy file hệ thống.

<img width="386" height="266" alt="image" src="https://github.com/user-attachments/assets/b119748b-f6d2-4ad9-96b8-01ee88431e46" />

Tại tab **Conditions**:
* Bỏ tích mục **Start the task only if the computer is on AC power** (nếu là máy ảo thì nên bỏ tích để đảm bảo task luôn chạy đúng giờ).

<img width="386" height="295" alt="image" src="https://github.com/user-attachments/assets/03236581-14f0-43cf-9df3-49b27812df17" />

> Kiểm tra / chạy thử task ngay bằng PowerShell:
> ```powershell
> Get-ScheduledTask -TaskName "MDaemon Backup"
> Start-ScheduledTask -TaskName "MDaemon Backup"
> ```
