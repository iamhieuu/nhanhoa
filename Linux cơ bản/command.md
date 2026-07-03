# 3 – Lệnh thực hành, Text Editor, Process và Network

---

## Mục lục

- [1. Các lệnh thực hành cơ bản](#1-các-lệnh-thực-hành-cơ-bản)
- [2. Text Editor: vi và nano](#2-text-editor-vi-và-nano)
- [3. Process trong Linux](#3-process-trong-linux)
- [4. Network trong Linux](#4-network-trong-linux)

---

## 1. Các lệnh thực hành cơ bản

### 1.1 Điều hướng và làm việc với file

| Lệnh | Mô tả | Ví dụ |
|------|-------|-------|
| `pwd` | In ra đường dẫn thư mục hiện tại | `pwd` → `/home/hieu` |
| `ls` | Liệt kê nội dung thư mục | `ls -la` |
| `cd` | Chuyển thư mục | `cd /var/www` |
| `mkdir` | Tạo thư mục | `mkdir -p /opt/app/logs` |
| `rmdir` | Xóa thư mục rỗng | `rmdir empty_dir` |
| `touch` | Tạo file rỗng / cập nhật timestamp | `touch index.html` |
| `cp` | Copy file/thư mục | `cp -r /src /dst` |
| `mv` | Di chuyển hoặc đổi tên | `mv old.txt new.txt` |
| `rm` | Xóa file/thư mục | `rm -rf /tmp/test/` |
| `find` | Tìm kiếm file | `find / -name "*.log" -type f` |
| `locate` | Tìm file qua database index | `locate nginx.conf` |


**Các option `ls` hay dùng:**

```bash
ls -l       # Long format: quyền, owner, size, date
ls -a       # Hiển thị cả file ẩn (bắt đầu bằng .)
ls -la      # Kết hợp cả hai
ls -lh      # Size dạng human-readable (KB, MB, GB)
ls -lt      # Sắp xếp theo thời gian sửa đổi mới nhất
ls -R       # Liệt kê đệ quy toàn bộ thư mục con
```

**Xem nội dung file:**

```bash
cat file.txt            # In toàn bộ nội dung
cat -n file.txt         # In kèm số dòng
cat -A file.txt         # Hiển thị ký tự đặc biệt
less file.txt           # Xem từng trang, dùng phím q để thoát
more file.txt           # Xem từng trang (đơn giản hơn less)
head -20 file.txt       # Xem 20 dòng đầu
tail -20 file.txt       # Xem 20 dòng cuối
tail -f /var/log/syslog # Xem log thời gian thực (live follow)
```

**Xử lý nội dung file:**

```bash
# Ghi đè nội dung vào file (cẩn thận!)
echo "Hello World" > output.txt
cat > output.txt        # Nhập nội dung, kết thúc bằng Ctrl+D

# Nối thêm nội dung (append)
echo "New line" >> output.txt
cat >> output.txt

# Ghép nhiều file thành một
cat file1.txt file2.txt > combined.txt

# Đếm dòng, từ, ký tự
wc -l file.txt          # Số dòng
wc -w file.txt          # Số từ
wc -c file.txt          # Số byte

# Tìm kiếm nội dung trong file
grep "error" /var/log/syslog
grep -r "TODO" /var/www/     # Tìm đệ quy trong thư mục
grep -n "keyword" file.txt   # Hiện số dòng kết quả
grep -i "error" file.txt     # Không phân biệt hoa thường
```

### 1.2 Quản lý hệ thống

| Lệnh | Mô tả |
|------|-------|
| `sudo` | Chạy lệnh với quyền root |
| `su <user>` | Chuyển sang user khác |
| `reboot` | Khởi động lại hệ thống |
| `shutdown -h now` | Tắt máy ngay lập tức |
| `shutdown -r +5` | Khởi động lại sau 5 phút |
| `history` | Xem lịch sử lệnh đã chạy |
| `clear` | Xóa màn hình terminal |
| `date` | Xem ngày giờ hệ thống |
| `cal` | Hiển thị lịch tháng |
| `man <lệnh>` | Xem tài liệu hướng dẫn của lệnh |
| `which <lệnh>` | Tìm đường dẫn đầy đủ của lệnh |

### 1.3 Kiểm tra thông tin hệ thống

```bash
# Thông tin kernel và hệ điều hành
uname -a                    # Đầy đủ: kernel, hostname, kiến trúc
uname -r                    # Chỉ phiên bản kernel
lsb_release -a              # Thông tin distro (Ubuntu/Debian)
hostnamectl                 # Hostname và thông tin OS

# CPU
cat /proc/cpuinfo           # Thông tin chi tiết CPU
nproc                       # Số CPU core
lscpu                       # Thông tin CPU dễ đọc hơn

# RAM
free -h                     # RAM còn trống (human-readable)
cat /proc/meminfo           # Chi tiết RAM

# Disk
df -h                       # Dung lượng các phân vùng
du -sh /var/log/            # Dung lượng thư mục cụ thể
du -ah --max-depth=1 /var/  # Dung lượng thư mục con cấp 1
lsblk                       # Liệt kê block device (ổ đĩa, phân vùng)

# Thời gian chạy
uptime                      # Thời gian hệ thống đã chạy + load average
```

### 1.4 Quản lý gói và phần mềm

```bash
# APT (Ubuntu/Debian)
sudo apt update                     # Cập nhật danh sách gói
sudo apt install nginx -y           # Cài gói
sudo apt remove nginx               # Gỡ gói (giữ config)
sudo apt purge nginx                # Gỡ gói và xóa config
sudo apt autoremove                 # Xóa gói phụ thuộc không cần nữa
apt list --installed                # Liệt kê gói đã cài
apt show nginx                      # Thông tin chi tiết gói

# Tải file
wget https://example.com/file.tar.gz
curl -O https://example.com/file.tar.gz
curl -L https://example.com/ -o output.html
```

### 1.5 Nén và giải nén

| Lệnh | Mô tả | Ví dụ |
|------|-------|-------|
| `tar -cvf archive.tar dir/` | Nén thành `.tar` | `tar -cvf backup.tar /var/www` |
| `tar -xvf archive.tar` | Giải nén `.tar` | `tar -xvf backup.tar` |
| `tar -czvf archive.tar.gz dir/` | Nén thành `.tar.gz` | `tar -czvf backup.tar.gz /etc` |
| `tar -xzvf archive.tar.gz` | Giải nén `.tar.gz` | `tar -xzvf backup.tar.gz` |
| `tar -cjvf archive.tar.bz2 dir/` | Nén thành `.tar.bz2` | |
| `tar -xjvf archive.tar.bz2` | Giải nén `.tar.bz2` | |
| `zip -r archive.zip dir/` | Nén thành `.zip` | |
| `unzip archive.zip` | Giải nén `.zip` | |
| `gzip file.txt` | Nén thành `.gz` | |
| `gunzip file.txt.gz` | Giải nén `.gz` | |

**Ý nghĩa các cờ `tar`:**

| Cờ | Ý nghĩa |
|----|---------|
| `c` | Tạo archive mới |
| `x` | Giải nén |
| `v` | Verbose – hiển thị file đang xử lý |
| `f` | Chỉ định tên file archive |
| `z` | Dùng gzip để nén/giải nén |
| `j` | Dùng bzip2 để nén/giải nén |

---

## 2. Text Editor: vi và nano

### 2.1 vi / vim

`vi`  là editor mặc định trên mọi hệ thống Linux. `vim` (Vi IMproved) là phiên bản mở rộng với nhiều tính năng hơn.

**Cài vim:**
```bash
sudo apt install vim -y
```

**Di chuyển trong Normal Mode:**

```bash
h / ←    # Sang trái
l / →    # Sang phải
j / ↓    # Xuống
k / ↑    # Lên
0        # Về đầu dòng
$        # Về cuối dòng
gg       # Nhảy đến dòng đầu file
G        # Nhảy đến dòng cuối file
:n       # Nhảy đến dòng n (ví dụ: :50)
```

**Chỉnh sửa trong Normal Mode:**

```bash
i        # Vào Insert mode tại vị trí con trỏ
a        # Vào Insert mode sau con trỏ
o        # Thêm dòng mới bên dưới, vào Insert mode
O        # Thêm dòng mới bên trên, vào Insert mode
dd       # Xóa dòng hiện tại
ndd      # Xóa n dòng (ví dụ: 5dd)
yy       # Copy  dòng hiện tại
nyy      # Copy n dòng
p        # Paste sau con trỏ
P        # Paste trước con trỏ
u        # Undo
Ctrl+r   # Redo
```

**Tìm kiếm và thay thế:**

```bash
/keyword    # Tìm xuôi
?keyword    # Tìm ngược
n           # Kết quả tiếp theo
N           # Kết quả trước
:%s/old/new/g    # Thay thế toàn bộ trong file
:10,20s/old/new/g # Thay thế trong dòng 10-20
```

**Lưu và thoát:**

```bash
:w          # Lưu
:w!         # Lưu (buộc, kể cả file read-only)
:q          # Thoát
:q!         # Thoát không lưu
:wq         # Lưu và thoát
:x          # Lưu và thoát (tương tự :wq)
ZZ          # Lưu và thoát (phím tắt)
```

**Hiển thị số dòng:**
```bash
:set nu     # Hiển thị số dòng
:set nonu   # Ẩn số dòng
```


### 2.2 nano

`nano` là editor đơn giản hơn vi, thân thiện với người mới. Các phím tắt hiển thị ngay ở cuối màn hình.

**Cài nano:**
```bash
sudo apt install nano -y
```

**Mở file:**
```bash
nano filename.txt         # Mở file (tạo mới nếu chưa có)
nano +50 filename.txt     # Mở file và nhảy đến dòng 50
nano -l filename.txt      # Mở file với số dòng
```

**Phím tắt cơ bản (ký hiệu `^` = Ctrl):**

| Phím | Chức năng |
|------|-----------|
| `Ctrl+O` | Lưu file |
| `Ctrl+X` | Thoát |
| `Ctrl+W` | Tìm kiếm |
| `Ctrl+\` | Tìm và thay thế |
| `Ctrl+K` | Cắt dòng hiện tại |
| `Ctrl+U` | Paste |
| `Ctrl+G` | Xem trợ giúp |
| `Ctrl+C` | Xem vị trí con trỏ (dòng, cột) |
| `Ctrl+Shift+_` | Nhảy đến dòng cụ thể |
| `Alt+U` | Undo |
| `Alt+E` | Redo |


| Tiêu chí | vi/vim | nano |
|----------|--------|------|
| Độ khó | Cao | Thấp |
| Tốc độ  | Rất nhanh | Trung bình |
| Có sẵn trên mọi Linux | Luôn luôn | Thường có |

---

## 3. Process trong Linux

### 3.1 Khái niệm

**Process** là một chương trình đang được thực thi. Mỗi lần chạy lệnh trong terminal, Linux tạo ra một process mới.

**Thông tin gắn với mỗi process:**
- **PID (Process ID):** Số định danh duy nhất, tối đa 5 chữ số
- **PPID (Parent Process ID):** PID của process cha
- **Trạng thái hiện tại**
- **Tài nguyên đang dùng:** CPU, RAM, file handles


### 3.2 Các trạng thái process

| Ký hiệu | Trạng thái | Mô tả |
|---------|------------|-------|
| `R` | Running / Runnable | Đang chạy hoặc sẵn sàng chạy trong run queue |
| `S` | Interruptible Sleep | Đang chờ sự kiện, có thể bị ngắt bởi signal |
| `D` | Uninterruptible Sleep | Đang chờ I/O, không phản hồi signal (thường là disk/network I/O) |
| `T` | Stopped | Bị dừng bởi signal `SIGSTOP` (Ctrl+Z) |
| `Z` | Zombie | Đã kết thúc nhưng chưa được process cha dọn dẹp |


> **Zombie process** không gây hại nhưng chiếm PID. Nếu có nhiều zombie, có thể do process cha bị lỗi không gọi `wait()`. Giải pháp: kill process cha hoặc reboot nếu quá nhiều.

### 3.3 Foreground và Background

```bash
# Chạy process ở foreground (mặc định)
sleep 100

# Chạy process ở background
sleep 100 &

# Đưa process foreground xuống background
Ctrl+Z          # Tạm dừng (Stopped)
bg              # Tiếp tục chạy ở background

# Đưa process background lên foreground
fg              # Hoặc fg %job_number

# Xem danh sách jobs
jobs
```

### 3.4 Kiểm tra và giám sát process

#### Lệnh ps

```bash
ps              # Process của terminal hiện tại
ps aux          # Tất cả process trên hệ thống (định dạng BSD)
ps -ef          # Tất cả process (định dạng UNIX)
ps -u hieu      # Process của user hieu
ps -p 1234      # Thông tin process PID 1234

# Kết hợp với grep để tìm process cụ thể
ps aux | grep nginx
```

Giải thích cột `ps aux`:

| Cột | Ý nghĩa |
|-----|---------|
| `USER` | User sở hữu process |
| `PID` | Process ID |
| `%CPU` | % CPU đang dùng |
| `%MEM` | % RAM đang dùng |
| `VSZ` | Virtual memory size (KB) |
| `RSS` | RAM thực tế đang dùng (KB) |
| `STAT` | Trạng thái process |
| `START` | Thời điểm bắt đầu |
| `TIME` | Tổng CPU time đã dùng |
| `COMMAND` | Lệnh khởi động process |

#### Lệnh top

```bash
top             # Giám sát realtime, cập nhật mỗi 3 giây
top -d 1        # Cập nhật mỗi 1 giây
top -u hieu     # Chỉ hiển thị process của user hieu
```

**Phím tắt trong top:**

| Phím | Chức năng |
|------|-----------|
| `q` | Thoát |
| `k` | Kill process (nhập PID) |
| `r` | Renice (đổi priority) |
| `M` | Sắp xếp theo RAM |
| `P` | Sắp xếp theo CPU |
| `1` | Hiển thị chi tiết từng CPU core |

<img width="434" height="319" alt="image" src="https://github.com/user-attachments/assets/ad1657d7-ef7d-4ecd-a6f9-8f30ada79b26" />

#### Lệnh htop

```bash
sudo apt install htop -y
htop
```

`htop` là phiên bản nâng cao của `top` với giao diện màu, hỗ trợ dùng chuột, dễ đọc hơn.

#### Lệnh pgrep / pstree

```bash
# Tìm PID theo tên process
pgrep nginx
pgrep -u hieu bash      # PID của bash đang chạy bởi hieu

# Hiển thị cây tiến trình
pstree
pstree -p              # Kèm PID
pstree hieu            # Cây tiến trình của user hieu
```
<img width="260" height="340" alt="image" src="https://github.com/user-attachments/assets/0fc92fd7-86cc-478c-9eca-0cb5d568943c" />

### 3.5 Quản lý process

#### Dừng và kill process

```bash
# Gửi signal SIGTERM (yêu cầu dừng nhẹ nhàng)
kill 1234
kill -15 1234

# Gửi signal SIGKILL (buộc dừng, không thể ignore)
kill -9 1234

# Kill theo tên process
pkill nginx
killall nginx

# Tạm dừng process (SIGSTOP)
kill -STOP 1234
kill -CONT 1234    # Tiếp tục chạy
```

**Các signal phổ biến:**

| Signal | Số | Ý nghĩa |
|--------|-----|---------|
| `SIGHUP` | 1 | Reload cấu hình (không restart process) |
| `SIGINT` | 2 | Ngắt (tương đương Ctrl+C) |
| `SIGKILL` | 9 | Buộc kết thúc, không thể bắt/ignore |
| `SIGTERM` | 15 | Yêu cầu kết thúc nhẹ nhàng (mặc định) |
| `SIGSTOP` | 19 | Tạm dừng process |
| `SIGCONT` | 18 | Tiếp tục process đang bị dừng |


---

## 4. Network trong Linux

### 4.1 Các khái niệm cơ bản

- **Network Interface:** Đại diện phần mềm của card mạng. Ví dụ: `eth0`, `ens33`, `lo` (loopback)
- **IP Address:** Định danh thiết bị trong mạng (IPv4 và IPv6)
- **Routing:** Quá trình định tuyến gói tin giữa các mạng
- **Firewall:** Kiểm soát lưu lượng vào/ra theo quy tắc


### 4.2 Các file cấu hình mạng

#### `/etc/hosts`

Phân giải hostname cục bộ — ưu tiên hơn DNS:

```
127.0.0.1   localhost
::1         localhost
192.168.1.100   server01.local server01
```

```bash
# Xem và sửa hosts
cat /etc/hosts
sudo nano /etc/hosts
```


#### `/etc/resolv.conf`

Cấu hình DNS server:

```
nameserver 8.8.8.8
nameserver 1.1.1.1
search example.com
```


#### `/etc/netplan/*.yaml` (Ubuntu 22.04)

Ubuntu 22.04 dùng **Netplan** để cấu hình mạng:

```bash
# Xem file cấu hình
cat /etc/netplan/50-cloud-init.yaml
```

**Ví dụ cấu hình IP tĩnh:**

```yaml
network:
  version: 2
  ethernets:
    ens33:
      dhcp4: false
      addresses:
        - 192.168.1.100/24
      gateway4: 192.168.1.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 1.1.1.1
```

```bash
# Áp dụng cấu hình
sudo netplan apply
```


### 4.3 Lệnh quản trị mạng phổ biến

```bash
# Xem thông tin interface và IP
ip addr show               # Hoặc: ip a
ip addr show ens33         # Thông tin interface cụ thể
ifconfig                   # Lệnh cũ (cần cài net-tools)

# Kiểm tra kết nối
ping 8.8.8.8               # Ping theo IP
ping google.com            # Ping theo domain
ping -c 4 google.com       # Ping 4 lần rồi dừng

# DNS lookup
nslookup google.com        # Truy vấn DNS đơn giản
dig google.com             # Chi tiết hơn nslookup
dig @8.8.8.8 google.com    # Truy vấn DNS server cụ thể
host google.com            # Lookup nhanh

# Kiểm tra kết nối đến port
telnet 192.168.1.1 80      # Kết nối TCP đến port 80
nc -zv 192.168.1.1 80      # Netcat kiểm tra port
curl -I https://google.com # Kiểm tra HTTP response
```


### 4.4 Kiểm tra cổng và kết nối (ss / netstat)

```bash
# ss (thay thế netstat, nhanh hơn)
ss -tuln              # TCP/UDP đang lắng nghe
ss -tulnp             # Kèm tên process
ss -s                 # Thống kê tổng hợp

# netstat (cần cài net-tools)
sudo apt install net-tools -y
netstat -tuln         # TCP/UDP đang lắng nghe
netstat -tulnp        # Kèm tên process
netstat -an           # Tất cả kết nối

# Kiểm tra process đang dùng port cụ thể
sudo ss -tulnp | grep :80
sudo lsof -i :80
```

<img width="745" height="317" alt="image" src="https://github.com/user-attachments/assets/a27b9fb5-ac42-44c1-9c77-cb77f2ce81d7" />

### 4.5 Bảng định tuyến (Routing Table)

```bash
# Xem bảng định tuyến
ip route
ip route list
route -n          # Lệnh cũ

# Thêm route tạm thời (mất sau reboot)
sudo ip route add 192.168.2.0/24 via 192.168.1.1

# Xóa route
sudo ip route del 192.168.2.0/24

# Xem default gateway
ip route | grep default
```

**Ý nghĩa các cột:**

| Cột | Ý nghĩa |
|-----|---------|
| `Destination` | Mạng đích |
| `Gateway` | Router trung gian để đến đích |
| `Genmask` | Subnet mask |
| `Metric` | Độ ưu tiên của route (thấp hơn = ưu tiên hơn) |
| `Iface` | Interface sử dụng |

### 4.6 Firewall với iptables

**iptables** là công cụ quản lý firewall tích hợp trong Linux Kernel. Gồm 3 thành phần:

**Tables:**

| Table | Vai trò |
|-------|---------|
| `filter` | Quyết định chấp nhận hay từ chối gói tin (mặc định) |
| `nat` | Biên dịch địa chỉ NAT (Source NAT, Destination NAT) |
| `mangle` | Chỉnh sửa header gói tin (TTL, TOS...) |
| `raw` | Xử lý gói tin trước khi tracking trạng thái |

**Chains:**

| Chain | Xử lý khi nào |
|-------|--------------|
| `PREROUTING` | Gói tin vừa vào interface, trước khi routing |
| `INPUT` | Gói tin đến process trên máy này |
| `FORWARD` | Gói tin đi qua máy này (routing) |
| `OUTPUT` | Gói tin xuất phát từ process trên máy |
| `POSTROUTING` | Gói tin chuẩn bị rời interface |

**Targets:**

| Target | Hành động |
|--------|-----------|
| `ACCEPT` | Chấp nhận gói tin |
| `DROP` | Hủy gói tin, không phản hồi |
| `REJECT` | Hủy gói tin và gửi thông báo lỗi về |
| `LOG` | Ghi log rồi tiếp tục xử lý |

**Các lệnh iptables thường dùng:**

```bash
# Xem rules hiện tại
sudo iptables -L -v
sudo iptables -L INPUT -v --line-numbers

# Cho phép traffic loopback
sudo iptables -A INPUT -i lo -j ACCEPT

# Cho phép kết nối đã thiết lập
sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# Cho phép SSH (port 22)
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Cho phép HTTP/HTTPS
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Chặn IP cụ thể
sudo iptables -A INPUT -s 192.168.1.100 -j DROP

# Xóa rule theo số thứ tự
sudo iptables -D INPUT 4

# Xóa toàn bộ rules
sudo iptables -F
```

> **Trên Ubuntu 22.04**, `ufw` thân thiện hơn cho iptables:
> ```bash
> sudo ufw enable
> sudo ufw allow 22/tcp
> sudo ufw allow 80/tcp
> sudo ufw status verbose
> ```




- [Basic terminal commands for Ubuntu 22.04 – Medium](https://medium.com/@photon17/basic-terminal-command-lines-for-ubuntu-22-04-beginners-51d83f8a1132)
- [Getting Started With Ubuntu 22.04 LTS – ITU Online](https://www.ituonline.com/blogs/getting-started-with-ubuntu-22-04-lts-features-installation-and-tips/)
- [Linux Processes – Baeldung](https://www.baeldung.com/linux/processes)
