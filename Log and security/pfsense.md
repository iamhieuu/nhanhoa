## Tìm hiểu triển khai Pfsense

---
## Table of Contents

  - [Tìm hiểu triển khai Pfsense](#tìm-hiểu-triển-khai-pfsense)
    - [1. Pfsense](#1-pfsense)
    - [2. Các lab thực hành với Pfsense](#2-các-lab-thực-hành-với-pfsense)
      - [2.1 Mô hình chung](#21-mô-hình-chung)
      - [2.2 Triển khai Pfsense](#22-triển-khai-pfsense)
      - [2.3 NAT Local Internet](#23-nat-local-internet)
      - [2.3.1 NAT (1-1)](#231-nat-1-1)
      - [2.3.2 Nat Port-forward](#232-nat-port-forward)
    - [2.4 Firewall Rule](#24-firewall-rule)
    - [2.5 DHCP](#25-dhcp)
    - [2.6 Setup OpenVPN với Pfsense (OpenVPN mode tap, OpenVPN mode Tun)](#26-setup-openvpn-với-pfsense-openvpn-mode-tap-openvpn-mode-tun)
      - [2.6.1 PFSense OpenVPN mode tap](#261-pfsense-openvpn-mode-tap)
      - [2.6.2 PFSense OpenVPN mode TUN](#262-pfsense-openvpn-mode-tun)
  - [References](#references)

### 1. Pfsense
- pfSense là một ứng dụng có chức năng định tuyến và tường lửa mạnh, miễn phí, cho phép mở rộng mạng mà không bị thỏa hiệp về bảo mật. Bắt đầu vào năm 2004 từ m0n0wall — một dự án bảo mật tập trung vào hệ thống nhúng — pfSense đã có hơn 1 triệu lượt download và được dùng để bảo vệ các mạng từ quy mô gia đình tới doanh nghiệp lớn. Cộng đồng phát triển tích cực, liên tục bổ sung tính năng cải thiện bảo mật, ổn định và linh hoạt.
- pfSense bao gồm nhiều tính năng thường thấy trên các thiết bị firewall/router thương mại, có GUI trên nền Web giúp quản lý dễ dàng.
- pfSense dựa trên FreeBSD và giao thức CARP (Common Address Redundancy Protocol) của FreeBSD, cung cấp khả năng dự phòng bằng cách nhóm hai hay nhiều firewall vào một nhóm tự động chuyển đổi dự phòng. Hỗ trợ nhiều kết nối WAN nên có thể cân bằng tải.
- pfSense hỗ trợ lọc theo địa chỉ nguồn/đích, cổng nguồn/đích hay địa chỉ IP; hỗ trợ chính sách định tuyến, hoạt động ở chế độ bridge/transparent . Cung cấp cơ chế NAT và port-forward, tuy vẫn còn hạn chế với PPTP, GRE, SIP khi dùng NAT.
- Các tính năng nổi bật của pfSense:
	- Cung cấp dịch vụ tường lửa, High Availability, Load Balancing, ...
	- Sử dụng Web Interface để quản trị, giao tiếp dễ dàng hơn.
	- Hiệu năng cao, ổn định, ...
	- Là phần mềm mã nguồn mở, miễn phí.
	- Cung cấp chức năng tạo VLAN, DNS, ...

Triển khai pfSense sử dụng 2 card mạng ảo:
* Adapter 1 (WAN): để chế độ NAT
* Adapter 2 (LAN): để chế độ Internal Network

<img width="325" height="238" alt="image" src="https://github.com/user-attachments/assets/9e4df100-f024-4c59-ad8a-3a34e66b452b" /> 

Giao diện Pfsense

<img width="860" height="419" alt="image" src="https://github.com/user-attachments/assets/34989ff9-b6a1-479e-bd54-9fee52ab7edf" /> 

### 2. Các lab thực hành với Pfsense
#### 2.1 Mô hình chung
- Bài thực hành thực hiện theo môi trường như trong sơ đồ sau:
![images](./images/pfsense.drawio.png)
- Cấu hình Virtual Network Editor của VMware
	- Trong đó
		* VMNet8: NAT ra Internet
		* VMNet1: Host-only (mạng quản lý PFSense)
		* VMNet2: Host-only (giả lập mạng private)
![images](./images/vm-1.png)

#### 2.2 Triển khai Pfsense
- Tải file iso Pfsense tại [đây](https://repo.ialab.dsu.edu/pfsense/pfSense-CE-2.7.2-RELEASE-amd64.iso.gz)
- Cấu hình máy ảo và card mạng của máy Pfsense:
	- Trong VMWare chọn File -> New Virtual Machine
![images](./images/vm-2.png)
	- Chọn `Next` -> Chọn tới file iso đã tải
![images](./images/vm-4.png)
	- Cấu hình tên và nơi lưu
![images](./images/vm-5.png)
	- Cấu hình dung lượng ổ đĩa
![images](./images/vm-6.png)
	- Chọn `Custom ...` để thêm card mạng
![images](./images/vm-7.png)
	- Chọn `Add` để thêm `Adapter` mạng
![images](./images/vm-8.png)
	- Chọn `Network Adapter` -> Finish
![images](./images/vm-9.png)
	- Cấu hình cho `Network Adapter 2` có network connection là `VMnet1`: đây là Interface Host-only (quản lý PFSense)
![images](./images/vm-10.png)
	- Tương tự thêm và cấu hình cho `Network Adapter 3` có network connection là `VMNet2`: đây là Interface Host-only (giả lập mạng private) -> Close và click `Finish` để khởi tạo máy ảo
![images](./images/vm-11.png)
- Cài đặt Pfsense
	- Sau khi khởi động VM chọn `Accept`
![images](./images/pf-1.png)
	- Chọn `Install` và chọn `OK` để vào mode cài đặt pfSense
![images](./images/pf-2.png)
	- Phần cấu hình phân vùng ổ cứng để tự động:
![images](./images/pf-3.png)
	- Qúa trình cài đặt diễn ra
![images](./images/pf-5.png)
	- Chọn `reboot`
![images](./images/pf-6.png)
	- Cấu hình card mạng cho PFSense
		- Sau khi khởi động lại chọn `1` để cấu hình
![images](./images/pf-7.png)
		- Chọn KHÔNG cấu hình vlan cho Pfsense
![images](./images/pf-8.png)
		- Lựa chọn card mạng cho WAN và LAN
![images](./images/pf-9.png)
	    - Xác nhận thông tin card mạng và chọn 'y'
![images](./images/pf-10.png)
	    - Card mạng đã nhận IP (nếu có DHCP Server), tiếp tục chọn '2' để set ip static cho card
![images](./images/pf-11.png)
![images](./images/pf-12.png)

	- Cấu hình IP cho card mạng WAN như sau (từ trên xuống dưới)
		* Chọn '1' để set IP cho IP WAN
		* Chọn 'no' để không nhận IP từ DHCP
		* Đặt IP tĩnh
		* Đặt subnet mask
		* Đặt Gateway
		* Chọn 'no' để không nhận IP từ DHCPv6
![images](./images/pf-13.png)
![images](./images/pf-15.png)
	- Tương tự cấu hình IP cho card mạng LAN, thực hiện tương tự như WAN. Lưu ý: không đặt gateway và IPv6 cho card LAN
![images](./images/pf-16.png)
- Truy cập web quản lý qua IP Lan
![images](./images/pf-17.png)
- Đăng nhập Username: admin, Password: pfsense.
![images](./images/pf-18.png)
- Tại web admin thực hiện cấu hình LAN2 ở tab Interfaces/InterfaceAssignments, add port `em2`
![images](./images/pf-19.png)
- Chọn `Add` và cấu hình thông số
	* Enable interface
	* Đổi tên interface là LAN2
	* Static IPv4
	* Đặt IP và subnet
![images](./images/pf-20.png)
![images](./images/pf-21.png)
- Hoàn tất triển khai pfsense với 3 interface 1 Wan, 1 Mnt, 1 Lan
![images](./images/pf-22.png)

#### 2.3 NAT Local Internet
- pfSense sử dụng Automatic Outbound NAT, nghĩa là các máy trong LAN sẽ tự động có Internet nếu WAN đã thông.

<img width="725" height="370" alt="image" src="https://github.com/user-attachments/assets/bfc971b7-734a-45ef-9f91-a4d1d3633aa5" /> 

Đảm bảo đang tích chọn Automatic outbound NAT rule generation

- pfSense còn hỗ trợ tính năng NAT 1:1 từ máy Ubuntu trong private network để có thể truy cập Internet ở Public Network
- "NAT 1:1" (One-to-One Network Address Translation) là một kỹ thuật mạng được sử dụng để ánh xạ một địa chỉ IP nội bộ duy nhất với một địa chỉ IP công cộng duy nhất. Điều này cho phép các thiết bị trong mạng nội bộ có thể truy cập và được truy cập từ internet thông qua một địa chỉ IP công cộng cụ thể.
- Cấu hình IP tĩnh trên máy Ubuntu
```
nano /etc/netplan/50-cloud-init.yaml
```
![images](./images/pf-23.png)
- Apply
```
netplan apply
```

#### 2.3.1 NAT (1-1)
- Tiến hành tạo ra Virtual IPs để có thể sử dụng virtual IP này NAT ra bên ngoài.
![images](./images/pf-24.png)
![images](./images/pf-25.png)
![images](./images/pf-26.png)
- Save lại và Apply change
- Tiếp theo thực hiện thiết lập NAT 1:1
![images](./images/pf-27.png)
![images](./images/pf-28.png)
- Chọn sang mục NAT \1:1 rồi sau đó thêm mới.
- Thiết lập như sau :
	* Interface: Chọn WAN
	* External subnet IP : là địa chỉ vitual IP chúng ta đã tạo dùng để thực hiện NAT.
	* Internal IP : Là địa chỉ IP của client mà chúng ta muốn NAT.
	* `SAVE` lại và `Apply change` để chấp nhận sự thay đổi.
![images](./images/pf-29.png)
- Thiết lập rules:
![images](./images/pf-30.png)
- Vào mục rules ấn vào `Add` để thêm một rules rồi thiết lập như sau:
	* Action : Pass
	* Destination : Là địa chỉ mà chúng ta thực hiện NAT
- Sau đó thực hiện `SAVE` lại và `Apply Change`
![images](./images/pf-31.png)
- Kiểm tra NAT qua virtual IP 1:1
![images](./images/pf-32.png)

#### 2.3.2 Nat Port-forward
- Thực hiện NAT port-forward theo mô hình với một client để test thử kết nối đến địa chỉ chúng ta muốn public website ra ngoài, một máy chủ pfSense để thực hiện NAT và một máy chủ web-server có card mạng private
- Thực hiện tương tự NAT 1:1 giống tới bước tạo VIP
- Tiếp tục từ phần tạo NAT
![images](./images/pf-27.png)
- Chọn NAT rồi chọn `Add` và thiết lập như sau :
	* Destination : WAN address - Interface mà chúng ta chọn để NAT ra ngoài
	* Port : HTTP (WEB)
	* Redirect target IP : Là địa chỉ chúng ta dùng để NAT ra ngoài
![images](./images/pf-33.png)
- Sau đó SAVE lại và Apply change để lưu lại thiết lập.
![images](./images/pf-34.png)

### 2.4 Firewall Rule
- Trong pfSense, quy tắc mặc định là: chặn mọi thứ từ ngoài vào WAN, và cho phép mọi thứ từ LAN ra ngoài.
- Để xem các Rule trong Firewall chúng ta truy cập vào Firewall > Rules. Theo mặc định không có mục nào khác ngoài bộ quy tắc Block private networks và Block bogon networks nếu các tùy chọn đó đang hoạt động trên giao diện WAN.
- Click icon Setting ở bên phải của rule Block private networks hoặc Block bogon networks để truy cập trang cấu hình giao diện WAN nơi có thể enable hoặc disable các tùy chọn này.
![images](./images/pf-35.png)
![images](./images/pf-36.png)

- Firewall -> Rules -> Tab LAN. Cho phép mọi thiết bị trong mạng LAN truy cập vào các trang web (duyệt web) trên Internet qua giao thức HTTP và HTTPS:

<img width="766" height="376" alt="image" src="https://github.com/user-attachments/assets/68369b35-69b1-4b12-87d0-fdbd736a9a7c" /> 

### 2.5 DHCP
- Quản lý DHCP của PfSense nằm trong Services -> DHCP Server -> LAN, tích Enable:

<img width="863" height="419" alt="image" src="https://github.com/user-attachments/assets/cb09fd74-5d1a-42d9-a490-e783abc13fbf" /> 

- Ở phần Range: Nhập range IP mà bạn muốn cấp cho máy trạm
- Tích Change DHCP display lease time from UTC to local time và Enable RRD statistics graphs
![images](./images/pf-38.png)
-  Nhấn Save để lưu lại
 - Nếu bạn muốn cấu hình DHCP static mapping cho các server hay muốn máy trạm yêu cầu không thay đổi IP Address khi DHCP Server cấp phát -> Ở mục DHCP Static Mapping for this Interface -> Nhấn Add
![images](./images/pf-39.png)

### 2.6 Setup OpenVPN với Pfsense (OpenVPN mode tap, OpenVPN mode Tun)

> Sử dụng OpenVPN để truy cập mạng nội bộ từ xa, kết nối hai văn phòng với nhau. pfSense hỗ trợ Wizard dựng nhanh (VPN -> OpenVPN -> Wizards -> Local User Access) hoặc tự tạo CA/Certificate thủ công — bên dưới trình bày theo luồng thủ công đầy đủ (2.6.1/2.6.2), có chèn ảnh thực tế lab của Hieuintern vào đúng bước tương ứng.

#### 2.6.1 PFSense OpenVPN mode tap
- Mô hình này sử dụng 3 server, trong đó:
	- Host Firewall cài đặt PFSense.
	- Host server_target: server trong mạng LAN (target network để kết nối VPN tới)
	- Client: cài đặt OpenVPN client. Bài lab thành công khi máy client nhận được IP của mạng LAN2 và có thể kết nối tới server_target.
- Thực hiện trên PFSense
	- Tạo User và Certificate: Tại tab System/Certificate/Authorities, tạo CA cho OpenVPN, CA này sẽ xác thực tất cả các certificate của server VPN và user VPN khi kết nối tới PFSense OpenVPN
![images](./images/pf-40.png)
![images](./images/pf-41.png)
![images](./images/pf-42.png)
	- Chọn "Save", kết quả;
![images](./images/pf-43.png)
	- Tại tab System/Certificate/Certificate, tạo certificate cho server VPN
![images](./images/pf-44.png)
![images](./images/pf-45.png)
![images](./images/pf-46.png)
![images](./images/pf-47.png)
	- Tiếp tục tạo certificate cho user:

<img width="513" height="362" alt="image" src="https://github.com/user-attachments/assets/a20b91c6-3b76-4ef1-8684-9c74ceeca58f" /> *(ảnh Hieuintern — Tạo Client Certificate)*

	- Tại tab System/UserManager, tạo user được VPN
![images](./images/pf-51.png)
	- Khai báo Username, password của User. Sau đó "Save"
![images](./images/pf-52.png)
![images](./images/pf-53.png)
	- Sau khi user được tạo, click vào nút "Edit user" để add certificate cho user đó
![images](./images/pf-54.png)
![images](./images/pf-55.png)
- Tạo VPN Server
	- Tại tab System/Package Manager, cài đặt Plugin openvpn-client-export
![images](./images/pf-56.png)
![images](./images/pf-57.png)
![images](./images/pf-58.png)
	- Tại tab VPN/OpenVPN/Servers, click "Add" để tạo VPN server
![images](./images/pf-59.png)
- Khai báo các thông tin về mode kết nối:
	* Server mode: Remote Access (SSL/TLS + User Auth)
	* Device mode: tap
	* Interface: WAN
	* Local port: 1194
![images](./images/pf-60.png)
- Khai báo các thông tin về mã hóa
	* TLS Configuration: chọn sử dụng TLS key
	* Peer Certificate Authority: chọn CA cho hệ thống đã tạo trước đó (server-ca)
	* Server certificate: chọn cert cho server được tạo (server-cert)
	* Enable NCP: lựa chọn sử dụng mã hóa đường truyền giữa Client và Server, sử dụng các giải thuật mặc định là AES-256-GCM và AES-128-GCM
	* Auth digest algorithm: lựa chọn giải thuật xác thực kênh truyền là SHA256
![images](./images/pf-61.png)
- Khai báo các thông tin về tap
	* Bridge DHCP: cho phép client nhận IP trong LAN thông qua DHCP Server
	* Bridge Interface: lựa chọn LAN được kết nối qua VPN
	* IPv4 local Network: khai báo dải mạng được truy cập thông qua VPN (LAN2)
	* Concurrent Connection: khai báo số lượng client được kết nối VPN đồng thời

![images](./images/pf-62.png)

- Cấu hình Interface
	* Tại tab Interfaces/InterfaceAssignments, add thêm network port của VPN, đặt tên là vpn_lab
![images](./images/pf-63.png)
![images](./images/pf-64.png)
	* Tại tab Interfaces/Bridges, tạo bridge mới và add 2 interface VPNLAB và LAN2 vào bridge
![images](./images/pf-65.png)
![images](./images/pf-66.png)
- Cấu hình Firewall
	* Tại tab Firewall/Rules/WAN, add thêm rule cho phép client kết nối tới port 1194 của VPN. Cho phép WAN port 1194:

<img width="635" height="380" alt="image" src="https://github.com/user-attachments/assets/1fac2c1d-51d7-482b-b7e0-2027372608bf" /> *(ảnh Hieuintern — mở WAN port 1194)*

	* Tại tab Firewall/Rules/LAN2, add rule cho phép lưu lượng đi qua
![images](./images/pf-68.png)
	* Tại tab Firewall/Rules/VPNLAB, add rule cho phép lưu lượng đi qua
![images](./images/pf-69.png)
![images](./images/pf-70.png)
	* Tại tab Firewall/Rules/OPENVPN, add rule cho phép lưu lượng đi qua. Cho phép OpenVPN Interface:

<img width="552" height="359" alt="image" src="https://github.com/user-attachments/assets/406128c4-49ce-49a3-bf80-c334435e53e7" /> *(ảnh Hieuintern — cho phép OpenVPN Interface)*

- Export OpenVPN config
	* Tại tab VPN/OpenVPN/ClientExport, khai báo các thông số:
		* Remote Access Server: lựa chọn OpenVPN server
		* Hostname Resolution: lựa chọn khai báo IP của WAN

<img width="819" height="374" alt="image" src="https://github.com/user-attachments/assets/3660f52c-57e5-49a9-897f-f173d73a013a" /> *(ảnh Hieuintern — Export Client Config)*

![images](./images/pf-73.png)
- Thực hiện trên Client, kết nối VPN
- Tải OpenVPN tại: https://openvpn.net/community-downloads/
- Gán file .ovpn vào thư mục `C:\Users\YourUsername\OpenVPN\config\`

<img width="497" height="268" alt="image" src="https://github.com/user-attachments/assets/1178f109-ecf4-42cd-bc01-e9c45c9c46e1" /> *(ảnh Hieuintern — chạy OpenVPN thành công)*

- Kiểm tra ping và truy cập `10.10.10.23` của `Ubuntu Server` nằm trong Private Lan
![images](./images/pf-95.png)

#### 2.6.2 PFSense OpenVPN mode TUN
- Mục tiêu LAB
	* Mô hình này sử dụng 3 server, trong đó:
		* Host Firewall cài đặt PFSense.
		* Host server_target: server trong mạng LAN (target network để kết nối VPN tới)
		* Client: cài đặt OpenVPN client. Bài lab thành công khi máy client nhận được IP của tunnel và có thể kết nối tới các private VLAN của server_target.
- Tại tab System/Certificate/Authorities, tạo CA cho OpenVPN, CA này sẽ xác thực tất cả các certificate của server VPN và user VPN khi kết nối tới PFSense OpenVPN
![images](./images/pf-40.png)
![images](./images/pf-41.png)
![images](./images/pf-42.png)
- Chọn "Save", kết quả;
![images](./images/pf-43.png)
- Tại tab System/Certificate/Certificate, tạo certificate cho server VPN
![images](./images/pf-44.png)
![images](./images/pf-45.png)
![images](./images/pf-46.png)
![images](./images/pf-47.png)
- Tiếp tục tạo certificate cho user
![images](./images/pf-76.png)
![images](./images/pf-77.png)
- Tại tab System/UserManager, tạo user được VPN
- Khai báo Username, password của User. Sau đó "Save"
![images](./images/pf-78.png)
![images](./images/pf-79.png)
- Sau khi user được tạo, click vào nút "Edit user", Edit user vừa tạo, add certificate cho user đó
![images](./images/pf-80.png)
![images](./images/pf-81.png)
- Tại tab VPN/OpenVPN/Servers, click "Add" để tạo VPN server mới
- Khai báo các thông tin về mode kết nối:
	* Server mode: Remote Access (SSL/TLS + User Auth)
	* Device mode: tun
	* Interface: WAN
	* Local port: 1195 (tùy ý lựa chọn port)
![images](./images/pf-82.png)
- Khai báo các thông tin về mã hóa
	* TLS Configuration: chọn sử dụng TLS key
	* Peer Certificate Authority: chọn CA cho hệ thống đã tạo trước đó (server-ca)
	* Server certificate: chọn cert cho server được tạo (server-cert)
	* Enable NCP: lựa chọn sử dụng mã hóa đường truyền giữa Client và Server, sử dụng các giải thuật mặc định là AES-256-GCM và AES-128-GCM
	* Auth digest algorithm: lựa chọn giải thuật xác thực kênh truyền là SHA256
![images](./images/pf-83.png)
- Khai báo các thông tin về tun
	* IPv4 Tunnel Network: khai báo network tunnel, VPN client sẽ được route tới Private LAN thông qua network này
	* IPv4 local Network: khai báo các dải Private LAN được truy cập thông qua VPN
	* Concurrent Connection: khai báo số lượng client được kết nối VPN đồng thời
![images](./images/pf-84.png)
- Click "Save" để tạo VPN Server
- Cấu hình Interface
	* Tại tab Interfaces/InterfaceAssignments, add thêm network port của VPN, đặt tên là vpn_lab_tun
![images](./images/pf-86.png)
![images](./images/pf-87.png)
- Cấu hình Firewall
	* Tại tab Firewall/Rules/WAN, add thêm rule cho phép client kết nối tới port 1195 của VPN. Khai báo các thông số như hình
![images](./images/pf-88.png)
	* Tại tab Firewall/Rules/LAN2, add rule cho phép lưu lượng đi qua
![images](./images/pf-89.png)
	* Tại tab Firewall/Rules/VPNLABTUN, add rule cho phép lưu lượng đi qua
![images](./images/pf-90.png)
	* Tại tab Firewall/Rules/OPENVPN, add rule cho phép lưu lượng đi qua
![images](./images/pf-91.png)
- Export OpenVPN config
	* Tại tab VPN/OpenVPN/ClientExport, khai báo các thông số:
		* Remote Access Server: lựa chọn OpenVPN server
		* Hostname Resolution: lựa chọn khai báo IP của WAN
![images](./images/pf-92.png)
![images](./images/pf-93.png)
- Kết nối VPN, nhập password của user anna, sau khi quay VPN thành công, client nhận IP của dải mạng LAN2 của pfSense là `10.10.60.2`
![images](./images/pf-93.png)
- Kiểm tra ping và truy cập `10.10.10.23` của `Ubuntu Server` nằm trong Private Lan
![images](./images/pf-96.png)

## References
1. [How to Install and Configure pfSense 2.1.5](https://www.tecmint.com/how-to-install-and-configure-pfsense/2/)
2. [pfSense remote access via OpenVPN](https://nguvu.org/pfsense/pfsense-inbound_vpn/)
3. [Cách cấu hình OpenVPN trên PFSense đơn giản nhất](https://vietnix.vn/cau-hinh-openvpn-tren-pfsense/)
4. [How to set up your own OpenVPN server in pfSense](https://www.comparitech.com/blog/vpn-privacy/openvpn-server-pfsense/)
