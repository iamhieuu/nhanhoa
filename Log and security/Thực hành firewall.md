# Firewall — Thực hành trên Ubuntu 22.04 & Windows Server 2022

## Mục lục

- [I. Firewall trên Linux (Ubuntu 22.04)](#i-firewall-trên-linux-ubuntu-2204)
  - [1. iptables](#1-iptables)
  - [2. ufw](#2-ufw)
  - [3. csf](#3-csf)
    - [3.1. Cài đặt CSF thật trên Ubuntu 22.04](#31-cài-đặt-csf-thật-trên-ubuntu-2204)
    - [3.2. Case thực tế: Gỡ chặn IP đăng nhập sai quá nhiều lần](#32-case-thực-tế-gỡ-chặn-ip-đăng-nhập-sai-quá-nhiều-lần)
- [II. Firewall trên Windows Server 2022](#ii-firewall-trên-windows-server-2022)
  - [1. Cấu hình qua GUI](#1-cấu-hình-qua-gui)
  - [2. Cấu hình qua PowerShell](#2-cấu-hình-qua-powershell)
- [III. Tcpdump và Wireshark](#iii-tcpdump-và-wireshark)
  - [1. TCPdump trên Ubuntu 22.04](#1-tcpdump-trên-ubuntu-2204)
  - [2. Wireshark trên Ubuntu 22.04](#2-wireshark-trên-ubuntu-2204)


---

## I. Firewall trên Linux (Ubuntu 22.04)

### 1. iptables

Check thông tin cơ bản:  

<img width="233" height="174" alt="{F7F6010F-F32A-4552-8368-94751BF8327B}" src="https://github.com/user-attachments/assets/929550bf-b487-4b04-9c0a-ab5b23da7a82" />

Xem tất cả rule:
```
sudo iptables -L -n
```
<img width="483" height="157" alt="{EF8125E2-F66F-48FF-918E-1265C3AB5E82}" src="https://github.com/user-attachments/assets/51c2721d-d5d6-4cc6-9c7c-429e92186d83" />

Xem dạng lệnh:
```
sudo iptables -S
```
<img width="289" height="112" alt="{09135A9E-E92E-4A86-B098-E438EDD5C838}" src="https://github.com/user-attachments/assets/edf6c338-c607-4ab2-9e46-dc388792d517" />

Cú pháp cơ bản dùng trong lab:
```
iptables -[A/I/D] CHAIN -[p/s/d/i/o] [giá_trị] --dport [port] -j [ACTION]
```
<img width="445" height="40" alt="{465EA0A8-D469-4207-995F-4E9FD4D62BA5}" src="https://github.com/user-attachments/assets/724e7a16-ddfe-4338-8214-557e6fb736bd" />

Chống brute force SSH — tối đa 1 kết nối mới mỗi 3 phút: 

<img width="442" height="82" alt="image" src="https://github.com/user-attachments/assets/5dd02942-2bd6-45f1-aeda-58ed34b0e1f8" />

Chèn rule (insert):  

<img width="862" height="184" alt="{1982501C-ACB2-4E9E-B87C-EB1C04577480}" src="https://github.com/user-attachments/assets/84731b3c-4b2f-469f-8474-3f97671667dc" />

Xóa rule:  

<img width="872" height="220" alt="{96506455-2421-403B-B32F-F525E3F8AF9F}" src="https://github.com/user-attachments/assets/b42a5e6c-5686-41c3-9e05-41779a1ccc13" />

Log lại các rule bị DROP:
```
sudo iptables -A INPUT -m limit --limit 5/min -j LOG --log-prefix "iptables-drop: "
```

Lưu rule vĩnh viễn (mặc định iptables mất rule khi reboot):  
```
sudo iptables-save > /tmp/iptables-backup.txt
```
<img width="879" height="253" alt="{FACAE0D0-3194-494C-B20B-24F4854EA615}" src="https://github.com/user-attachments/assets/0e0aa80a-a0e9-41ec-9a6c-ef2c293d5e27" />

```
sudo iptables-restore < /tmp/iptables-backup.txt
```
Hoặc dùng cách chuẩn hơn:  
```
sudo netfilter-persistent save
```
<img width="529" height="88" alt="{3A10AA04-B675-4FFC-BF48-D58EF2510C19}" src="https://github.com/user-attachments/assets/b0631c3d-bf8c-4522-b94d-b7bc3025aaf8" />

Clear toàn bộ rule (flush):  

<img width="575" height="231" alt="{F1AC5CAF-38D7-4A0B-AB8C-8A60CD9FD68A}" src="https://github.com/user-attachments/assets/8919bd3a-5ace-4b44-a9a1-fce5ea39d9c1" />

### 2. ufw

Lệnh cơ bản check trạng thái và xem rule:
```
sudo ufw status verbose
sudo ufw status numbered
```

<img width="495" height="337" alt="{A1C5DEFF-58E0-4868-BD89-2816E7174BB5}" src="https://github.com/user-attachments/assets/983490b4-a422-4455-9279-b2d2732817f4" />

Thêm rule:  

<img width="381" height="316" alt="{925E20C1-8855-4F20-B32C-E4A6E4DBAF9C}" src="https://github.com/user-attachments/assets/db12d792-c4dd-40d5-a173-c63afc9a30e3" />

`sudo ufw allow from ... to ...`: cho phép/chặn từ 1 IP cụ thể đến port cụ thể.

Xóa rule:  

<img width="410" height="219" alt="{91B86AA2-87FD-4966-B8C7-F83FE90F2091}" src="https://github.com/user-attachments/assets/511eeb31-04b2-4049-aa04-03e6fc7f79c3" />

Bật log:  

<img width="324" height="72" alt="{1F354A98-C653-4FCE-8B41-DF8A3BD778F7}" src="https://github.com/user-attachments/assets/5d0b5c84-398a-4d71-a763-8143d5f087fd" />

Check log ufw:  

<img width="868" height="88" alt="{EA5DC344-BC4A-4020-9D72-6493C0E193E4}" src="https://github.com/user-attachments/assets/c79763f0-03c9-4ab9-ab10-9560f768e0f5" />

Reset toàn bộ rule về mặc định:
```
sudo ufw reset
```

### 3. csf

#### 3.1. Cài đặt CSF thật trên Ubuntu 22.04

Cài thư viện Perl cần thiết trước:
```
sudo apt install perl libwww-perl libio-socket-ssl-perl curl -y
```

Tải, giải nén và chạy script cài đặt:
```
wget http://download.configserver.com/csf.tgz
tar -xzf csf.tgz
cd csf
sudo sh install.sh
```

Kiểm tra các module Perl cần thiết đã đủ chưa (CSF có sẵn script test):  
```
perl /usr/local/csf/bin/csftest.pl
```

Do Ubuntu dùng UFW mặc định, cần tắt UFW trước khi bật CSF để tránh xung đột rule:
```
sudo ufw disable
sudo systemctl enable csf
sudo systemctl start csf
```

Kiểm tra CSF đã chạy:
```
sudo csf -v
sudo systemctl status csf
sudo systemctl status lfd
```

> `lfd` (Login Failure Daemon) là tiến trình đi kèm CSF, chạy nền để theo dõi log đăng nhập sai và tự động đẩy IP vi phạm vào danh sách chặn tạm thời — đây chính là thành phần đứng sau case thực tế ở mục 3.2 bên dưới.

#### 3.2. Case thực tế: Gỡ chặn IP đăng nhập sai quá nhiều lần

Tình huống thường gặp khi hỗ trợ khách hàng: khách hàng SSH/đăng nhập sai mật khẩu nhiều lần → bị CSF/lfd tự động chặn → không đăng nhập được nữa dù đã nhớ đúng mật khẩu. Quy trình xử lý (tham khảo & áp dụng theo hướng dẫn Nhân Hòa — xem link ở phần cuối):

**Bước 1: Xác định IP đang bị chặn**  

Khách hàng thường thấy lỗi kết nối bị từ chối/timeout. Yêu cầu khách hàng cung cấp IP hiện tại (kiểm tra nhanh qua trang [showip.net](https://showip.net) hoặc tương đương), hoặc tự tra trong log SSH trên server:
```
sudo grep "Failed password" /var/log/auth.log | tail -20
```

**Bước 2: SSH vào server, kiểm tra IP có đang bị CSF chặn không**
```
sudo csf -g 45.117.80.119
```
- `-g` (grep): tìm kiếm trong toàn bộ rule iptables/ip6tables và các danh sách tempban của CSF xem IP này có đang bị chặn ở đâu không.
- Nếu bị chặn, kết quả sẽ hiển thị IP nằm trong chain `DENY` hoặc trong `csf.tempban`, kèm lý do (VD: đăng nhập sai quá 5 lần trong 3600 giây).

**Bước 3: Gỡ chặn IP**
```
sudo csf -dr 45.117.80.119
```
- `-dr` (deny remove): gỡ IP khỏi danh sách chặn `csf.deny` (hoặc danh sách tempban), mở lại kết nối ngay lập tức.

**Bước 4: Xác nhận đã gỡ thành công**
```
sudo csf -g 45.117.80.119
```
Chạy lại lệnh kiểm tra ở Bước 2 — nếu không còn thấy IP xuất hiện trong kết quả, nghĩa là đã gỡ chặn thành công. Yêu cầu khách hàng thử kết nối lại để xác nhận.

> **Lưu ý vận hành:** nếu IP bị chặn lặp đi lặp lại do khách hàng/nhân viên thường xuyên gõ sai mật khẩu (IP tin cậy, cố định), nên cân nhắc thêm hẳn IP đó vào whitelist thay vì gỡ chặn thủ công mỗi lần:
> ```
> sudo csf -a 45.117.80.119 Whitelist IP van phong
> ```
> `-a` sẽ ghi IP vào `/etc/csf/csf.allow`, luôn được phép đi qua firewall bất kể có bị đưa vào `csf.deny` sau này hay không.

---

## II. Firewall trên Windows Server 2022

### 1. Cấu hình qua GUI

Windows Defender Firewall là thành phần bảo mật tích hợp sẵn, quản lý theo Profiles (Domain/Private/Public) thay vì Chains như Linux.

Cách truy cập: `Win + R` → gõ `wf.msc` → Enter.  

<img width="781" height="478" alt="image" src="https://github.com/user-attachments/assets/0ec6e956-7caa-4b79-9dc5-6fbbec5dd316" />

Các thành phần chính: **Inbound Rules** (kết nối từ ngoài vào server, VD cho phép truy cập web port 80), **Outbound Rules** (ứng dụng trong server ra internet, VD chặn phần mềm độc hại gửi dữ liệu ra ngoài), **Connection Security Rules** (kết nối bảo mật IPsec giữa các máy).

**Tạo Inbound Rule qua GUI:**  
`Inbound Rules → New Rule → Rule Type: Port → chọn Allow/Deny the connection → chọn Profile: Domain, Private, Public`  

<img width="731" height="385" alt="{40DC1218-7B97-4E67-83ED-A274BC64599F}" src="https://github.com/user-attachments/assets/cc8a79c7-e267-4f70-aebe-209f8f9756c9" />

**Tạo Outbound Rule — chặn 1 ứng dụng cụ thể:**  
`Outbound Rules → New Rule → Rule Type: Program → Browse tới file executable → Block the connection → đặt tên → Finish`  

<img width="473" height="394" alt="{79916A0B-3476-4F62-B725-08CD681DD898}" src="https://github.com/user-attachments/assets/7a3df255-806c-4355-a6da-ea5015d75eff" />

### 2. Cấu hình qua PowerShell

PowerShell cho phép tự động hóa hoàn toàn.

Xem trạng thái tất cả profile:
```powershell
Get-NetFirewallProfile | Select Name, Enabled, DefaultInboundAction, DefaultOutboundAction
```

<img width="475" height="78" alt="{67B1261E-48CB-4D11-8713-48052C05A0FA}" src="https://github.com/user-attachments/assets/f576fd31-bf5f-4126-8b1a-62f071fd7b18" />

Xem tất cả rules:
```powershell
Get-NetFirewallRule | Select DisplayName, Direction, Action, Enabled | Format-Table
```

<img width="478" height="264" alt="{47916A39-1225-46DD-B4F6-B0F1287ABDE3}" src="https://github.com/user-attachments/assets/23e98844-9355-4fc9-ae8d-3bc26f155073" />

Lọc rules đang bật:
```powershell
Get-NetFirewallRule -Enabled True | Format-Table DisplayName, Direction, Action
```

<img width="482" height="146" alt="{CC21258B-2B78-4A26-8CBF-75366A3A29D1}" src="https://github.com/user-attachments/assets/971ae27f-d80c-4445-ac22-4b2f898fb8c0" />

Xem rule kèm thông tin port:
```powershell
Get-NetFirewallRule -DisplayName "Allow HTTP*" | Get-NetFirewallPortFilter
```

<img width="416" height="129" alt="{C2799A54-7F72-4F04-9C51-C0793C435F8D}" src="https://github.com/user-attachments/assets/399dfe41-d2db-4cc6-9d26-5a93f74d80f8" />

Cho phép port inbound (HTTP):
```powershell
New-NetFirewallRule `
    -DisplayName "Allow HTTP Inbound" `
    -Direction Inbound `
    -Protocol TCP `
    -LocalPort 80 `
    -Action Allow `
    -Profile Any `
    -Enabled True
```

<img width="486" height="312" alt="{7F1864BE-4D81-4B00-9F42-AEDD40FE1365}" src="https://github.com/user-attachments/assets/1fc15927-ef40-46a6-b5ae-06cfa21899b0" />

Cho phép RDP chỉ từ dải mạng nội bộ:
```powershell
New-NetFirewallRule `
    -DisplayName "Allow RDP Internal Only" `
    -Direction Inbound `
    -Protocol TCP `
    -LocalPort 3389 `
    -RemoteAddress 192.168.136.133/24 `
    -Action Allow `
    -Profile Domain,Private `
    -Enabled True
```

Chặn 1 IP cụ thể:
```powershell
New-NetFirewallRule `
    -DisplayName "Block Attacker 192.168.136.121" `
    -Direction Inbound `
    -RemoteAddress 192.168.136.121 `
    -Action Block `
    -Enabled True
```

---

## III. Tcpdump và Wireshark

### 1. TCPdump trên Ubuntu 22.04

Cài đặt tcpdump:
```
sudo apt install -y tcpdump
```
<img width="391" height="111" alt="{B9BD8002-E595-48E9-B262-E2C51C2BC4A1}" src="https://github.com/user-attachments/assets/67e87d46-85e0-4f49-888d-0f5df5ce46ae" />

Tcpdump cần quyền root để bắt gói tin — dùng `sudo tcpdump`, hoặc gán user vào group `pcap`:  

<img width="437" height="129" alt="{E2E1323F-1A76-4FBF-AAB7-AA4DE43BB518}" src="https://github.com/user-attachments/assets/691150ac-d0e6-4b4d-9a9c-b906c4e9992d" />

Xem tất cả card mạng:
```
sudo tcpdump -D
```
<img width="432" height="128" alt="{F3978C58-1BC4-4E69-AF7B-615966A004AD}" src="https://github.com/user-attachments/assets/7c68364a-e234-4d2b-ae17-eead8a028d31" />

Cú pháp cơ bản `tcpdump [options] [filter expression]`. Ví dụ:
```
sudo tcpdump -i ens33            # bắt tất cả gói tin
sudo tcpdump -i ens33 -c 10      # bắt 10 gói rồi dừng
```
<img width="792" height="210" alt="{54178F18-5B61-4D04-92B6-1AB5C1323A6A}" src="https://github.com/user-attachments/assets/34c840bb-a070-4210-b24a-c9562d61d0d2" />

**Thực hành các kịch bản bắt gói tin cụ thể:**

Bắt 10 gói tin bất kỳ:  

<img width="858" height="224" alt="{5E92B9AA-5665-4E12-896F-EAEC81CF8820}" src="https://github.com/user-attachments/assets/58be28e5-981e-4d2b-bfa8-20897a8df6da" />

Bắt gói TCP:
```
sudo tcpdump -i ens33 tcp -c 20
```
<img width="846" height="331" alt="{063A7491-1A97-4C97-894A-E3FF9F9B33A9}" src="https://github.com/user-attachments/assets/e6e5b0da-8c71-4061-8ad8-c361c90228b3" />

Bắt port 22 (SSH):
```
sudo tcpdump -i ens33 port 22 -c 20
```
<img width="828" height="331" alt="{E928AB06-D2DA-41AD-989F-25AB7D85F602}" src="https://github.com/user-attachments/assets/e82e5670-149a-4851-b0d8-b42345832928" />

Bắt DNS:
```
sudo tcpdump -i ens33 udp port 53 -A -c 20
```
<img width="689" height="350" alt="{D5C4B4AC-B18C-4D57-B2C6-B635F9C75337}" src="https://github.com/user-attachments/assets/dfe1c1c6-df3d-4abc-be72-0cec6f925456" />

Xem nội dung ASCII của gói tin:
```
sudo tcpdump -i ens33 -A port 80 -c 20
```
<img width="817" height="404" alt="{26190858-7BC3-4BB5-9245-B0492DFE585C}" src="https://github.com/user-attachments/assets/4e24e7c4-901d-4b05-b3fa-71613b8f323f" />

Xem Hex + ASCII:
```
sudo tcpdump -i ens33 -X port 80 -c 10
```
<img width="860" height="330" alt="{0705BC20-2D1F-4239-B4C0-5E3A2E123565}" src="https://github.com/user-attachments/assets/a0cccfe2-6eaf-429f-a9b1-b70f2ddd98b2" />

Lưu file và đọc file:
```
sudo tcpdump -i ens33 -w /var/log/tcpdump/capture_$(date +%Y%m%d_%H%M%S).pcap
```
<img width="873" height="354" alt="{3E6A0890-2410-46F2-B8B5-419465680967}" src="https://github.com/user-attachments/assets/e1af904a-1c25-4ed7-9452-c861be53fa20" />

### 2. Wireshark trên Ubuntu 22.04

Wireshark là công cụ phân tích gói tin mạng có giao diện đồ họa (GUI), miễn phí và mã nguồn mở — giúp nhìn thấy toàn bộ dữ liệu qua card mạng, phân tích gói tin chi tiết, debug sự cố mạng/ứng dụng, kiểm tra bảo mật hệ thống.

Công cụ thực hiện tương ứng: **Ubuntu Server → TShark** (CLI), **Ubuntu Desktop → Wireshark** (GUI). Trên Ubuntu Server chủ yếu dùng TShark để bắt/phân tích gói tin, ghi log hoặc lưu `.pcap` để phân tích sau bằng Wireshark GUI trên máy khác.

#### Cài TShark

```
sudo apt install -y tshark
```
<img width="381" height="93" alt="{6C55FC12-A9FA-47EE-A1E7-5177266237D8}" src="https://github.com/user-attachments/assets/93196040-4026-4e30-bdd5-4c5185b27ae2" />

Thêm user vào group wireshark:
```
sudo usermod -aG wireshark $USER
```

Xem danh sách card mạng:  
```
tshark -D
```
<img width="366" height="203" alt="{2FA47188-50C4-48B2-867D-88878FDB6648}" src="https://github.com/user-attachments/assets/37e0e836-9329-4b9e-a73d-234c4ea0c299" />

Bắt gói tin cơ bản:
```
tshark -i ens33 -c 10
```
<img width="644" height="195" alt="{8D1C01A2-BE91-48F9-BDB3-050E68992352}" src="https://github.com/user-attachments/assets/2522f2ac-2e86-4d00-b189-a005afd89504" />

Lưu vào file pcap:
```
tshark -i ens33 -c 100 -w /var/log/tcpdump/tshark.pcap
```
<img width="793" height="358" alt="{B4F3044F-F3AA-4A2A-A486-ED9D458CD6FB}" src="https://github.com/user-attachments/assets/0e406bab-c686-4d71-8bc6-eee69ab92dd9" />

Chỉ hiển thị IP nguồn, đích và giao thức:
```
tshark -i ens33 -c 20 -T fields \
    -e frame.number \
    -e ip.src \
    -e ip.dst \
    -e _ws.col.Protocol \
    -e frame.len
```
<img width="877" height="324" alt="{BC58000A-7F6A-4AFD-A382-7940AE4B4A1B}" src="https://github.com/user-attachments/assets/ac11cf9b-def1-4cde-9ebf-fecde9f6f722" />

#### Cài Wireshark (GUI)

```
sudo apt install -y wireshark
```
<img width="489" height="408" alt="{E2218600-F563-4D29-8F1E-3F9E0F6CC4B4}" src="https://github.com/user-attachments/assets/6d10003b-acc6-4bbd-82b8-63e9b436f3e2" />

#### Copy file PCAP về máy local

```
scp hostname@IPserver:/var/log/tcpdump/file.pcap ~/Downloads/
```
<img width="367" height="240" alt="{5A23A37B-88AC-4A2F-8939-32D7CC318ACA}" src="https://github.com/user-attachments/assets/9ce5be1a-c0bd-4ca4-8e42-aae2b3cdd903" />

#### Phân tích với Wireshark GUI

Mở Wireshark → **File → Open** → chọn file `.pcap` đã copy về → **Open**.  
<img width="479" height="418" alt="{021B106B-5C10-44D9-910D-FA2B87248BFA}" src="https://github.com/user-attachments/assets/cc665498-b0b3-4404-8dcb-8daa4cee9527" />

**Quy ước màu sắc mặc định trong Wireshark:**
-  Xanh lá: TCP traffic thông thường
-  Xanh dương: UDP traffic
-  Đen: Gói tin lỗi
-  Vàng: Retransmissions, out of order
-  Đỏ: TCP RST, lỗi kết nối
-  Hồng: ICMP errors
-  Tím: IPv6 traffic

**Display Filters** — lọc theo giao thức, IP, port:  
<img width="475" height="403" alt="{5108A702-4BD6-4C91-9A0D-7D34EC950A47}" src="https://github.com/user-attachments/assets/ecafbc67-b15b-49a8-8e62-93e12661bf62" />

**Follow TCP Stream** — màu đỏ: client gửi (request), màu xanh: server trả lời (response):  
<img width="823" height="415" alt="{D89335AE-069B-43A2-B78A-946B5E0EE794}" src="https://github.com/user-attachments/assets/60b1845d-1bf7-403e-9467-0bebd63b97f0" />  
<img width="329" height="401" alt="{9C8A1659-1921-4C95-91F1-BA127E55DE7D}" src="https://github.com/user-attachments/assets/45c28889-6110-4fe3-bd9d-3901f7a4ade1" />

**Statistics (Thống kê)** — menu Statistics gồm: Protocol Hierarchy (% từng giao thức), Conversations (tất cả kết nối), Endpoints (tất cả IP/port), IO Graphs (biểu đồ băng thông theo thời gian), Flow Graph (luồng kết nối), DNS (thống kê DNS queries), HTTP (thống kê HTTP requests):  
<img width="392" height="249" alt="{5DEA6652-EAAD-412E-B597-18E952A59D8C}" src="https://github.com/user-attachments/assets/4864bcd7-f935-4c30-a012-142b0424d634" />
