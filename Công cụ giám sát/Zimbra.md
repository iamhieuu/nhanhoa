## Triển khai và Sử dụng Zimbra trên Ubuntu 22.04
***
## Table of Contents

- [Triển khai và Sử dụng Zimbra trên Ubuntu 22.04](#triển-khai-và-sử-dụng-zimbra-trên-ubuntu-2204)
  - [1. Tổng quan Zimbra](#1-tổng-quan-zimbra)
  - [2. Chuẩn bị hệ thống trước khi cài đặt](#2-chuẩn-bị-hệ-thống-trước-khi-cài-đặt)
    - [2.1 Tắt auto update](#21-tắt-auto-update)
    - [2.2 Cài đặt IP tĩnh](#22-cài-đặt-ip-tĩnh)
    - [2.3 Cấu hình hostname và /etc/hosts](#23-cấu-hình-hostname-và-etchosts)
    - [2.4 Tắt IPv6](#24-tắt-ipv6)
    - [2.5 Cấu hình firewall](#25-cấu-hình-firewall)
  - [3. Cấu hình DNS Server](#3-cấu-hình-dns-server)
    - [3.1 Vì sao Zimbra cần DNS nội bộ](#31-vì-sao-zimbra-cần-dns-nội-bộ)
    - [3.2 Phương án BIND9 (khuyến nghị Production)](#32-phương-án-bind9-khuyến-nghị-production)
    - [3.3 Phương án dnsmasq (đơn giản, dùng cho LAB)](#33-phương-án-dnsmasq-đơn-giản-dùng-cho-lab)
  - [4. Cài đặt Zimbra](#4-cài-đặt-zimbra)
    - [4.1 Tải source Zimbra](#41-tải-source-zimbra)
    - [4.2 Giải nén và chạy script cài đặt](#42-giải-nén-và-chạy-script-cài-đặt)
    - [4.3 Cấu hình trong quá trình cài đặt](#43-cấu-hình-trong-quá-trình-cài-đặt)
    - [4.4 Kiểm tra sau khi cài đặt](#44-kiểm-tra-sau-khi-cài-đặt)
    - [4.5 Kích hoạt license](#45-kích-hoạt-license)
  - [5. Sử dụng và Quản trị Zimbra](#5-sử-dụng-và-quản-trị-zimbra)
    - [5.1 Tạo user mới](#51-tạo-user-mới)
    - [5.2 Thiết lập chính sách mật khẩu](#52-thiết-lập-chính-sách-mật-khẩu)
    - [5.3 Thiết lập chữ ký và Forward email](#53-thiết-lập-chữ-ký-và-forward-email)
    - [5.4 Tìm ID Mailbox và chỉnh sửa Quota](#54-tìm-id-mailbox-và-chỉnh-sửa-quota)
    - [5.5 Đổi mật khẩu account admin](#55-đổi-mật-khẩu-account-admin)
    - [5.6 Kiểm tra Log gửi/nhận email](#56-kiểm-tra-log-gửinhận-email)
    - [5.7 Thay đổi logo giao diện](#57-thay-đổi-logo-giao-diện)
    - [5.8 Thay đổi title web (webmail/webadmin)](#58-thay-đổi-title-web-webmailwebadmin)
    - [5.9 Backup và Restore](#59-backup-và-restore)
    - [5.10 Chuyển data sang node khác (Migration)](#510-chuyển-data-sang-node-khác-migration)
  - [References](#references)

---

## 1. Tổng quan Zimbra

Zimbra hay còn gọi Zimbra Collaboration Suite là một bộ giải pháp email được cài trên máy chủ riêng. Tại đây cung cấp một hệ thống email theo tên miền riêng, webmail, lịch làm việc dành cho doanh nghiệp. Zimbra được phát triển bởi LiquidSys vào năm 2005, sau này được đổi tên thành Zimbra.

Zimbra cung cấp trải nghiệm cùng giao diện webmail với những tính năng phong phú, sáng tạo cho người dùng đầu cuối. Với Zimbra, bạn có thể gửi và nhận mail, chia sẻ, quản lý tài liệu trên thiết bị di động và đồng bộ hóa máy tính để bàn. Ngoài ra Zimbra còn là ứng dụng nguồn mở miễn phí nổi tiếng với tính năng phong phú, độ ổn định và bảo mật cao.

**Thông tin LAB sử dụng trong tài liệu này:**

| Thông số | Giá trị |
|---|---|
| Hệ điều hành | Ubuntu 22.04 |
| Domain | anthanh264.com |
| Hostname | mail.anthanh264.com |
| Địa chỉ IP | 192.168.57.134 |

---

## 2. Chuẩn bị hệ thống trước khi cài đặt

Trước khi cài đặt Zimbra, hệ thống cần được chuẩn bị đầy đủ: tắt cập nhật tự động (tránh xung đột gói trong lúc cài), gán IP tĩnh, khai báo hostname chuẩn FQDN, tắt IPv6 (Zimbra khuyến nghị chạy thuần IPv4 để tránh lỗi phân giải), và mở các port cần thiết trên firewall.

### 2.1 Tắt auto update

<img width="605" height="70" alt="image" src="https://github.com/user-attachments/assets/0a6c0f03-a3e5-4020-8581-8977903646c3" />

### 2.2 Cài đặt IP tĩnh

<img width="596" height="284" alt="image" src="https://github.com/user-attachments/assets/746d2e9d-1f2d-4ca8-bdfe-86dcc2af0f7d" />

### 2.3 Cấu hình hostname và /etc/hosts

<img width="545" height="203" alt="image" src="https://github.com/user-attachments/assets/9fc1450c-5821-4ade-a273-5ed835a05448" />

### 2.4 Tắt IPv6

<img width="527" height="369" alt="image" src="https://github.com/user-attachments/assets/9fdc049d-5ad5-4d2e-ab96-cfd5d0e7fcfc" />

<img width="730" height="375" alt="image" src="https://github.com/user-attachments/assets/731ef4cd-487f-40d8-b8a9-8acf7115de2a" />

### 2.5 Cấu hình firewall

<img width="505" height="397" alt="image" src="https://github.com/user-attachments/assets/01aab918-ceb5-47e0-969c-c8240a379bac" />

---

## 3. Cấu hình DNS Server

### 3.1 Vì sao Zimbra cần DNS nội bộ

Zimbra khuyến nghị sử dụng caching named server trong trường hợp bạn triển khai split domain (hệ thống nằm sau firewall/nat), Zimbra cần MX và A record để cài đặt và vận hành. Ngoài ra spamassassin thực hiện rất nhiều dns query đến các cơ sở dữ liệu blacklist nên việc sử dụng trực tiếp các DNS server phổ biến (google, opendns,…) sẽ dẫn đến việc bị hạn chế quota query.

Spamassassin khuyến cáo sử dụng non-forwarding caching DNS servers vì thế ta nên sử dụng BIND hoặc UNBOUND DNS server trong môi trường production. Bạn có thể sử dụng dnsmasq trong môi trường LAB.

LAB có:
- Domain là `anthanh264.com`
- Tên server là `mail.anthanh264.com`
- Địa chỉ IP là `192.168.57.134`

### 3.2 Phương án BIND9 (khuyến nghị Production)

**Bước 1 – Cài đặt các gói cần thiết**

```bash
sudo apt-get install bind9 bind9utils resolvconf net-tools -y
```
![image](./images/zim-cf0.png)

**Bước 2 – Set hostname**

```bash
hostnamectl set-hostname mail.anthanh264.com
```
![image](./images/zimb-1.png)

**Bước 3 – Chỉnh sửa file hosts, thêm dòng**

```bash
nano /etc/hosts
```
```
192.168.57.134 anthanh264.com     
192.168.57.134 mail.anthanh264.com     mail
```
![image](./images/zimb-2.png)

**Bước 4 – Cấu hình resolv.conf**

```bash
systemctl enable resolvconf
systemctl start resolvconf
cp /etc/resolv.conf /etc/resolv.conf.conf.backup
echo "search anthanh264.com" >> /etc/resolvconf/resolv.conf.d/head
echo "nameserver 192.168.57.134" >> /etc/resolvconf/resolv.conf.d/head
sudo resolvconf --enable-updates
sudo resolvconf -u
```
![image](./images/zimb-3.png)

**Bước 5 – Cấu hình DNS server local**

- Tạo Zone DNS bind

```bash
cp /etc/bind/named.conf.local /etc/bind/named.conf.local.backup
cp /etc/bind/named.conf.options /etc/bind/named.conf.options.backup
> /etc/bind/named.conf.local
sed -i '/directory*/a\        forwarders {8.8.8.8; 8.8.4.4;};' /etc/bind/named.conf.options;
```
![image](./images/zimb-4.png)
![image](./images/zimb-5.png)

- Tạo file `/etc/bind/named.conf.local` và thêm nội dung

```bash
nano /etc/bind/named.conf.local
```
```
zone  anthanh264.com {
        type master;
                file "/var/lib/bind/anthanh264.com.hosts";
        allow-transfer {
                127.0.0.1;
                localnets;
                };
        };
```
![image](./images/zimb-6.png)

- Tạo file `/var/lib/bind/anthanh264.com.hosts` và thêm nội dung

```
$ttl 3600
@      IN      SOA     mail.anthanh264.com. root.mail.anthanh264.com. (
                        1615364925
                        3600
                        600
                        1209600
                        3600 )
anthanh264.com.       IN      NS      mail.anthanh264.com.
mail.anthanh264.com.  IN      A       192.168.57.134
anthanh264.com.       IN      MX      10 mail
```
![image](./images/zimb-7.png)

- Khởi động lại `named` để apply

```bash
systemctl restart named
systemctl enable named
```
![image](./images/zimb-8.png)

**Bước 6 – Kiểm tra**

```bash
nslookup mail.anthanh264.com
```
Kết quả như trong ảnh là thành công:
![image](./images/zimb-9.png)

### 3.3 Phương án dnsmasq (đơn giản, dùng cho LAB)

Với môi trường LAB đơn giản, không cần zone file phức tạp như BIND, có thể dùng `dnsmasq` để cấu hình DNS nội bộ nhanh cho hostname phân giải đúng trước khi cài Zimbra.

<img width="628" height="374" alt="image" src="https://github.com/user-attachments/assets/b56cae9f-9247-4cf3-994a-2734eb37111f" />

> **Lưu ý:** Chỉ chọn một trong hai phương án (BIND hoặc dnsmasq) cho một server. BIND phù hợp production nhờ khả năng làm caching non-forwarding DNS server đúng khuyến nghị của Zimbra/Spamassassin; dnsmasq phù hợp LAB/test nhanh.

---

## 4. Cài đặt Zimbra

### 4.1 Tải source Zimbra

```bash
wget https://files.zimbra.com/downloads/10.1.0_GA/zcs-NETWORK-10.1.0_GA_4655.UBUNTU22_64.20240819064312.tgz
```
![image](./images/zim-cf5.png)

### 4.2 Giải nén và chạy script cài đặt

```bash
tar xzvf zcs-NETWORK-10.1.0_GA_4655.UBUNTU22_64.20240819064312.tgz
cd zcs-NETWORK-10.1.0_GA_4655.UBUNTU22_64.20240819064312
./install.sh
```
![image](./images/zim-cf6.png)
![image](./images/zim-cf7.png)

Ngoài ra có thể tham khảo bước cài đặt gói zimbra thực tế trên Ubuntu 22.04:

<img width="893" height="183" alt="image" src="https://github.com/user-attachments/assets/d5b7f4af-b170-480e-abbe-3ac1ce0a0f6d" />

Giao diện sau khi cài:

<img width="627" height="394" alt="image" src="https://github.com/user-attachments/assets/c5e2753f-f1cb-45d3-a012-e04ba01cf1f0" />

### 4.3 Cấu hình trong quá trình cài đặt

- Chọn tính năng cài theo chỉ dẫn của script
![image](./images/zim-cf10.png)
- Các cấu hình cơ bản zimbra
![image](./images/zim-cf11.png)
- Cấu hình mật khẩu admin: Chọn `6` và `4` để tới dòng cấu hình mật khẩu
![image](./images/zimb-12.png)
![image](./images/zimb-13.png)

Minh họa thao tác cài đặt mật khẩu cho Zimbra:

<img width="569" height="398" alt="image" src="https://github.com/user-attachments/assets/cb7ae056-7c1f-4bb7-9365-33f2bf3048ca" />

- Cấu hình trạng thái kích hoạt license: Chọn tiếp `32` rồi chọn `2` để cấu hình activate sau khi cài.
![image](./images/zimb-14.png)
- Chọn `r` để về menu chính, `a` để apply cấu hình, `yes` để thực hiện cài đặt zimbra
![image](./images/zimb-15.png)

Lưu cấu hình và chạy:

<img width="641" height="397" alt="image" src="https://github.com/user-attachments/assets/e3f63539-ebe9-4244-9023-47d4e25c55b4" />

### 4.4 Kiểm tra sau khi cài đặt

- Sau khi có thông báo này là quá trình cài đặt hoàn tất
![image](./images/zimb-16.png)

Giao diện thành công:

<img width="859" height="319" alt="image" src="https://github.com/user-attachments/assets/d688159c-e7ed-4abd-b296-e62e1deb5ea6" />

- Truy cập giao diện quản lý Zimbra tại `https://192.168.57.134:7071/zimbraAdmin/`
![image](./images/zimb-17.png)

### 4.5 Kích hoạt license

- Tiến hành kích hoạt trial license
![image](./images/zimbra-key.png)
  - Key lấy từ mail nhận được sau khi submit form tại [Zimbra Trial-license](https://www.zimbra.com/connect/forms/?form=trial-license)
  ![image](./images/zimb-18.png)
- Kích hoạt thành công license
![image](./images/zimbra-activate.png)

---

## 5. Sử dụng và Quản trị Zimbra

### 5.1 Tạo user mới

- Tại giao diện admin click chọn `Manage`
![image](./images/zimb-19.png)
- Click chọn icon bánh răng góc trên bên phải để tạo user mới
![image](./images/zimb-20.png)
- Cấu hình tên, username và mật khẩu -> Click Finish
![image](./images/zimb-21.png)
![image](./images/zimb-22.png)
- Kiểm tra bằng cách đăng nhập user/pass trên zimbra webmail `http://192.168.57.134:8080/`
![image](./images/zimb-23.png)
- Đăng nhập thành công
![image](./images/zimb-24.png)

Minh họa thao tác tạo user mới:

<img width="673" height="288" alt="image" src="https://github.com/user-attachments/assets/7aa6c644-bca5-423c-9cf8-03b2e54bcbd2" />

### 5.2 Thiết lập chính sách mật khẩu

- Tại giao diện admin click chọn `Configure`
![image](./images/zimb-25.png)
- Tại phần COS click chuột phải `default` chọn `edit`
![image](./images/zimb-26.png)
- Chuyển tới tab `Advanced`, panel bên phải hiển thị phần cấu hình chính sách mật khẩu
![image](./images/zimb-27.png)

<img width="849" height="393" alt="image" src="https://github.com/user-attachments/assets/3ddc8d3a-3929-45a4-ba30-79efd8019693" />

### 5.3 Thiết lập chữ ký và Forward email

- Tại giao diện web mail chọn `PREFERENCES` -> Signature
![image](./images/zimb-28.png)
- Tại đây cấu hình tên, chữ ký và lưu.
![image](./images/zimb-29.png)

- Tại giao diện web mail chọn `PREFERENCES` -> Mail
![image](./images/zimb-28.png)
- Tại phần Receiving Message cấu hình mail muốn forward
![image](./images/zimb-30.png)

Minh họa thiết lập chữ ký & forward:

<img width="857" height="380" alt="image" src="https://github.com/user-attachments/assets/9a28e0b5-b19f-4f56-9c64-e5998ee24466" />
<img width="854" height="284" alt="image" src="https://github.com/user-attachments/assets/40501703-ab07-4dae-98ca-6b30c745096f" />

### 5.4 Tìm ID Mailbox và chỉnh sửa Quota

- Trên server Zimbra đi tới thư mục `/opt/zimbra/store/0/`

```bash
cd /opt/zimbra/store/0/
```
![image](./images/zimb-31.png)

- Tìm ID mailbox từ account email

```bash
su zimbra
zmprov getMailboxInfo admin@anthanh264.com
```
![image](./images/zimb-32.png)

- Tại giao diện admin click chọn `Manage Accounts`
![image](./images/zimb-43.png)
- Click chuột phải vào user cần sửa, chọn `edit`
![image](./images/zimb-44.png)
- Tại tab `Advanced`, panel bên phải là nơi chỉnh sửa cấu hình quota, có thể chỉnh sửa và lưu ở nút `Save` góc trên bên phải
![image](./images/zimb-45.png)

Minh họa tìm ID mailbox và chỉnh sửa quota:

<img width="403" height="74" alt="image" src="https://github.com/user-attachments/assets/974761ad-57a5-4914-b591-53dd80df6519" />

<img width="839" height="233" alt="image" src="https://github.com/user-attachments/assets/aeff5ba8-1b71-4970-8d15-0eeb44afd6b6" />

Chỉnh sửa quota để tránh làm "tràn" ổ cứng máy chủ:

<img width="839" height="314" alt="image" src="https://github.com/user-attachments/assets/459075e2-b27d-4ab7-8d9a-28c7004c8daa" />

### 5.5 Đổi mật khẩu account admin

- Trên server Zimbra login với user zimbra

```bash
su zimbra
```

- Kiểm tra user nào có quyền admin

```bash
zmprov gaaa
```

- Đổi pass admin bằng lệnh

```bash
zmprov sp admin@anthanh264.com Qaz123456
```
![image](./images/zimb-33.png)

Cú pháp đổi mật khẩu tổng quát:

```bash
su - zimbra -c "zmprov sp admin@domain.com *mật_khẩu_mới*"
```

### 5.6 Kiểm tra Log gửi/nhận email

- Trên webmail thực hiện gửi mail trong local để test
![image](./images/zimb-35.png)
- Trên server thực hiện kiểm tra log bằng lệnh

```bash
tail -f /var/log/mail.log
```

- Log gửi và nhận mail
![image](./images/zimb-34.png)

Bổ sung các lệnh kiểm tra log chi tiết theo từng thành phần Zimbra (`/var/log/zimbra.log`):

- Xem log thời gian thực (theo dõi)

```bash
tail -f /var/log/zimbra.log
```
<img width="830" height="133" alt="image" src="https://github.com/user-attachments/assets/0e19c15b-2777-450b-b1a9-1ec31f70ebfe" />

- Lọc log để xem quá trình gửi nhận cụ thể (qua Postfix LMTP)

```bash
cat /var/log/zimbra.log | grep "postfix/lmtp"
```
<img width="892" height="353" alt="image" src="https://github.com/user-attachments/assets/b69aa348-96ab-4634-9d14-1f22b3aeb38d" />

- Log liên quan đến Amavis (chống spam/virus)

```bash
grep -i "amavis" /var/log/zimbra.log | tail -n 50
```

### 5.7 Thay đổi logo giao diện

- Chuẩn bị logo cần đổi: Logo trước khi đăng nhập (440×60 px) và Logo sau khi đăng nhập (200×35 px). Trong lab này file logo mới là `logo1` upload lên `/home/mailserver`.
- Trên server tạo thư mục và upload logo

```bash
mkdir /opt/zimbra/jetty/webapps/zimbra/logos
cd /opt/zimbra/jetty/webapps/zimbra/logos
cp /home/mailserver/logo1.jpg logo1.jpg
```
![image](./images/zimb-36.png)

- Phân quyền zimbra

```bash
chown zimbra:zimbra *
```
![image](./images/zimb-37.png)

- Login vào acc zimbra và đổi logo bằng lệnh sau

```bash
su zimbra
cd /opt/zimbra/jetty/webapps/zimbra/logos
zmprov md anthanh264.com zimbraSkinLogoURL /logos/logo1.jpg
zmprov md anthanh264.com zimbraSkinLogoLoginBanner /logos/logo1.jpg
zmprov md anthanh264.com zimbraSkinLogoAppBanner /logos/logo1.jpg
```
![image](./images/zimb-38.png)

### 5.8 Thay đổi title web (webmail/webadmin)

**Thay đổi title webmail**

- Chỉnh sửa dòng 3709 trong file `/opt/zimbra/jetty/webapps/zimbra/WEB-INF/classes/messages/ZmMsg.properties`

```bash
nano /opt/zimbra/jetty/webapps/zimbra/WEB-INF/classes/messages/ZmMsg.properties
```
Nhấn `Ctrl + /` và điền `3709` để tới dòng chỉnh sửa
![image](./images/zimb-39.png)

- Sửa, lưu, restart mailboxd để apply

```bash
su zimbra
zmmailboxdctl restart
```
- Kiểm tra
![image](./images/zimb-41.png)

**Thay đổi title webadmin**

- Chỉnh sửa dòng 20 trong file `/opt/zimbra/jetty_base/webapps/zimbraAdmin/WEB-INF/classes/messages/ZabMsg.properties`

```bash
nano /opt/zimbra/jetty_base/webapps/zimbraAdmin/WEB-INF/classes/messages/ZabMsg.properties
```
Nhấn `Ctrl + /` và điền `20` để tới dòng chỉnh sửa
![image](./images/zimb-40.png)

- Sửa, lưu, restart mailboxd để apply

```bash
su zimbra
zmmailboxdctl restart
```
- Kiểm tra
![image](./images/zimb-42.png)

### 5.9 Backup và Restore

**Cách 1 – Dùng lệnh `zmmailbox` (đơn giản, theo từng user)**

- Backup: login vào user zimbra để thực hiện chạy lệnh backup. Ví dụ với acc `user1@anthanh264.com`

```bash
su - zimbra
zmmailbox -z -m user1@anthanh264.com getRestURL "/?fmt=tgz" > /opt/zimbra/backup/backup_user1.tgz
```

- Restore: login vào user zimbra để thực hiện chạy lệnh restore

```bash
su - zimbra
zmmailbox -z -m user1@anthanh264.com postRestURL "/?fmt=tgz&resolve=reset" "/opt/zimbra/backup/backup_user1.tgz"
```

**Cách 2 – Dùng script tự động backup/restore (khuyến nghị vận hành thực tế)**

- Tạo file script tự động backup, restore (1 user, 1 server)

<img width="904" height="371" alt="image" src="https://github.com/user-attachments/assets/3fd6d9a1-a037-4110-ab24-f27376d10cba" />

- Chạy backup thủ công

<img width="658" height="381" alt="image" src="https://github.com/user-attachments/assets/f19af92a-396b-4e4d-9e24-5768f8590b36" />

- Thiết lập backup tự động lúc 2h sáng bằng cron

```bash
echo "0 2 * * * /opt/zimbra/zimbra_backup_complete.sh backup" | crontab -
```

- Backup tất cả

```bash
/opt/zimbra/zimbra_backup_complete.sh backup
```

- Backup 1 user

```bash
/opt/zimbra/zimbra_backup_complete.sh backup nguyenvana@example.com.vn
```

- List backup

```bash
/opt/zimbra/zimbra_backup_complete.sh list
```
<img width="560" height="316" alt="image" src="https://github.com/user-attachments/assets/76d8f746-873d-4324-bfd6-b63386d511f3" />

- Cleanup (xóa backup cũ hơn N ngày, ví dụ 30 ngày)

```bash
/opt/zimbra/zimbra_backup_complete.sh cleanup 30
```

- Xem hướng dẫn restore

```bash
/opt/zimbra/zimbra_backup_complete.sh restore
```
<img width="895" height="393" alt="image" src="https://github.com/user-attachments/assets/2b336d15-5e36-45b7-964b-de81f94afe5a" />

> **Lưu ý:** Đã restore thử nhưng thời điểm test hiện đang không có dữ liệu để restore — cần kiểm tra lại nguồn backup trước khi thao tác trên môi trường production.

### 5.10 Chuyển data sang node khác (Migration)

- Backup từ server cũ

```bash
/opt/zimbra/zimbra_migration_complete.sh backup-old mail.old.com
```

- Copy sang server mới

```bash
/opt/zimbra/zimbra_migration_complete.sh copy mail.old.com mail.new.com
```

- Prepare users trên server mới

```bash
/opt/zimbra/zimbra_migration_complete.sh prepare-new mail.new.com
```

- Restore trên server mới

```bash
/opt/zimbra/zimbra_migration_complete.sh restore-new mail.new.com
```

- Verify

```bash
/opt/zimbra/zimbra_migration_complete.sh verify mail.old.com mail.new.com
```

---

## References

1. [Zimbra Daffodil (v10) Single-Server Installation Guide](https://zimbra.github.io/documentation/zimbra-10/single-server-install.html)
2. [Instalasi Zimbra OSE 10.1.4 di Ubuntu 22.04](https://saad.web.id/2025/02/instalasi-zimbra-ose-10-1-4-di-ubuntu-22-04/)
3. [Cài đặt Zimbra 10 Open Source Edition trên Ubuntu 20.04](https://dotrungquan.info/cai-dat-zimbra-10-open-source-edition-tren-ubuntu-20-04/)
4. [SOLVED MISSING: sysstat does not appear to be installed](https://forums.zimbra.org/viewtopic.php?t=39042)
5. [Zimbra webmail not working on port 80/443](https://serverok.in/zimbra-webmail-not-working)
6. [Setting up a password security policy in Zimbra](https://community.zextras.com/setting-up-a-password-security-policy-in-zimbra/)
7. [Đổi mật khẩu account admin zimbra – [HƯỚNG DẪN]](https://wiki.nhanhoa.com/kb/doi-mat-khau-account-admin-zimbra/)
