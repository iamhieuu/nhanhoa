# BÁO CÁO THỰC TẬP NGÀY 02
# DNS — Tài liệu kỹ thuật nội bộ 

---

## 1. DNS là gì?

DNS (Domain Name System) là hệ thống phân giải qua lại giữa **tên miền** (dễ nhớ, dạng chữ — `example.com`) và **địa chỉ IP** (dạng số — `123.11.5.19`) mà máy móc dùng để định tuyến traffic.

Bản chất: một **database phân tán, phân cấp** (distributed hierarchical database), không tồn tại ở một điểm trung tâm duy nhất — mỗi domain có name server thẩm quyền (authoritative) riêng, tự chịu trách nhiệm cho zone của mình và có thể delegate tiếp cho sub-domain. Cơ chế này giúp DNS có khả năng chịu lỗi (fault-tolerant) và scale tốt ở quy mô toàn cầu.

Ngoài phân giải A/AAAA, DNS còn lưu các loại thông tin khác: mail server (MX), bản ghi xác thực bảo mật (TXT — SPF/DKIM/DMARC), dịch vụ (SRV)...

---

## 2. Chức năng cốt lõi

| Chức năng | Mô tả ngắn |
|---|---|
| **Phân giải tên miền** | Domain ↔ IP, để user không cần nhớ IP |
| **Quản lý domain** | Đăng ký, cập nhật, hủy bản ghi qua DNS zone |
| **Thông tin bổ sung** | MX (mail), SRV (service discovery), TXT (xác thực/bảo mật) |
| **Cache & tăng tốc** | Resolver cache kết quả theo TTL, giảm round-trip |
| **Load balancing / HA** | Nhiều bản ghi A/AAAA cho cùng 1 domain → round-robin, failover |
| **Bảo mật** | DNSSEC chống giả mạo response; DoH/DoT mã hóa query (chi tiết mục 5) |

---

## 3. Nguyên tắc làm việc

- Mỗi tổ chức/ISP vận hành DNS server riêng. DNS server thẩm quyền cho 1 domain **luôn** thuộc tổ chức quản lý domain đó, không phải của bên thứ ba.
- DNS hoạt động theo cơ chế **truy vấn đệ quy/lặp** (recursive/iterative query) giữa các resolver, không có một "trung tâm" duy nhất xử lý mọi truy vấn.
- Mỗi DNS server có hai vai trò song song:
  - **Resolve outbound**: phân giải tên cho client bên trong miền nó quản lý (cả tên trong lẫn ngoài miền).
  - **Authoritative response**: trả lời các resolver bên ngoài hỏi về domain nó quản lý.
- Resolver **cache** lại kết quả đã phân giải theo TTL của bản ghi để tối ưu cho các truy vấn lặp lại.



---

## 4. Luồng phân giải DNS (resolution flow)

```
Client → Resolver (ISP/8.8.8.8/1.1.1.1)
       → Root server        (xác định TLD server, vd .com)
       → TLD server         (xác định authoritative NS của domain)
       → Authoritative NS   (trả về bản ghi thực tế: A/CNAME/MX...)
       → Resolver cache kết quả theo TTL → trả về Client
```

**Các bước:**
1. **Check local cache** — OS/browser cache trước, có thì trả ngay.
2. **Query resolver** (thường là DNS của ISP hoặc public resolver như 8.8.8.8, 1.1.1.1).
3. **Resolver query đệ quy theo cấp**: Root → TLD (.com/.vn...) → Authoritative NS của domain.
4. **Authoritative NS trả bản ghi** tương ứng (A, CNAME, MX...).
5. **Resolver cache kết quả** (theo TTL) và trả về client.
6. Client dùng IP nhận được để kết nối thẳng đến server đích — DNS không tham gia vào bước này.

---

## 5. Bảo mật DNS — chuẩn 2026

DNS truyền thống (port 53/UDP) **không mã hóa** → dễ bị nghe lén, can thiệp (DNS spoofing, cache poisoning, ISP tracking). Ba lớp bảo mật cần nắm khi vận hành:

| Công nghệ | Bảo vệ gì | Cơ chế | Ghi chú vận hành |
|---|---|---|---|
| **DNSSEC** | Chống giả mạo/đầu độc response | Ký số (digital signature) chuỗi bản ghi bằng RRSIG/DNSKEY, resolver verify chain-of-trust từ root xuống | Bảo vệ **tính toàn vẹn** dữ liệu, **không mã hóa** query. Bắt buộc bật cho domain doanh nghiệp/tài chính |
| **DoT (DNS over TLS)** | Mã hóa kênh truyền | Query DNS được bọc trong TLS, chạy trên port riêng (853) | Dễ nhận diện & chặn ở firewall vì dùng port cố định |
| **DoH (DNS over HTTPS)** | Mã hóa kênh truyền | Query DNS gửi qua HTTPS (port 443), lẫn vào traffic web bình thường | Khó chặn/giám sát hơn DoT vì dùng chung port với HTTPS — cần lưu ý khi viết policy firewall nội bộ |

**Khuyến nghị thực chiến:**
- Domain production (web/mail) → bật **DNSSEC** ở registrar/DNS provider (Cloudflare, Nhân Hòa DNS...).
- Endpoint/client nội bộ → cân nhắc resolver hỗ trợ **DoH/DoT** (Cloudflare 1.1.1.1, Google 8.8.8.8) để chống MITM trên mạng không tin cậy.
- DNSSEC và DoH/DoT là 2 lớp **độc lập, bổ trợ nhau** — không thay thế nhau (một bảo vệ data integrity, một bảo vệ transport).

---

## 6. Các bản ghi DNS — Bảng tra cứu nhanh

### 6.1 Bản ghi cốt lõi (Web & Mail)

| Loại | Tên đầy đủ | Chức năng thực tế |
|---|---|---|
| **A** | Address | Trỏ domain → IPv4. Bản ghi cơ bản nhất, dùng cho web server |
| **AAAA** | IPv6 Address | Trỏ domain → IPv6 |
| **CNAME** | Canonical Name | Alias domain này → domain khác (vd `www` → `example.com`). Không dùng chung được với bản ghi khác trên cùng hostname (vd MX) |
| **NS** | Name Server | Khai báo server thẩm quyền (authoritative) cho zone/domain |
| **SOA** | Start of Authority | Metadata của zone: primary NS, email admin, serial number, refresh/retry/expire/TTL — bắt buộc có 1 bản ghi/zone |
| **PTR** | Pointer (reverse lookup) | Ngược lại với A: IP → domain. Quan trọng cho mail server (reverse DNS check chống spam) |
| **MX** | Mail Exchanger | Chỉ định mail server nhận email cho domain, có priority — domain luôn nên có ≥1 MX backup |
| **SRV** | Service | Khai báo host + **port** cho 1 service cụ thể (vd SIP, VoIP, Minecraft server...) |
| **TXT** | Text | Lưu chuỗi text tùy ý — nền tảng cho xác thực email (xem bảng 6.2) và domain verification |

### 6.2 TXT record cho xác thực email — bộ ba SPF/DKIM/DMARC

| Bản ghi | Mục đích | Đặt ở đâu |
|---|---|---|
| **SPF** | Khai báo IP/server nào **được phép** gửi mail thay mặt domain | TXT tại root domain |
| **DKIM** | Ký số mỗi email gửi đi, mail nhận verify để xác minh không bị sửa nội dung trên đường truyền | TXT tại subdomain selector (vd `default._domainkey`) |
| **DMARC** | Policy: mail fail SPF/DKIM thì xử lý thế nào (reject/quarantine/none) + report về địa chỉ giám sát | TXT tại `_dmarc.domain.com` |

> Thiếu 1 trong 3 → email dễ vào spam hoặc bị giả mạo (spoofing) gửi thay domain.

### 6.3 Các bản ghi khác (ít dùng trong vận hành thường ngày)

| Bản ghi | Công dụng ngắn |
|---|---|
| **CAA** | Chỉ định CA nào được phép cấp SSL cho domain — nên cấu hình để chống cấp chứng chỉ trái phép |
| **DNSKEY / RRSIG / NSEC** | Thành phần của DNSSEC: public key, chữ ký số, chứng minh bản ghi không tồn tại |
| **DNAME** | Giống CNAME nhưng redirect cả cây sub-domain |
| **NAPTR** | Kết hợp SRV, tạo URI động bằng regex (VoIP/SIP) |
| **LOC** | Tọa độ địa lý của domain |
| **SSHFP** | Lưu fingerprint SSH public key, hỗ trợ verify host khi SSH |
| **HINFO, AFSDB, APL, HIP, IPSECKEY, CERT, CDNSKEY, DCHID, RP** | Legacy/niche, gần như không gặp trong vận hành hosting thông thường |

---


## 7. Thực hành với Bind9 (Linux)
Trong quá trình cấu hình DNS Server trên môi trường Linux Ubuntu server 22.04, file cấu hình chính thường nằm tại `/etc/bind/named.conf`.
#### Bước 1 : cài đặt bind9, dnsutils, bind9utils
<img width="335" height="67" alt="image" src="https://github.com/user-attachments/assets/72e1dccf-3ed7-424c-bec5-00c5f1edbb40" />

#### Bước 2 : khai báo tên miền /etc/named.conf.local
<img width="441" height="190" alt="image" src="https://github.com/user-attachments/assets/3b2fb5c9-3547-4714-bf1e-18fcbdb6e9a8" />  

#### Bước 3 : Tạo bản ghi DNS
<img width="436" height="206" alt="image" src="https://github.com/user-attachments/assets/9ed28b91-830e-4e3e-a226-2b5111cfae41" />   

#### Bước 4: Check
<img width="493" height="56" alt="image" src="https://github.com/user-attachments/assets/252282d9-dc76-45eb-882d-808c2423b24b" />  

#### Bước 5: Mở tường lửa, restart và check dịch vụ
<img width="746" height="346" alt="image" src="https://github.com/user-attachments/assets/6a6abdb9-4f9d-412b-b13e-0f4ac0cf76d8" />  

#### Bước 6 : check dig
<img width="460" height="257" alt="image" src="https://github.com/user-attachments/assets/23a07f1d-4c51-467f-aa0b-8db15aaaeab1" />

DNS đã hoạt động thành công


---
