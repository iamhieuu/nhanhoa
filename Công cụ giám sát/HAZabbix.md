# Cấu hình High Availability (HA) cho Zabbix Server

***
## Mục lục

- [Cấu hình High Availability (HA) cho Zabbix Server](#cấu-hình-high-availability-ha-cho-zabbix-server)
  - [ Tổng quan](#tổng-quan)
    - [ HA là gì, vì sao cần](#ha-là-gì-vì-sao-cần)
    - [ Vai trò từng máy](#vai-trò-từng-máy)
    - [ Cơ chế hoạt động HA](#cơ-chế-hoạt-động-ha)
  - [Phần 1 — Cài đặt Zabbix Server trên máy backup (Node2)](#phần-1--cài-đặt-zabbix-server-trên-máy-backup-node2)
  - [Phần 2 — Cho phép Node2 kết nối vào Database của Node1](#phần-2--cho-phép-node2-kết-nối-vào-database-của-node1)
  - [Phần 3 — Cấu hình HA trên Node1](#phần-3--cấu-hình-ha-trên-node1)
  - [Phần 4 — Cấu hình HA trên Node2](#phần-4--cấu-hình-ha-trên-node2)
  - [Phần 5 — Kiểm tra HA hoạt động trên GUI](#phần-5--kiểm-tra-ha-hoạt-động-trên-gui)
  - [Phần 6 — Test Failover (Chuyển đổi dự phòng)](#phần-6--test-failover-chuyển-đổi-dự-phòng)
  - [Phần 7 — Cấu hình Web Frontend kết nối cả 2 Node](#phần-7--cấu-hình-web-frontend-kết-nối-cả-2-node)
  - [Phần 8 — Thêm Node2 vào Zabbix để giám sát](#phần-8--thêm-node2-vào-zabbix-để-giám-sát)

---

## Tổng quan

###  HA là gì, vì sao cần

Setup zabbix ban đầu chỉ có 1 Zabbix Server duy nhất — nếu máy đó down thì toàn bộ hệ thống giám sát ngưng luôn, không cảnh báo được gì trong lúc đó. HA (High Availability) từ Zabbix 6.0 trở lên giải quyết vấn đề này bằng cách cho phép chạy nhiều Zabbix Server node cùng trỏ vào 1 database, nhưng chỉ 1 node được active tại một thời điểm — node còn lại đứng standby, tự động lên thay khi node active chết.

### Vai trò từng máy

| VM | Hostname | IP | Vai trò |
|----|----------|----|---------|
| VM1 | zabbix-node1 | 192.168.136.131 | Zabbix Server Node 1 (Active) + MariaDB |
| VM2 | zabbix-node2 | 192.168.136.134 | Zabbix Server Node 2 (Standby) |
| VM3 | zabbix-agent01 | 192.168.136.146 | Zabbix Agent (host được giám sát) |

> Trong lab này, MariaDB chạy trên Node1 và cả 2 Zabbix Server node đều kết nối vào đó. Đây là cấu hình đơn giản cho mục đích học tập. Môi trường production cần Database HA riêng (Galera Cluster) — nếu không, DB trên Node1 vẫn là single point of failure dù đã có 2 node Zabbix Server.

### Cơ chế hoạt động HA

```
Trạng thái bình thường:

  [Node1 - ACTIVE]  ←── poll data, process triggers, send alerts
       │
       │ cập nhật heartbeat vào DB mỗi 5 giây
       ▼
  [MariaDB]  ←── [Node2 - STANDBY] đọc heartbeat của Node1

  Node2 không làm gì, chỉ theo dõi heartbeat của Node1

Khi Node1 bị down:

  [Node1 - DOWN]  ✗  heartbeat ngừng cập nhật
       │
       │ Sau 30 giây không thấy heartbeat
       ▼
  [Node2] phát hiện → tự promote thành ACTIVE
  [Node2 - ACTIVE]  ←── tiếp tục toàn bộ monitoring
```

Ghi chú: cơ chế HA của Zabbix không dùng VIP hay keepalived như HAProxy — 2 node "nói chuyện" với nhau qua bảng `ha_node` trong chính database dùng chung, node nào cập nhật heartbeat gần nhất thì được coi là active.

---

## Phần 1 — Cài đặt Zabbix Server trên máy backup (Node2)

### 1.1 Cập nhật hệ thống VM2

```bash
sudo apt update && sudo apt upgrade -y
```

### 1.2 Cài đặt Zabbix 7.0 repository trên VM2

```bash
# Tải Zabbix 7.0 LTS repository
wget https://repo.zabbix.com/zabbix/7.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_7.0-2+ubuntu22.04_all.deb

# Cài đặt repository
sudo dpkg -i zabbix-release_7.0-2+ubuntu22.04_all.deb

# Cập nhật package list
sudo apt update
```

### 1.3 Cài đặt Zabbix Server trên VM2

```bash
# Cài Zabbix Server (dùng MySQL vì dùng chung MariaDB với Node1)
# Không cài frontend, không cài MariaDB (dùng chung DB của Node1)
sudo apt install -y \
  zabbix-server-mysql \
  zabbix-agent2

# Xác nhận đã cài
sudo dpkg -l | grep zabbix | awk '{print $2, $3}'
```

<img width="481" height="74" alt="image" src="https://github.com/user-attachments/assets/ee2f7be3-1365-41d9-873a-55f0b423b15d" />

Ghi chú: Node2 không cài MariaDB riêng, chỉ cài Zabbix Server + Agent2 rồi trỏ config DB sang IP của Node1 (làm ở Phần 4).

---

## Phần 2 — Cho phép Node2 kết nối vào Database của Node1

> Thực hiện trên **VM1** (192.168.136.131) — nơi có MariaDB.

### 2.1 Tạo user cho Node2 kết nối từ xa

```bash
# Đăng nhập MariaDB trên VM1
sudo mysql -u root -p'123456a@'
```

Chạy các lệnh SQL trong MariaDB prompt:

```sql
-- Tạo user cho phép Node2 kết nối từ IP của Node2
CREATE USER 'zabbix'@'192.168.136.145' IDENTIFIED BY 'Zabbix@DB2026!';

GRANT ALL PRIVILEGES ON zabbix.* TO 'zabbix'@'192.168.136.145';

FLUSH PRIVILEGES;

EXIT;
```

Kiểm tra user đã tạo:

```bash
sudo mysql -u root -p'123456a@' \
  -e "SELECT user, host FROM mysql.user WHERE user='zabbix';"
```

<img width="395" height="114" alt="image" src="https://github.com/user-attachments/assets/992b0ebf-b812-4f97-b79a-c611b7c0d2da" />

Ghi chú: MySQL/MariaDB coi `'zabbix'@'localhost'` và `'zabbix'@'192.168.136.145'` là 2 user khác nhau dù trùng username — phải tạo riêng user gắn với IP của Node2 thì Node2 mới login được từ xa.

### 2.2 Cho phép MariaDB lắng nghe từ bên ngoài

Mặc định MariaDB chỉ lắng nghe trên `127.0.0.1`. Cần cho phép Node2 kết nối vào:

```bash
# Trên VM1
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
```

Tìm dòng `bind-address` và sửa:

```ini
# Tìm dòng này:
bind-address = 127.0.0.1

# Sửa thành (lắng nghe tất cả interface):
bind-address = 0.0.0.0
```

Lưu file và restart MariaDB:

```bash
sudo systemctl restart mariadb

# Kiểm tra MariaDB đang lắng nghe port 3306
sudo ss -tlnp | grep 3306
```

<img width="665" height="60" alt="image" src="https://github.com/user-attachments/assets/cd50cd30-5a3d-48c1-98da-6bce3560c261" />

### 2.3 Mở firewall port 3306 trên VM1

```bash
# Trên VM1 — cho phép Node2 kết nối vào MariaDB
sudo ufw allow from 192.168.136.145 to any port 3306 comment 'Zabbix Node2 to DB'

sudo ufw reload
sudo ufw status
```

<img width="572" height="184" alt="image" src="https://github.com/user-attachments/assets/7bd1fa14-4db3-479c-b312-4bdffd8b5f75" />

Ghi chú: chỉ mở port 3306 cho đúng IP của Node2 (`from 192.168.136.145`), không mở toàn mạng — DB là chỗ nhạy cảm nhất trong hệ thống HA này.

### 2.4 Kiểm tra kết nối từ Node2 vào DB của Node1

```bash
# Trên VM2 — test kết nối vào MariaDB của VM1
mysql -h 192.168.136.131 -u zabbix -p'Zabbix@DB2026!' \
  -e "SELECT COUNT(*) AS hosts FROM zabbix.hosts;"
```

<img width="921" height="197" alt="image" src="https://github.com/user-attachments/assets/bd52c7c5-4472-472b-a626-5d84967808c3" />

---

## Phần 3 — Cấu hình HA trên Node1

> Thực hiện trên **VM1** (192.168.136.131).

### 3.1 Chỉnh sửa cấu hình Zabbix Server Node1

```bash
# Backup file cấu hình hiện tại
sudo cp /etc/zabbix/zabbix_server.conf /etc/zabbix/zabbix_server.conf.bak_ha

# Mở file cấu hình
sudo nano /etc/zabbix/zabbix_server.conf
```

Tìm và chỉnh sửa (dùng `Ctrl+W` để tìm trong nano):

```ini
# Tìm dòng: # HANodeName=
# Bỏ dấu # và điền tên node:
HANodeName=zabbix-node1

# Tìm dòng: # NodeAddress=
# Bỏ dấu # và điền địa chỉ của node này:
NodeAddress=192.168.136.131:10051
```

<img width="291" height="182" alt="image" src="https://github.com/user-attachments/assets/73a68aae-8119-4e3b-b004-c58c326e5911" />

Ghi chú: `NodeAddress` là địa chỉ để node còn lại (hoặc Frontend) biết gọi tới node này ở đâu — khác với `HANodeName` chỉ là cái tên định danh trong bảng `ha_node`.

### 3.2 Restart Zabbix Server Node1

```bash
sudo systemctl restart zabbix-server

# Kiểm tra không có lỗi
sudo systemctl status zabbix-server
```

Kiểm tra log — Node1 phải khởi động ở chế độ **active**:

```bash
sudo tail -20 /var/log/zabbix/zabbix_server.log | grep -i "ha\|active\|standby\|node"
```

Kết quả mong đợi:

<img width="698" height="52" alt="image" src="https://github.com/user-attachments/assets/394b8e91-075a-4798-9fbf-c8e1402736c0" />

---

## Phần 4 — Cấu hình HA trên Node2

> Thực hiện trên **VM2** (192.168.136.145).

### 4.1 Chỉnh sửa cấu hình Zabbix Server Node2

```bash
# Backup
sudo cp /etc/zabbix/zabbix_server.conf /etc/zabbix/zabbix_server.conf.bak

# Mở file cấu hình
sudo nano /etc/zabbix/zabbix_server.conf
```

Tìm và sửa các dòng sau:

**Phần Database — trỏ vào DB của Node1:**

```ini
# Tìm dòng: DBHost=localhost
# Sửa thành IP của Node1 (nơi có MariaDB):
DBHost=192.168.136.131

# Tìm dòng: # DBPassword=
# Sửa thành:
DBPassword=Zabbix@DB2026!

# Tìm dòng: DBSocket=
# Comment lại dòng này (Node2 kết nối qua TCP/IP không phải socket)
# DBSocket=/var/run/mysqld/mysqld.sock
```

**Phần HA:**

```ini
# Tìm dòng: # HANodeName=
# Sửa thành:
HANodeName=zabbix-node2

# Tìm dòng: # NodeAddress=
# Sửa thành:
NodeAddress=192.168.136.145:10051
```

Ghi chú: điểm khác biệt quan trọng nhất giữa config Node1 và Node2 là `DBHost` — Node1 dùng `localhost`/socket vì DB nằm ngay trên máy, Node2 phải trỏ IP + bỏ socket vì DB nằm trên máy khác.

### 4.2 Cấu hình Zabbix Agent2 trên Node2 (tự giám sát)

```bash
sudo nano /etc/zabbix/zabbix_agent2.conf
```

Tìm và sửa:

```ini
Server=192.168.136.131

ServerActive=192.168.136.131

Hostname=zabbix-node2
```

Lưu file.

### 4.3 Khởi động dịch vụ trên Node2

```bash
# Khởi động Zabbix Server
sudo systemctl start zabbix-server
sudo systemctl enable zabbix-server

# Khởi động Agent2
sudo systemctl start zabbix-agent2
sudo systemctl enable zabbix-agent2

# Kiểm tra trạng thái
sudo systemctl status zabbix-server
sudo systemctl status zabbix-agent2
```

Kiểm tra log — Node2 phải khởi động ở chế độ **standby**:

```bash
sudo tail -20 /var/log/zabbix/zabbix_server.log | grep -i "ha\|active\|standby\|node"
```

<img width="673" height="101" alt="image" src="https://github.com/user-attachments/assets/28649b2b-8348-4171-8a7e-e771412b91a9" />

### 4.4 Mở firewall trên Node2

```bash
sudo ufw allow 10051/tcp comment 'Zabbix Server trapper'
sudo ufw allow 10050/tcp comment 'Zabbix Agent'
sudo ufw allow 22/tcp    comment 'SSH'

sudo ufw reload
sudo ufw status
```

<img width="570" height="276" alt="image" src="https://github.com/user-attachments/assets/e87e4727-a5bb-4c7d-a90d-617d2ccf896b" />

---

## Phần 5 — Kiểm tra HA hoạt động trên GUI

### 5.1 Đăng nhập Zabbix Frontend

```
http://192.168.136.131/
```

Đăng nhập với tài khoản Admin.

### 5.2 Xem trạng thái HA Nodes

Trên menu bên trái, vào:

```
Reports → System information → High availability ở cuối
```

Màn hình hiển thị danh sách nodes:

<img width="959" height="437" alt="image" src="https://github.com/user-attachments/assets/f8e065b3-734e-42bb-aa8d-9893881c11df" />

### 5.3 Kiểm tra qua lệnh trực tiếp trên database

```bash
# Trên VM1
sudo mysql -u root -p'123456a@' zabbix -e "
SELECT name,
       status,
       FROM_UNIXTIME(lastaccess) AS last_seen
FROM ha_node
ORDER BY status;"
```

<img width="440" height="201" alt="image" src="https://github.com/user-attachments/assets/28f8c808-61d6-4a56-ab2d-7489feb0f690" />

Ghi chú: bảng `ha_node` chính là "nguồn sự thật" của cơ chế HA — GUI cũng chỉ đọc từ bảng này ra để hiển thị thôi, nên nếu GUI báo sai thì query thẳng bảng này để đối chiếu.

---

## Phần 6 — Test Failover (Chuyển đổi dự phòng)

Đây là bước quan trọng nhất — kiểm tra HA thực sự hoạt động.

### 6.1 Chuẩn bị — mở 2 terminal song song

**Terminal 1** — Theo dõi log Node2 (VM2):

```bash
# SSH vào VM2
ssh apache1@192.168.136.145

# Theo dõi log real-time
sudo tail -f /var/log/zabbix/zabbix_server.log | grep -i "ha\|active\|standby\|failover\|node"
```

**Terminal 2** — Thực hiện test trên Node1 (VM1):

```bash
# SSH vào VM1
ssh iamhieu@192.168.136.131
```

### 6.2 Dừng Zabbix Server trên Node1 (giả lập sự cố)

Trên **Terminal 2 (VM1)**:

```bash
# Dừng Zabbix Server Node1
sudo systemctl stop zabbix-server

# Xác nhận đã dừng
sudo systemctl status zabbix-server
# Phải thấy: Active: inactive (dead)
```

### 6.3 Quan sát Node2 tự động promote

Sau khoảng **30 giây**, sẽ thấy log:

<img width="776" height="106" alt="image" src="https://github.com/user-attachments/assets/ec93e7de-1d34-470b-88b8-68e31beb19a1" />

### 6.4 Xác nhận Node2 đã trở thành Active

Kiểm tra trên GUI (`Administration → High availability`):

<img width="950" height="438" alt="image" src="https://github.com/user-attachments/assets/630d47a0-9414-41b6-ae06-125f2f099c54" />

Kiểm tra monitoring vẫn tiếp tục — vào `Monitoring → Latest data` trên GUI, dữ liệu vẫn cập nhật bình thường.

### 6.5 Khôi phục Node1 và quan sát

Trên **Terminal 2 (VM1)**:

```bash
# Khởi động lại Zabbix Server Node1
sudo systemctl start zabbix-server
```

Sau vài giây, kiểm tra log Node1:

```bash
sudo tail -20 /var/log/zabbix/zabbix_server.log | grep -i "ha\|active\|standby"
```

<img width="648" height="81" alt="image" src="https://github.com/user-attachments/assets/417baa72-afde-4f86-803b-7dc5277481df" />

Node1 **tự nhận biết Node2 đang active** và tự chuyển sang **standby** — không xảy ra xung đột.

Kiểm tra GUI:

<img width="857" height="78" alt="image" src="https://github.com/user-attachments/assets/41aaf88e-c916-4062-a956-ae94ac2b5739" />

Ghi chú: Zabbix HA mặc định **không tự failback** — tức là Node2 đang active thì cứ giữ active luôn, Node1 lên lại chỉ đứng standby chứ không giành lại quyền active. Muốn Node1 active trở lại phải chủ động stop Node2 (hoặc gọi API/CLI để chuyển).

---

## Phần 7 — Cấu hình Web Frontend kết nối cả 2 Node

Mặc định, Web Frontend đang cấu hình kết nối đến `127.0.0.1:10051` (chỉ Node1). Cần cho phép Frontend biết về cả 2 nodes để khi Node1 down, Frontend vẫn hoạt động.

### 7.1 Sửa file cấu hình Frontend

```bash
# Trên VM1
sudo nano /etc/zabbix/web/zabbix.conf.php
```

Tìm các dòng liên quan đến server:

```php
// Tìm:
$ZBX_SERVER      = 'localhost';
$ZBX_SERVER_PORT = '10051';

// Sửa thành (kết nối đến Node1, dùng IP thay vì localhost):
$ZBX_SERVER      = '192.168.136.131';
$ZBX_SERVER_PORT = '10051';
```

Lưu file.

### 7.2 Cài đặt Web Frontend trên Node2 (tuỳ chọn — để Frontend HA)

Nếu muốn Web Frontend cũng dự phòng (không bắt buộc cho lab cơ bản):

```bash
# Trên VM2
sudo apt install -y \
  zabbix-frontend-php \
  zabbix-nginx-conf

# Cấu hình Nginx
sudo nano /etc/zabbix/nginx.conf
```

Sửa:

```nginx
listen          80;
server_name     192.168.136.145;
```

```bash
sudo rm -f /etc/nginx/sites-enabled/default

# Cấu hình PHP timezone
sudo nano /etc/zabbix/php-fpm.conf
# Sửa: php_value[date.timezone] = Asia/Ho_Chi_Minh
```

Tạo file cấu hình Frontend:

```bash
sudo cp /etc/zabbix/web/zabbix.conf.php.example \
        /etc/zabbix/web/zabbix.conf.php 2>/dev/null || true

sudo nano /etc/zabbix/web/zabbix.conf.php
```

Điền thông tin kết nối database và server:

```php
<?php
$DB['TYPE']               = 'MYSQL';
$DB['SERVER']             = '192.168.136.131';
$DB['PORT']               = '3306';
$DB['DATABASE']           = 'zabbix';
$DB['USER']               = 'zabbix';
$DB['PASSWORD']           = 'Zabbix@DB2026!';
$DB['SCHEMA']             = '';
$DB['ENCRYPTION']         = false;
$DB['KEY_FILE']           = '';
$DB['CERT_FILE']          = '';
$DB['CA_FILE']            = '';
$DB['VERIFY_HOST']        = false;
$DB['CIPHER_LIST']        = '';
$DB['VAULT_URL']          = '';
$DB['VAULT_DB_PATH']      = '';
$DB['VAULT_TOKEN']        = '';
$DB['VAULT_CERT_FILE']    = '';
$DB['VAULT_KEY_FILE']     = '';
$ZBX_SERVER               = '192.168.136.145';
$ZBX_SERVER_PORT          = '10051';
$ZBX_SERVER_NAME          = 'My Zabbix Lab';
$IMAGE_FORMAT_DEFAULT     = IMAGE_FORMAT_PNG;
```

```bash
sudo systemctl start nginx php8.1-fpm
sudo systemctl enable nginx php8.1-fpm
```

Sau bước này, truy cập Frontend qua cả 2 địa chỉ:

- `http://192.168.136.131/` — Node1
- `http://192.168.136.145/` — Node2

---

## Phần 8 — Thêm Node2 vào Zabbix để giám sát

### 8.1 Thêm Host zabbix-node2 vào Zabbix Frontend

Đăng nhập vào GUI → `Data collection → Hosts → Create host`

**Tab: Host**

```
Host name:     zabbix-node2
Visible name:  Zabbix Server Node2 (Standby)
Templates:     Linux by Zabbix agent 2
Host groups:   Zabbix servers
```

**Interfaces:**

```
Loại:     Agent
IP:       192.168.136.145
Port:     10050
```

<img width="590" height="338" alt="image" src="https://github.com/user-attachments/assets/8f123878-32d7-4dc8-8c44-504d5d12871d" />

### 8.2 Kiểm tra Agent Node2 kết nối

Sau 1–2 phút, vào `Data collection → Hosts` → kiểm tra cột **Availability** của `zabbix-node2`.

Kiểm tra bằng lệnh từ Node1:

```bash
# Trên VM1
sudo zabbix_get -s 192.168.136.145 -p 10050 -k "agent.ping"
```

<img width="563" height="41" alt="image" src="https://github.com/user-attachments/assets/565ae96a-bc01-4f73-aef0-09dbd49cd2db" />

<img width="959" height="470" alt="image" src="https://github.com/user-attachments/assets/cc6cd804-d731-4675-860f-f98ce01cd2bd" />
