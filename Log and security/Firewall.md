# Firewall — Linux, Windows & Tổng quan Tcpdump/Wireshark

## Mục lục

- [I. Firewall trên Linux](#i-firewall-trên-linux)
  - [1. Tổng quan](#1-tổng-quan)
  - [2. Yêu cầu cơ bản](#2-yêu-cầu-cơ-bản)
    - [2.1. iptables](#21-iptables)
    - [2.2. firewalld](#22-firewalld)
    - [2.3. ufw](#23-ufw)
    - [2.4. csf](#24-csf)
    - [2.5. apf](#25-apf)
  - [3. Tìm hiểu: Cấu hình, sử dụng](#3-tìm-hiểu-cấu-hình-sử-dụng)
    - [3.1. iptables](#31-iptables)
    - [3.2. firewalld](#32-firewalld)
    - [3.3. ufw](#33-ufw)
    - [3.4. csf](#34-csf)
    - [3.5. apf](#35-apf)
- [II. Firewall trên Windows](#ii-firewall-trên-windows)
  - [1. Tổng quan](#1-tổng-quan-1)
  - [2. Yêu cầu cơ bản](#2-yêu-cầu-cơ-bản-1)
  - [3. Tìm hiểu: Cấu hình, sử dụng](#3-tìm-hiểu-cấu-hình-sử-dụng-1)
- [III. So sánh](#iii-so-sánh)
- [IV. Tìm hiểu về Tcpdump và Wireshark](#iv-tìm-hiểu-về-tcpdump-và-wireshark)
  - [1. Tổng quan](#1-tổng-quan-2)
  - [2. Cài đặt, cấu hình](#2-cài-đặt-cấu-hình)
- [References](#references)

---

## I. Firewall trên Linux

### 1. Tổng quan

Chức năng tường lửa Linux được cung cấp bởi **Netfilter** — một mô-đun kernel có trong tất cả bản phân phối Linux. Một số công cụ quản trị firewall phổ biến dựa trên Netfilter: **iptables, nftables, ufw, firewalld, csf, apf**.

### 2. Yêu cầu cơ bản

- **Giao thức:** quản trị tường lửa dựa trên giao thức mạng (TCP/UDP) để xác định lưu lượng nào được phép/chặn. Rule thường được định nghĩa theo giao thức + cổng + địa chỉ IP.
  - TCP: truyền ổn định (HTTP, HTTPS, SSH).
  - UDP: truyền nhanh, không đảm bảo (DNS, VoIP).
  - Cổng phổ biến: HTTP 80, HTTPS 443, SSH 22, FTP 21, MySQL 3306...
- **Quyền quản trị:** cần quyền root để cài đặt/cấu hình firewall.
- **OS tương ứng:** mỗi distro có công cụ mặc định riêng:
  - Debian/Ubuntu: **ufw, iptables**
  - CentOS/Fedora: **firewalld, iptables**

#### 2.1. iptables

Ứng dụng quản lý packet filtering và NAT rules, chạy trên console Linux — nhỏ gọn, miễn phí. Gồm 2 phần: **netfilter** (nằm trong kernel) và **iptables** (nằm ngoài kernel, giao tiếp với người dùng rồi đẩy rule vào cho netfilter xử lý ở mức IP).

**Tính năng chính:** phân tích gói tin hiệu quả; filtering theo MAC/TCP flags; hỗ trợ NAT; ghi log sự kiện; chống một số kiểu tấn công DoS; xây dựng & quản lý rule.

**3 thành phần cốt lõi — Tables, Chains, Targets:**
- **Table:** cách xử lý gói tin theo mục đích cụ thể (mặc định là `filter` nếu không chỉ định).
- **Chain:** gắn vào table, cho phép xử lý gói tin ở các giai đoạn khác nhau: `PREROUTING`, `INPUT`, `OUTPUT`, `FORWARD`, `POSTROUTING`.
- **Target:** hành động áp dụng cho gói tin khớp rule: `ACCEPT` (cho qua), `DROP` (loại bỏ, không phản hồi), `RETURN` (dừng rule hiện tại, trả về chain gọi), `REJECT` (loại bỏ + phản hồi lỗi), `LOG` (chấp nhận + ghi log).

**Cấu trúc 3 table chính:**
| Table | Chain con | Vai trò |
|---|---|---|
| **FILTER** (mặc định) | INPUT, OUTPUT, FORWARD | Quyết định gói tin có được chuyển đến đích hay không. |
| **NAT** | PREROUTING (DNAT), POSTROUTING (SNAT), OUTPUT | Thay đổi IP nguồn/đích để route gói tin đến host khác. |
| **MANGLE** | PREROUTING, OUTPUT, FORWARD, INPUT | Sửa header gói tin (TTL, MTU, Type of Service). |

**Khái niệm khác:** NAT (đổi IP/port nguồn-đích), filtering (chặn theo tiêu chí), mangle (sửa QoS bits trong IP Header), rule (luật cụ thể), port (vị trí ra/vào của gói TCP/UDP trên 1 IP).

#### 2.2. firewalld

Mặc định trên CentOS/RHEL — quản lý theo **Zones**, cho phép thay đổi cấu hình linh hoạt mà không cần restart firewall.

**Zone** là tập rule định sẵn theo mức độ tin cậy của nguồn traffic, xếp từ ít tin cậy nhất → tin cậy nhất:
`drop` → `block` → `public` → `external` → `internal` → `dmz` → `work` → `home` → `trusted`.

**Runtime vs Permanent:**
- **Runtime** (mặc định): có hiệu lực ngay, mất khi reboot.
- **Permanent:** cần reload mới áp dụng, tồn tại vĩnh viễn kể cả sau reboot.
- Restart/Reload sẽ hủy Runtime và áp dụng Permanent mà không phá vỡ session hiện tại.

#### 2.3. ufw

**UFW (Uncomplicated Firewall)** — công cụ cấu hình chạy trên nền iptables, có sẵn mặc định trên Ubuntu. Cung cấp giao diện dòng lệnh đơn giản, thân thiện cho cả người không chuyên bảo mật.

**Lợi ích:** kiểm soát traffic dễ dàng; bảo vệ cổng quan trọng (SSH, HTTP, MySQL); hạn chế DDoS cơ bản bằng giới hạn kết nối; kết hợp tốt với Fail2Ban để tăng bảo mật.

#### 2.4. csf

**CSF (ConfigServer Security & Firewall)** — tường lửa miễn phí, nâng cao cho hầu hết distro Linux/VPS. Ngoài lọc gói tin cơ bản còn tích hợp phát hiện đăng nhập/xâm nhập/flood. Có UI tích hợp cho cPanel, DirectAdmin, Webmin (hướng dẫn này chỉ dùng CLI).

**Các biện pháp bảo vệ chính:**
- **Xác thực lỗi đăng nhập:** kiểm tra log, tự block sau N lần sai — hỗ trợ Courier IMAP, Dovecot, openSSH, cPanel/WHM, Pure-ftpd/vsftpd/Proftpd, htpasswd, ModSecurity, Suhosin, Exim SMTP AUTH.
- **Theo dõi tiến trình/thư mục:** phát hiện tiến trình đáng ngờ, script độc hại trong `/tmp`, gửi email cảnh báo.
- **Port flood protection**, **Port knocking**, **giới hạn kết nối đồng thời** theo IP/port.
- **Chuyển hướng IP/port**, **danh sách chặn IP tự động tải từ nguồn ngoài**.

#### 2.5. apf

**APF (Advanced Policy Firewall)** — firewall dựa trên iptables/netfilter, thiết kế cho các server triển khai Internet. Quản lý hàng ngày qua lệnh `apf`. Bộ lọc gồm 3 phần:
- **Static rule based policies:** bộ luật cố định xử lý traffic theo điều kiện.
- **Connection based stateful policies:** chỉ chấp nhận gói tin khớp kết nối đã biết.
- **Sanity based policies:** khớp mẫu traffic với kiểu tấn công đã biết hoặc kiểm tra chuẩn Internet.

### 3. Tìm hiểu: Cấu hình, sử dụng

#### 3.1. iptables

Cài đặt (thường có sẵn mặc định):
```
sudo apt-get install iptables
```
Sau khi cài, rule IPv4/IPv6 lưu tại `/etc/iptables/rules.v4` và `/etc/iptables/rules.v6`.

**Cú pháp cơ bản:**
```
iptables -t {table} -{chain option} {chain} {matching component} {action}
```
- `-t`: chọn 1 trong 5 table có sẵn — bỏ qua thì mặc định dùng `filter`.
- **Chain option:** `-A` (append cuối chain), `-D` (xóa rule), `-F` (flush toàn bộ rule), `-I` (insert vào vị trí chỉ định), `-N` (tạo chain mới), `-X` (xóa chain), `-L` (liệt kê rule), `-v` (verbose).
- **Matching component:** `-p` (protocol), `-s` (source IP), `-d` (destination IP), `--dport`/`--sport` (port đích/nguồn), `-i`/`-o` (interface vào/ra).
- **Action:** `-j ACCEPT/DROP/REJECT/LOG...`

**Ví dụ thường dùng:**
```
# Cho phép SSH
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Chặn 1 IP cụ thể
sudo iptables -A INPUT -s 1.2.3.4 -j DROP

# Cho phép traffic đã thiết lập (stateful)
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
```

**NAT — Masquerade (dùng khi làm gateway/router):**
```
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```
Cần bật `sysctl net.ipv4.ip_forward=1` trước khi cấu hình.

#### 3.2. firewalld

Cài đặt & khởi động (mặc định có trên CentOS 7):
```
yum install firewalld
systemctl start firewalld
systemctl enable firewalld
```

**Lệnh cơ bản thường dùng:**
```
firewall-cmd --get-zones                     # xem danh sách zone
firewall-cmd --get-default-zone              # zone mặc định hiện tại
firewall-cmd --zone=public --add-service=http --permanent
firewall-cmd --zone=public --add-port=8080/tcp --permanent
firewall-cmd --reload                        # áp dụng permanent rule
firewall-cmd --list-all                      # xem toàn bộ rule zone hiện tại
```

#### 3.3. ufw

Cài đặt:
```
sudo apt install ufw
```

**Lệnh cơ bản:**
```
sudo ufw enable                # bật ufw
sudo ufw status verbose        # xem trạng thái + rule
sudo ufw allow 22/tcp          # cho phép port
sudo ufw allow from 192.168.1.0/24 to any port 22   # cho phép theo dải IP
sudo ufw deny 23               # chặn port
sudo ufw delete allow 22/tcp   # xóa rule
sudo ufw logging on            # bật log
```

#### 3.4. csf

Cài đặt:
```
wget http://download.configserver.com/csf.tgz
tar -xzf csf.tgz
cd csf
./install.sh
```

File cấu hình nằm ở `/etc/csf/`:
- `csf.conf`: cấu hình chính.
- `csf.allow`: IP/CIDR luôn được phép.
- `csf.deny`: IP/CIDR không bao giờ được phép.
- `csf.ignore`: IP/CIDR mà LFD (Login Failure Daemon) sẽ bỏ qua, không chặn.

**Một số option cấu hình hay chỉnh trong `csf.conf`:**
| Option | Ý nghĩa |
|---|---|
| `TCP_IN` / `TCP_OUT` / `UDP_IN` / `UDP_OUT` | Danh sách port cho phép theo chiều vào/ra. |
| `ICMP_IN` / `ICMP_IN_LIMIT` | Bật/tắt ping và giới hạn tần suất ping mỗi IP. |
| `CONNLIMIT` | Giới hạn số kết nối đồng thời trên mỗi port. VD: `22;5;443;20`. |
| `PORTFLOOD` | Giới hạn số kết nối mới theo thời gian. VD: `22;tcp;5;250` = quá 5 kết nối TCP/22 trong 250s thì chặn. |
| `LF_SSHD` / `LF_SSHD_PERM` | Số lần đăng nhập SSH sai tối đa / thời gian khóa IP vi phạm (giây). |
| `SYNFLOOD` / `SYNFLOOD_RATE` / `SYNFLOOD_BURST` | Bật chống SYN Flood + ngưỡng giới hạn. |
| `DENY_IP_LIMIT` | Số IP tối đa bị chặn vĩnh viễn (tránh giảm hiệu suất server). |

Áp dụng cấu hình sau khi sửa `csf.conf`:
```
csf -r
```

**Các lệnh cốt lõi hay dùng nhất**:
| Lệnh | Chức năng |
|---|---|
| `csf -e` / `csf -x` | Bật / Tắt CSF |
| `csf -r` | Restart rule tường lửa sau khi sửa cấu hình |
| `csf -a [IP]` | Cho phép 1 IP, ghi vào `csf.allow` |
| `csf -d [IP]` | Chặn 1 IP, ghi vào `csf.deny` |
| `csf -dr [IP]` | Gỡ chặn 1 IP khỏi `csf.deny` |
| `csf -g [IP]` | Tra cứu xem IP có đang bị chặn ở đâu (iptables/tempban) |
| `csf -t` | Xem danh sách IP đang bị chặn/cho phép tạm thời (kèm TTL) |


#### 3.5. apf

**Cài đặt:**
```
wget http://www.rfxn.com/downloads/apf-current.tar.gz
tar -xzf apf-current.tar.gz
cd apf-*
./install.sh
```

File cấu hình chính: `/etc/apf/conf.apf`. Quản lý qua lệnh `apf`:
```
apf -s      # start
apf -f      # flush/stop
apf -r      # restart
apf -a [IP] # allow IP
apf -d [IP] # deny IP
```

---

## II. Firewall trên Windows

### 1. Tổng quan

**Windows Defender Firewall** là thành phần bảo mật tích hợp sẵn trong Windows, chặn traffic trái phép. Khác với Linux dùng "Chains", Windows quản lý theo **Profiles**:
- **Domain Profile:** áp dụng khi máy kết nối vào mạng nội bộ có domain controller quản lý.
- **Private Profile:** mạng tin cậy (nhà riêng, văn phòng nhỏ).
- **Public Profile:** mạng công cộng (cafe, sân bay) — mức bảo mật cao nhất, chặn hầu hết kết nối vào.

### 2. Yêu cầu cơ bản

#### 2.1. Giao thức (Protocols)

Tương tự Linux: rule dựa trên TCP/UDP + port + địa chỉ IP nguồn/đích.

#### 2.2. Quyền quản trị (Administrative Privileges)

Cần chạy với quyền Administrator để tạo/sửa/xóa rule qua GUI (`wf.msc`) hoặc PowerShell.

### 3. Tìm hiểu: Cấu hình, sử dụng

**Truy cập GUI:** `Win + R` → gõ `wf.msc` → Enter.

**3 thành phần chính trong GUI:**
- **Inbound Rules:** kiểm soát kết nối từ ngoài vào server (VD: cho phép truy cập web port 80).
- **Outbound Rules:** kiểm soát ứng dụng trong server truy cập ra ngoài (VD: chặn phần mềm độc hại gửi dữ liệu ra).
- **Connection Security Rules:** thiết lập kết nối bảo mật IPsec giữa các máy.

**Tạo Inbound Rule (theo Port):**
```
Inbound Rules → New Rule → Rule Type: Port → chọn TCP/UDP + số port
→ Allow/Block the connection → chọn Profile (Domain/Private/Public) → đặt tên → Finish
```

**Tạo Outbound Rule (chặn 1 ứng dụng):**
```
Outbound Rules → New Rule → Rule Type: Program → Browse tới file .exe
→ Block the connection → đặt tên → Finish
```

**Cấu hình qua PowerShell** — cho phép tự động hóa hoàn toàn:
```powershell
# Xem trạng thái các profile
Get-NetFirewallProfile | Select Name, Enabled, DefaultInboundAction, DefaultOutboundAction

# Xem toàn bộ rule
Get-NetFirewallRule | Select DisplayName, Direction, Action, Enabled

# Tạo rule cho phép inbound theo port
New-NetFirewallRule -DisplayName "Allow HTTP Inbound" -Direction Inbound -Protocol TCP -LocalPort 80 -Action Allow -Profile Any -Enabled True

# Chặn 1 IP cụ thể
New-NetFirewallRule -DisplayName "Block Attacker" -Direction Inbound -RemoteAddress 1.2.3.4 -Action Block -Enabled True

# Xóa rule
Remove-NetFirewallRule -DisplayName "Allow HTTP Inbound"
```

---

## III. So sánh

Không có công cụ nào "tốt hơn" tuyệt đối — mỗi bên phù hợp với hệ sinh thái riêng, người vận hành hạ tầng hỗn hợp (mixed infra) cần thành thạo cả hai:

| Tiêu chí | Linux (iptables/ufw) | Windows Defender Firewall |
| :--- | :--- | :--- |
| Giao diện | CLI, file config | GUI (`wf.msc`) + PowerShell |
| Tính linh hoạt | Rất cao | Trung bình |
| Tự động hóa | Shell script, Ansible | PowerShell, Group Policy |
| Profiles/Zones | firewalld zones / ufw app | Domain / Private / Public |
| Logging | `/var/log/ufw.log`, iptables LOG | Event Viewer, WFP log |
| NAT/Routing | Có (`iptables -t nat`) | Hạn chế, cần RRAS |
| Chi phí | Miễn phí | Có phí (license Windows) |

---

## IV. Tìm hiểu về Tcpdump và Wireshark

### 1. Tổng quan

Tcpdump và Wireshark đều dùng để **bắt và phân tích gói tin mạng** (định dạng `.pcap`) — công cụ giám sát/debug mạng. Tcpdump hoạt động ở tầng 2, 3, 4 của mô hình OSI.

| Tiêu chí | TCPDump | Wireshark |
| :--- | :--- | :--- |
| Giao diện | Dòng lệnh (CLI) | Đồ họa (GUI) |
| Tài nguyên | Rất nhẹ | Nặng, cần nhiều tài nguyên |
| Dùng trên server | Rất tốt, không cần màn hình | Hạn chế, cần cài thêm X11 |
| Phân tích trực quan | Không, chỉ text | Rất tốt, có biểu đồ/màu sắc |
| Đọc file `.pcap` | Có hỗ trợ | Có hỗ trợ (mạnh hơn) |
| Cú pháp lọc | BPF | Display Filter chi tiết hơn |
| Phù hợp cho | Capture nhanh, automation, qua SSH | Phân tích sâu, trực quan |

### 2. Cài đặt, cấu hình

**Cài tcpdump:**
```
sudo apt install -y tcpdump
```
Tcpdump cần quyền root để bắt gói tin — 2 cách: dùng `sudo tcpdump`, hoặc gán user vào group `pcap`.

**Cú pháp cơ bản:**
```
tcpdump [options] [filter expression]
```

| Option | Ý nghĩa |
|---|---|
| `-i <interface>` | Chọn card mạng (`-i ens33`, `-i any` = tất cả) |
| `-n` / `-nn` | Không phân giải DNS / không phân giải cả port |
| `-v` / `-vvv` | Verbose / verbose nhất |
| `-c <number>` | Bắt N gói rồi dừng |
| `-w <file>` | Lưu ra file `.pcap` |
| `-r <file>` | Đọc từ file `.pcap` |
| `-A` | Hiển thị nội dung ASCII |
| `-X` / `-XX` | Hiển thị Hex + ASCII (kèm cả Ethernet header) |
| `-e` | Hiển thị thông tin Ethernet (MAC) |
| `-s <size>` | Kích thước snapshot; `-s 0` = bắt toàn bộ gói tin |
| `-q` | Quiet mode |

**Cài Wireshark/TShark:** trên Ubuntu Server (không GUI) dùng **TShark** — bản dòng lệnh của Wireshark; trên Ubuntu Desktop có thể cài thẳng **Wireshark** (GUI).
```
sudo apt install -y tshark      # bản CLI, dùng cho server
sudo apt install -y wireshark   # bản GUI, dùng cho desktop hoặc phân tích file .pcap đã copy về
sudo usermod -aG wireshark $USER
```

>  phổ biến: bắt gói tin trên server bằng `tcpdump`/`tshark` → lưu file `.pcap` → copy file về máy có GUI (`scp`) → mở phân tích bằng **Wireshark**. Toàn bộ các bước thực hành chi tiết (kèm ảnh) nằm trong file **Thực hành**.

---

## References

1. [Linux Firewall](https://www.sciencedirect.com/topics/computer-science/linux-firewall)
2. [Hướng dẫn mở chặn IP từ VPS bị chặn do đăng nhập sai — Nhân Hòa Wiki](https://wiki.nhanhoa.com/kb/huong-dan-mo-chan-ip-tu-vps-cua-khach-hang-bi-chan-do-dang-nhap-sai-qua-nhieu-lan-cuc-nhanh/)
