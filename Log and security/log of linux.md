# Log và Security cơ bản trên Linux & Windows

---

## Mục lục

- [Giới thiệu](#gioi-thieu)
- [Phần I. Log và Security trên Linux](#phan-i-log-va-security-tren-linux)
  - [1. Các loại Logs quan trọng trên Linux](#1-cac-loai-logs-quan-trong)
    - [1.1. System Logs](#11-system-logs)
    - [1.2. Application Logs](#12-application-logs)
    - [1.3. Non-Human-Readable Logs](#13-non-human-readable-logs)
    - [1.4. Kiểm tra log đăng nhập, reboot/shutdown](#14-kiem-tra-log-dang-nhap)
  - [2. Công cụ quản lý Logs](#2-cong-cu-quan-ly-logs)
    - [2.1. journalctl](#21-journalctl)
    - [2.2. logrotate](#22-logrotate)
    - [2.3. syslog](#23-syslog)
    - [2.4. rsyslog](#24-rsyslog)
    - [2.5. syslog-ng](#25-syslog-ng)
    - [2.6. auditd — Audit Framework](#26-auditd)
  - [3. Security cơ bản trên Linux](#3-security-co-ban-tren-linux)
    - [3.1. Bảo mật đăng nhập](#31-bao-mat-dang-nhap)
    - [3.2. Quản lý user và quyền](#32-quan-ly-user-va-quyen)
    - [3.3. Firewall](#33-firewall)
    - [3.4. Cập nhật hệ thống](#34-cap-nhat-he-thong)
    - [3.5. Giám sát hệ thống](#35-giam-sat-he-thong)
    - [3.6. Kiểm tra tính toàn vẹn log với AIDE](#36-aide)
  - [4. Phân tích logs để phát hiện tấn công (Linux)](#4-phan-tich-logs-linux)
    - [4.1. Tổng quan](#41-tong-quan)
    - [4.2. Phương pháp và dấu hiệu tấn công](#42-phuong-phap)
    - [4.3. Công cụ dòng lệnh hỗ trợ phân tích log](#43-cong-cu-cli)
    - [4.4. Công cụ giám sát & quản lý log mã nguồn mở](#44-cong-cu-giam-sat)
    - [4.5. Ví dụ thực chiến phân tích log](#45-vi-du-thuc-chien)
  - [5. Tự động hoá giám sát & cảnh báo](#5-tu-dong-hoa)
  - [6. Xử lý sự cố logging thường gặp](#6-xu-ly-su-co)
---

<a id="gioi-thieu"></a>
## Giới thiệu

Log (nhật ký hệ thống) là bản ghi các sự kiện xảy ra trên một hệ thống — từ việc người dùng đăng nhập, một dịch vụ khởi động/dừng, cho đến một request HTTP hay một lệnh `sudo` được thực thi. Log đóng ba vai trò chính:

1. **Vận hành (Operations):** giúp quản trị viên khắc phục sự cố, theo dõi hiệu năng hệ thống.
2. **Bảo mật (Security):** là nguồn bằng chứng gần như duy nhất để phát hiện, điều tra và ứng phó khi có sự cố tấn công.
3. **Tuân thủ (Compliance):** nhiều chuẩn như PCI DSS, ISO 27001, SOC 2 yêu cầu lưu trữ và giám sát log trong một khoảng thời gian nhất định.

Tài liệu này trình bày lần lượt kiến thức nền tảng về log và bảo mật cơ bản trên **Linux** và **Windows** — hai hệ điều hành phổ biến nhất trong môi trường server — sau đó so sánh và tổng hợp thành một checklist thực hành.

---

<a id="phan-i-log-va-security-tren-linux"></a>
# Phần I. Log và Security trên Linux

<a id="1-cac-loai-logs-quan-trong"></a>
## 1. Các loại Logs quan trọng

Hầu hết các tệp nhật ký Linux được lưu dưới dạng văn bản thuần (ASCII) trong thư mục `/var/log` và các thư mục con, được tạo bởi các daemon ghi log như `syslogd`, `rsyslogd` hoặc `systemd-journald`. Quản lý tốt các nhật ký này đảm bảo dữ liệu luôn sẵn sàng để phân tích, khắc phục sự cố và điều tra bảo mật.

Log trên Linux thường được chia thành 4 nhóm chính:

| Nhóm | Mô tả |
|---|---|
| System Logs | Log liên quan trực tiếp đến hoạt động của hệ điều hành |
| Event Logs | Log ghi nhận các sự kiện cụ thể (đăng nhập, khởi động lại, lỗi phần cứng...) |
| Application Logs | Log do các ứng dụng cài thêm (Apache, MySQL, Samba...) sinh ra |
| Service Logs | Log của các dịch vụ hệ thống (cron, daemon...) |

<a id="11-system-logs"></a>
### 1.1. System Logs

Đây là các nhật ký gắn liền với hoạt động lõi của hệ thống Ubuntu, không phụ thuộc vào ứng dụng người dùng cài thêm.

| Tệp log | Đường dẫn | Nội dung |
|---|---|---|
| Authorization Log | `/var/log/auth.log` | Ghi lại việc sử dụng cơ chế xác thực: PAM, `sudo`, đăng nhập SSH từ xa... Hữu ích để theo dõi thông tin đăng nhập và việc dùng `sudo`. |
| Daemon Log | `/var/log/daemon.log` | Log của các daemon chạy nền (gdm, hcid, mysqld...), giúp khắc phục sự cố của một daemon cụ thể. |
| Debug Log | `/var/log/debug` | Thông báo debug chi tiết ở mức DEBUG từ hệ thống và ứng dụng. |
| Kernel Log | `/var/log/kern.log` | Thông báo từ kernel: phần cứng, module kernel. |
| Kernel Ring Buffer | `/var/log/dmesg` | Thông tin về quá trình khởi động, phát hiện phần cứng, khởi tạo driver. |
| System Log | `/var/log/syslog` | Chứa nhiều thông tin nhất theo mặc định; nơi tra cứu khi không tìm thấy thông tin ở log khác. Trên các bản phân phối khác còn có `/var/log/messages` với vai trò tương tự. |

<a id="12-application-logs"></a>
### 1.2. Application Logs

Nhiều ứng dụng cũng tạo log riêng trong `/var/log/<tên-ứng-dụng>`:

- **Apache HTTP Server** (`/var/log/apache2/`):
  - `access.log`: ghi lại mọi trang/tệp được phục vụ — địa chỉ IP, thời gian, user-agent, mã HTTP, nội dung truy vấn (thường là `GET`).
  - `error.log`: ghi lại mọi lỗi mà HTTP server báo cáo — quan trọng khi PHP tắt hiển thị lỗi.
- **CUPS Print System**: `/var/log/cups/error_log`.
- **Rootkit Hunter (rkhunter)**: `/var/log/rkhunter.log` — kiểm tra backdoor, sniffer, rootkit.
- **Samba (SMB) Server** (`/var/log/samba/`):
  - `log.nmbd`: thông tin NETBIOS over IP.
  - `log.smbd`: thông tin chia sẻ tệp/bản in (SMB/CIFS).
  - `log.<IP_ADDRESS>`: log riêng theo từng địa chỉ IP kết nối.
- **X11 Server**: `/var/log/Xorg.0.log`.

<a id="13-non-human-readable-logs"></a>
### 1.3. Non-Human-Readable Logs

Một số log trong `/var/log` được thiết kế để ứng dụng/tiện ích đọc bằng công cụ chuyên dụng thay vì đọc trực tiếp:

| Tệp log | Đường dẫn | Công cụ đọc |
|---|---|---|
| Login Failures | `/var/log/faillog` | lệnh `faillog` |
| Last Logins | `/var/log/lastlog` | lệnh `lastlog` |
| Login Records | `/var/log/wtmp` | lệnh `who`, `last` |

<a id="14-kiem-tra-log-dang-nhap"></a>
### 1.4. Kiểm tra log đăng nhập, reboot/shutdown

```bash
# Hiển thị người dùng hiện đang đăng nhập, terminal, thời gian, IP nguồn
who

# Danh sách các lần đăng nhập gần đây (đọc từ /var/log/wtmp)
last

# Thời gian và ngày khởi động lại gần nhất
last reboot

# Số lần hệ thống đã khởi động, dùng journalctl
journalctl --list-boot
```

---

<a id="2-cong-cu-quan-ly-logs"></a>
## 2. Công cụ quản lý Logs

<a id="21-journalctl"></a>
### 2.1. journalctl

`journald` là systemd journal daemon — thành phần trung tâm của systemd, đi kèm nhiều bản phân phối như Ubuntu, Fedora, Arch Linux. `journalctl` là tiện ích để truy vấn và hiển thị các log này.

- File cấu hình: `/etc/systemd/journald.conf`.
- Log được lưu ở dạng **nhị phân** tại `/var/log/journal`.

**Các lệnh thường dùng:**

```bash
# Xem log cơ bản
journalctl

# Hiển thị theo múi giờ UTC
journalctl --utc

# Lọc theo mức độ nghiêm trọng (0=emergency ... 7=debug)
journalctl -p <code>

# Lọc theo khoảng thời gian
journalctl --since "2020-12-04 06:00:00"
journalctl --since "2020-12-03" --until "2020-12-05 03:00:00"
journalctl --since yesterday
journalctl --since 09:00 --until "1 hour ago"

# Log liên quan đến kernel
journalctl -k

# Log của một service cụ thể
journalctl -u apache2.service

# Log của một user/group (tra UID bằng `id -u <user>`)
journalctl _UID=1001 --since today

# Log của một tiến trình thực thi cụ thể
journalctl /usr/bin/gnome-shell --since today
```

Bảng mã mức độ nghiêm trọng dùng cho `-p`:

| Code | Mức độ |
|---|---|
| 0 | emergency |
| 1 | alerts |
| 2 | critical |
| 3 | errors |
| 4 | warning |
| 5 | notice |
| 6 | info |
| 7 | debug |

<a id="22-logrotate"></a>
### 2.2. logrotate

**Logrotate** là công cụ quản lý vòng đời file log: tự động xoay vòng (rotate), nén, xoá log cũ để tránh log phình to gây tắc nghẽn hệ thống.

**Cách hoạt động:** logrotate chạy theo lịch (thường hàng ngày qua `/etc/cron.daily/logrotate`). Mỗi lần chạy nó sẽ:
1. Đọc tệp cấu hình để biết log nào cần rotate, tần suất, số bản lưu.
2. Kiểm tra điều kiện (kích thước, thời gian).
3. Thực hiện rotate: đổi tên/xoá log cũ, tạo log mới, nén log cũ (thường bằng gzip).
4. Chạy script trước/sau rotate nếu được cấu hình (ví dụ restart service).

**Cấu trúc cấu hình:**
- `/etc/logrotate.conf`: cấu hình chung, mặc định cho mọi log trừ khi bị ghi đè.
- `/etc/logrotate.d/`: cấu hình riêng cho từng ứng dụng/dịch vụ.

**Kiểm tra version:**
```bash
logrotate --version
```

**Các tuỳ chọn cấu hình phổ biến:**

```conf
/root/logrotate_demo/myapp.log {
    daily          # tần suất: daily | weekly | monthly | yearly
    rotate 3       # giữ tối đa 3 bản cũ, -1 = không giới hạn theo số lượng
    compress       # nén bằng gzip
    dateext        # thêm hậu tố ngày vào tên file đã rotate (myapp.log-20250513)
    missingok      # không báo lỗi nếu file bị thiếu
    notifempty     # không rotate nếu file rỗng
    size 100M      # (tuỳ chọn) rotate theo dung lượng thay vì thời gian
    delaycompress  # trì hoãn nén sang lần rotate kế tiếp
    create 660 ubuntu www-data   # quyền + owner cho file log mới tạo
    prerotate
        # lệnh chạy trước khi rotate
    endscript
}
```

**Chạy thử / chạy thủ công:**
```bash
# Chạy ở chế độ debug (không thực thi thật)
sudo logrotate -d /etc/logrotate.d/myapp.conf

# Chạy verbose + ép rotate ngay lập tức
sudo logrotate -vf /etc/logrotate.d/myapp.conf
```

<a id="23-syslog"></a>
### 2.3. syslog

**Tổng quan:** Syslog (System Logging Protocol) là giao thức client/server dùng để chuyển log/thông điệp đến máy nhận log (syslog server/daemon), qua UDP hoặc TCP, cổng mặc định **514**, dữ liệu truyền dạng cleartext. Được Eric Allman phát triển năm 1980 như một phần của dự án Sendmail, sau đó trở thành chuẩn ghi log phổ biến trên Unix/Linux và cả thiết bị mạng (router...). IETF chuẩn hoá syslog trong **RFC 5424** (2009); độ tin cậy truyền tải được bổ sung qua RFC 3195 và RFC 6587 (truyền qua TCP).

**Cấu trúc thông điệp Syslog** (tối đa 1024 byte), gồm 3 phần: **PRI – HEADER – MSG**

- **PRI**: số 8-bit thể hiện *facility* (cơ sở sinh log, 5 bit) và *severity* (mức nghiêm trọng, 3 bit).
  - Ví dụ: Priority = 191 → 191 ÷ 8 = 23 dư 7 → Facility = 23 (`local7`), Severity = 7 (`debug`).
- **HEADER**: gồm `TIMESTAMP` (định dạng `Mmm dd hh:mm:ss`, lấy theo giờ hệ thống của máy gửi) và `HOSTNAME`.
- **MSG**: nội dung sự kiện, chia thành `TAG` (chương trình sinh log) và `CONTENT` (chi tiết).

Facility phổ biến: `auth`, `authpriv`, `daemon`, `cron`, `ftp`, `dhcp`, `kern`, `mail`, `syslog`, `user`...
Severity (từ cao xuống thấp): Emergency, Alert, Critical, Error, Warning, Notice, Info, Debug.

**Syslog Forwarding:** gửi log từ nhiều máy client đến một syslog server tập trung, giúp phân tích dễ hơn khi phải quản lý hàng chục máy — dùng UDP hoặc TCP qua cổng 514.

<a id="24-rsyslog"></a>
### 2.4. rsyslog

"The rocket-fast system for log processing" — bắt đầu phát triển từ 2004 bởi Rainer Gerhards, hiện được cài đặt sẵn trên hầu hết bản phân phối Linux (Ubuntu, Fedora, Debian, RHEL...). Rsyslog là bản kế thừa mở rộng của syslog, hỗ trợ module hoá, filter riêng, template định dạng tuỳ biến, và truyền tải qua TCP.

**4 thành phần chính:**

| Thành phần | Vị trí |
|---|---|
| File cấu hình | `/etc/rsyslog.conf`, `/etc/rsyslog.d/` |
| Thư viện/module | `/usr/lib64/rsyslog/`, `/usr/share/doc/rsyslog` |
| Nơi lưu log | `/var/log` |

**Một số module đáng nhớ:**
- **Input:** `imuxsock` (Unix socket), `imklog` (kernel), `imfile` (đọc từ file), `imtcp`/`imudp` (qua TCP/UDP), `imrelp` (RELP).
- **Output:** `omfile` (ghi ra file), `omstdout`, `omprog` (đến chương trình khác), `omrelp`.
- **Parser/Stats:** `impstats` (thống kê hoạt động rsyslog).
- **Library:** `librelp`, `libgcry` (mã hoá), `libgpg-error`.

**Cấu hình:** mỗi dòng cấu hình gồm 2 trường, cách nhau bởi khoảng trắng:

```conf
# <selector>            <action>
mail.info                /var/log/maillog
```

- **Selector:** nguồn log + mức cảnh báo, gồm 2 phần cách nhau bởi dấu `.` — ví dụ `mail.info` nghĩa là facility `mail` từ mức `info` trở lên. Muốn log đúng mức `info` (không lấy các mức cao hơn): `mail.=info`. Ký tự `*` đại diện cho tất cả mức/facility; `!` dùng để loại trừ (`mail.!info`).
- **Action:** nơi lưu log — file cục bộ hoặc gửi đến IP của một log server từ xa.

```conf
# Ví dụ: log tổng hợp mức info trở lên, loại trừ mail/authpriv/cron
*.info;mail.none;authpriv.none;cron.none    /var/log/messages
```

<a id="25-syslog-ng"></a>
### 2.5. syslog-ng

Một daemon syslog mạnh mẽ khác, mở rộng thêm khả năng lưu log vào cơ sở dữ liệu, lọc/chuyển đổi log linh hoạt, ghi log an toàn qua TLS, hỗ trợ hàng đợi tin nhắn, tích hợp SQL/NoSQL, và tuân thủ chuẩn syslog (IPv4/IPv6).

**Kiến trúc pipeline** gồm 4 khối:

1. **Source** — nơi nhận log đến.
2. **Filter** — điều kiện lọc thông điệp.
3. **Destination** — nơi log được gửi tới.
4. **Log statement** — kết nối source, filter, destination thành một luồng xử lý.

**File cấu hình:** `/etc/syslog-ng/syslog-ng.conf`

```conf
@version: 3.38

source s_local {
    unix-dgram("/dev/log"); internal();
};

destination d_file {
    file("/var/log/messages_syslog-ng.log");
};

log {
    source(s_local); destination(d_file);
};
```

**Source drivers thường gặp:** `internal()` (thông điệp nội bộ của syslog-ng), `unix-stream()/unix-dgram()/pipe()` (socket/pipe), `file()`, `network()` (RFC 3164), `syslog()` (RFC 5424), `program()`, `python()`, `system()` (tự động thu thập log hệ thống, chuyển đổi mượt giữa `/dev/log` và `systemd-journal`).

**Destination drivers thường gặp:** `file()`, `pipe()/unix-stream()/unix-dgram()`, `network()`, `syslog()`, `usertty()`, `program()`, `http()` (gửi tới Elasticsearch, Slack, Sumologic...), `python()`.

**Filter ví dụ:**
```conf
filter f_default { level(info..emerg) and not (facility(mail)); };
```
Filter phổ biến: `level` (theo mức nghiêm trọng), `facility` (theo danh mục nguồn).

<a id="26-auditd"></a>
### 2.6. auditd — Audit Framework

Bên cạnh syslog/rsyslog, Linux còn có **Linux Audit Framework** (`auditd`) — hệ thống ghi log ở mức chi tiết hơn nhiều: theo dõi trực tiếp **system call**, có thể cấu hình để giám sát việc đọc/ghi vào các file, thư mục hoặc hành vi cụ thể. Đây là công cụ quan trọng cho các yêu cầu tuân thủ (compliance) vì độ chi tiết và khả năng truy vết theo từng sự kiện.

**Cài đặt:**
```bash
sudo apt install auditd audispd-plugins   # Debian/Ubuntu
sudo systemctl enable --now auditd
```

**Log được lưu tại:** `/var/log/audit/audit.log`

**Thêm rule giám sát cơ bản** — chỉnh sửa `/etc/audit/rules.d/audit.rules`:

```conf
# Theo dõi thay đổi user/group
-w /etc/passwd -p wa -k user_modification
-w /etc/group  -p wa -k group_modification

# Theo dõi thay đổi cấu hình mạng
-w /etc/network/ -p wa -k network_modifications

# Theo dõi thay đổi sudoers
-w /etc/sudoers -p wa -k sudoers_changes
```

```bash
sudo systemctl restart auditd
```

**Tra cứu log audit với `ausearch`:**
```bash
# Tất cả lệnh chạy qua sudo
ausearch -m USER_CMD -k sudo_log

# Hành động của một user cụ thể trong ngày hôm nay
ausearch -ua root -ts today

# Log gắn với một key cụ thể
ausearch -k user_modification -i
```

> `auditd` bổ sung cho syslog/rsyslog chứ không thay thế: syslog phù hợp cho log vận hành tổng quát, còn auditd phù hợp khi cần bằng chứng chi tiết, có thể tra vết theo system call cho mục đích điều tra và tuân thủ.


---

<a id="3-security-co-ban-tren-linux"></a>
## 3. Security cơ bản trên Linux

<a id="31-bao-mat-dang-nhap"></a>
### 3.1. Bảo mật đăng nhập

- Sử dụng mật khẩu mạnh.
- Vô hiệu hoá đăng nhập root trực tiếp qua SSH.
- Ưu tiên xác thực bằng **SSH key** thay vì mật khẩu. Chỉnh sửa `/etc/ssh/sshd_config`:

```conf
PasswordAuthentication no   # tắt xác thực bằng mật khẩu
Port 2222                   # đổi cổng SSH mặc định
PermitRootLogin no          # vô hiệu hoá đăng nhập root
PermitEmptyPasswords no     # tắt mật khẩu trống
```

```bash
sudo systemctl restart sshd
```

<a id="32-quan-ly-user-va-quyen"></a>
### 3.2. Quản lý user và quyền

- **Nguyên tắc đặc quyền tối thiểu (PoLP – Principle of Least Privilege):** chỉ cấp cho user quyền truy cập cần thiết cho công việc của họ, không hơn.
- **umask:** thiết lập quyền mặc định khi tạo file/thư mục mới.
  - File mặc định bắt đầu ở `666` (rw-rw-rw-), thư mục ở `777` (rwxrwxrwx); giá trị umask được trừ đi từ đó.
  - Ví dụ `umask 027` cho ra: thư mục `750` (rwxr-x---), file `640` (rw-r-----) — cân bằng giữa bảo mật và khả năng dùng.
- **Lập kế hoạch quyền theo nhóm:**
  - Nhóm user theo logic (dev, marketing, HR...).
  - Áp dụng RBAC (Role-Based Access Control): gán quyền cho vai trò, không gán trực tiếp cho từng cá nhân.
  - Rà soát định kỳ thành viên nhóm, gỡ quyền không còn cần thiết.
  - Không dùng trực tiếp tài khoản root; dùng `sudo` để giới hạn và ghi log hành vi quản trị.
- **ACL (Access Control List):** kiểm soát quyền chi tiết hơn permission chuẩn.

```bash
# Xem ACL của một file
getfacl filename

# Gán quyền rwx cho một user cụ thể
setfacl -m u:username:rwx filename
```

<a id="33-firewall"></a>
### 3.3. Firewall (Tường lửa)

Bảo mật hệ thống bằng tường lửa nghĩa là cấu hình quy tắc kiểm soát lưu lượng mạng đến/đi. Công cụ phổ biến: `iptables` (IPv4, truyền thống), `nftables` (kế thừa iptables), hoặc công cụ cấp cao hơn như `ufw` (Uncomplicated Firewall) và `firewalld`.

- Mặc định: lưu lượng **đến** thường bị từ chối, lưu lượng **đi** thường được phép.
- Cần rà soát và cập nhật rule định kỳ để thích ứng với yêu cầu bảo mật thay đổi.

```bash
# Kiểm tra trạng thái/rule hiện tại
sudo iptables -L

# Chỉ cho phép dịch vụ cần thiết (vd: SSH)
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Từ chối các kết nối không được liệt kê rõ (đặt sau các rule ACCEPT)
sudo iptables -A INPUT -j DROP
```

<a id="34-cap-nhat-he-thong"></a>
### 3.4. Cập nhật hệ thống

Bản cập nhật phần mềm thường đi kèm bản vá bảo mật quan trọng — nên cập nhật thường xuyên.

```bash
# Debian/Ubuntu
sudo apt update && sudo apt upgrade -y

# RHEL/Fedora/CentOS
sudo yum update
```

<a id="35-giam-sat-he-thong"></a>
### 3.5. Giám sát hệ thống

Kiểm tra log thường xuyên là nền tảng của giám sát bảo mật. Để tăng cường bảo mật, nên cân nhắc chuyển log đến một máy chủ log tập trung — giúp ngăn kẻ tấn công dễ dàng xoá dấu vết trên máy cục bộ sau khi xâm nhập.

**Một số tệp log mặc định quan trọng:**

| Log | Mục đích |
|---|---|
| `/var/log/messages` | Nhật ký hoạt động toàn hệ thống |
| `/var/log/auth.log` (`secure` trên RHEL) | Sự kiện xác thực |
| `/var/log/kern.log` | Log liên quan kernel |
| `/var/log/cron.log` | Log tác vụ cron |
| `/var/log/maillog` | Log máy chủ thư |
| `/var/log/boot.log` | Hoạt động khởi động hệ thống |
| `/var/log/mysqld.log` | Log MySQL server |
| `/var/log/wtmp` / `/var/log/utmp` | Hồ sơ đăng nhập |
| `/var/log/yum.log` / `dpkg.log` | Log trình quản lý gói |

**Công cụ giám sát tài nguyên hệ thống:**
```bash
top      # tiến trình theo thời gian thực
htop     # phiên bản top nâng cao
vmstat   # thống kê bộ nhớ ảo
netstat  # kết nối mạng (cũ)
ss       # kết nối mạng (thay thế netstat)
```

**Công cụ IDS/IPS** (phát hiện & ngăn chặn xâm nhập): Snort, Suricata, Splunk...

**Đồng bộ thời gian bằng NTP:** timestamp không đồng nhất giữa các máy sẽ khiến việc đối chiếu log giữa nhiều nguồn trở nên không chính xác.

```bash
sudo apt install ntp
sudo systemctl enable --now ntp
```

**Đẩy log ra máy chủ tập trung (remote logging) qua rsyslog** — cấu hình phía client trong `/etc/rsyslog.conf`:

```conf
*.* @192.168.1.100:514     # gửi qua UDP; dùng @@ để gửi qua TCP
```

Phía server nhận cần bật module UDP/TCP tương ứng, ví dụ:

```conf
module(load="imudp")
input(type="imudp" port="514")
```

<a id="36-aide"></a>
### 3.6. Kiểm tra tính toàn vẹn log với AIDE

**AIDE (Advanced Intrusion Detection Environment)** là công cụ tạo "dấu vân tay" (checksum) cho các file quan trọng — bao gồm cả file log — để phát hiện việc chỉnh sửa/xoá trái phép, một dấu hiệu điển hình khi kẻ tấn công cố gắng xoá vết sau khi xâm nhập.

```bash
sudo apt install aide
sudo aide --init
sudo cp /var/lib/aide/aide.db.new /var/lib/aide/aide.db

# Chạy kiểm tra định kỳ (nên đặt lịch qua cron)
sudo aide --check
```

---

<a id="4-phan-tich-logs-linux"></a>
## 4. Phân tích logs để phát hiện tấn công (Linux)

<a id="41-tong-quan"></a>
### 4.1. Tổng quan

Phân tích nhật ký là quá trình thu thập, tập trung và phân tích dữ liệu log do hệ thống, ứng dụng, thiết bị sinh ra nhằm tìm ra mẫu bất thường, xác định nguyên nhân gốc rễ và phát hiện các dấu hiệu bảo mật. Việc phân tích log cẩn thận giúp:

- **Phát hiện sớm dấu hiệu tấn công** — ví dụ nhiều lần đăng nhập thất bại liên tiếp từ một IP là dấu hiệu brute-force.
- **Ứng phó sự cố hiệu quả** — log cho biết kẻ tấn công xâm nhập bằng cách nào, đã làm gì, khi nào.
- **Nâng cao khả năng phòng thủ** — phân tích mẫu tấn công cũ để vá lỗ hổng chủ động.
- **Đảm bảo tuân thủ** — chứng minh việc tuân thủ các chuẩn/quy định bảo mật.

**Quy trình phân tích log điển hình:**

1. **Thu thập dữ liệu** — gom log từ hệ thống, ứng dụng, thiết bị mạng.
2. **Lập chỉ mục dữ liệu** — tổ chức để dễ tìm kiếm/phân tích.
3. **Phân tích** — áp dụng kỹ thuật phù hợp để tìm dấu hiệu tấn công.
4. **Giám sát** — theo dõi liên tục theo thời gian thực.
5. **Báo cáo** — tổng hợp phát hiện, sự cố tiềm ẩn hoặc đã xảy ra.

**Các nguồn log quan trọng:** log hệ thống (`auth.log`, `syslog`), log ứng dụng, log web server (access/error), log firewall, log IDS/IPS, log proxy, log DNS, log kiểm soát truy cập.

<a id="42-phuong-phap"></a>
### 4.2. Phương pháp và dấu hiệu tấn công

Với khối lượng log khổng lồ, việc phân tích thủ công gần như bất khả thi ở quy mô lớn — cần các kỹ thuật:

- **Chuẩn hoá (Normalization):** đưa các thuộc tính (IP, timestamp...) về một định dạng nhất quán.
- **Nhận dạng mẫu (Pattern recognition):** lọc sự kiện dựa trên "sổ mẫu" để phân biệt sự kiện thường lệ và bất thường.
- **Phân loại và gắn thẻ (Classification & tagging):** gắn từ khoá, nhóm các sự kiện liên quan.
- **Phân tích tương quan (Correlation):** kết hợp log từ nhiều nguồn để nhìn tổng thể.

**Dấu hiệu của một số loại tấn công phổ biến:**

| Loại tấn công | Dấu hiệu trong log |
|---|---|
| Brute-force | Nhiều lần đăng nhập thất bại liên tiếp từ cùng 1 IP hoặc nhiều IP khác nhau |
| SQL Injection | Request HTTP chứa ký tự/câu lệnh SQL bất thường (`'; DROP TABLE`, `UNION SELECT`) |
| Cross-Site Scripting (XSS) | Request chứa đoạn mã JS đáng ngờ (`<script>`, `onerror`, `alert`) |
| Directory Traversal | Request cố truy cập ngoài thư mục gốc (`../../../../`) |
| DoS/DDoS | Lượng lớn request dồn dập vào một dịch vụ gây quá tải |
| Malware activity | Kết nối đến IP/domain độc hại đã biết, tiến trình bất thường |
| Privilege Escalation | Hành động vượt quyền hạn thông thường của user, khai thác lỗ hổng leo thang |
| Web Shell activity | Request tới các file dạng `cmd.php`, `admin.jsp`, hành vi đáng ngờ qua giao diện web |
| Lateral Movement | Đăng nhập thành công bất thường giữa các máy nội bộ sau khi đã xâm nhập một máy |

<a id="43-cong-cu-cli"></a>
### 4.3. Công cụ dòng lệnh hỗ trợ phân tích log

**grep** — tìm kiếm văn bản:

```bash
# Tìm kiếm cơ bản
grep "user ubuntu" /var/log/auth.log

# Dùng regex (Perl-compatible)
grep -P "(?<=port\s)4792" /var/log/auth.log

# Surround search: hiện thêm dòng trước/sau kết quả khớp
grep -B 3 -A 2 'Invalid user contador from 75.178.99.71' /var/log/auth.log
```

**tail** — theo dõi log realtime:

```bash
# In 5 dòng cuối
tail -n 5 /var/log/syslog

# Theo dõi realtime kết hợp lọc bằng grep
tail -f /var/log/auth.log | grep 'Invalid user'
```

**cut** — trích trường từ bản ghi có dấu phân cách:

```bash
# Trích IP người truy cập từ access log của Apache
cat access.log | cut -d ' ' -f 1
```

**awk** — công cụ mạnh cho việc lọc/xử lý theo trường:

```bash
awk '/Accepted password for/ || /Accepted publickey for/ { print $11 }' /var/log/auth.log
```

**ausearch** — tra cứu log của `auditd` (xem thêm [mục 2.6](#26-auditd)).

<a id="44-cong-cu-giam-sat"></a>
### 4.4. Công cụ giám sát & quản lý log mã nguồn mở

| Công cụ | Đặc điểm |
|---|---|
| **Graylog** | Nền tảng quản lý log tập trung, có thể lưu trữ hàng terabyte log/ngày, giao diện web hỗ trợ tìm kiếm, sắp xếp, hiển thị dạng bảng/biểu đồ. |
| **Elastic Stack (ELK)** | Bộ 3 dự án mã nguồn mở Elasticsearch (lưu trữ/tìm kiếm), Logstash (thu thập/xử lý), Kibana (trực quan hoá) — thường kết hợp Filebeat để đẩy log lên. |
| **Logcheck** | Tự động rà soát log, phát hiện sự cố/vi phạm bảo mật, gửi báo cáo định kỳ qua email. |
| **OSSEC** | Hệ thống phát hiện xâm nhập dựa trên host (HIDS), mạnh về phân tích log, kiểm tra tính toàn vẹn file, phát hiện rootkit. |

<a id="45-vi-du-thuc-chien"></a>
### 4.5. Ví dụ thực chiến phân tích log

**Phát hiện brute-force SSH — đếm số lần đăng nhập thất bại theo IP:**

```bash
grep "Failed password" /var/log/auth.log | grep "ssh" \
  | awk '{print $11}' | sort | uniq -c | sort -nr
```

**Theo dõi việc leo thang đặc quyền qua `sudo`:**

```bash
grep "COMMAND=" /var/log/auth.log
```

**Phát hiện tài khoản mới được tạo bất thường:**

```bash
grep "new user" /var/log/auth.log
grep -Ei "add.*user" /var/log/auth.log
```

**Theo dõi thay đổi quyền file bằng audit:**

```bash
ausearch -k file_modifications -i
```

> Sau khi xác định được IP nghi ngờ từ log, nên tiếp tục đối chiếu IP đó với các nguồn log khác (web server, mail server, firewall) để dựng lại toàn bộ dấu vết của kẻ tấn công thay vì chỉ dừng ở lớp SSH.

---

<a id="5-tu-dong-hoa"></a>
## 5. Tự động hoá giám sát & cảnh báo

Kiểm tra log thủ công không khả thi khi hệ thống có quy mô lớn. Một số cách tự động hoá cơ bản:

**Script cảnh báo khi có nhiều lần đăng nhập SSH thất bại:**

```bash
#!/bin/bash
THRESHOLD=5
COUNT=$(grep "Failed password" /var/log/auth.log | grep "ssh" | wc -l)

if [ "$COUNT" -gt "$THRESHOLD" ]; then
  echo "Alert: $COUNT failed SSH login attempts detected" \
    | mail -s "Security Alert" admin@example.com
fi
```

**Đặt lịch kiểm tra định kỳ bằng cron:**

```bash
# Chạy kiểm tra mỗi 15 phút
*/15 * * * * /path/to/security_check.sh
```

**Gửi cảnh báo tới Slack/Teams qua webhook:**

```bash
curl -X POST -H 'Content-type: application/json' \
  --data '{"text":"Security Alert: Multiple failed logins detected"}' \
  https://hooks.slack.com/services/YOUR/WEBHOOK/URL
```

> Với hệ thống quy mô lớn hơn, nên cân nhắc dùng nền tảng quan sát (observability) chuyên dụng để gom log, metric, trace và cảnh báo tập trung thay vì chỉ dựa vào script rời rạc.

---

<a id="6-xu-ly-su-co"></a>
## 6. Xử lý sự cố logging thường gặp

**Log không được sinh ra:**

```bash
# Kiểm tra quyền truy cập thư mục/tệp log
ls -la /var/log
sudo chmod 755 /var/log
sudo chmod 640 /var/log/auth.log

# Kiểm tra cú pháp cấu hình rsyslog
sudo rsyslogd -f /etc/rsyslog.conf -N1

# Kiểm tra trạng thái dịch vụ
sudo systemctl status rsyslog
sudo systemctl status auditd
```

**File log phình to quá nhanh:**

```conf
# Lọc bớt sự kiện không quan trọng trong rsyslog.conf
if $programname != 'sshd' then stop

# Rotate gắt hơn theo dung lượng trong logrotate.conf
size 100M
```

**Không tìm thấy sự kiện bảo mật cần tra cứu:**

```bash
# Kết hợp nhiều điều kiện tìm kiếm với auditd
ausearch -i -k user_modification | grep username

# Tìm kiếm có cấu trúc bằng journalctl
journalctl _COMM=sshd --since today

# Tìm nhanh bằng grep
grep 'authentication failure' /var/log/auth.log
```

**Định nghĩa chính sách retention (thời gian lưu log):**

```conf
# Trong logrotate.conf — ví dụ giữ log 90 ngày
rotate 90
```

> Gợi ý: 90 ngày là mốc phổ biến cho nhu cầu vận hành thông thường; một số chuẩn tuân thủ (ví dụ PCI DSS) yêu cầu lưu tối thiểu 1 năm — cần cân bằng giữa yêu cầu bảo mật/tuân thủ và chi phí lưu trữ.

