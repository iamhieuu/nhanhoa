# DNS

## Mục lục

- [1. DNS là gì?](#1-dns-là-gì)
- [2. Chức năng cốt lõi](#2-chức-năng-cốt-lõi)
- [3. Nguyên tắc làm việc](#3-nguyên-tắc-làm-việc)
- [4. Luồng phân giải DNS (resolution flow)](#4-luồng-phân-giải-dns-resolution-flow)
- [5. Bảo mật DNS — chuẩn 2026](#5-bảo-mật-dns--chuẩn-2026)
- [6. Các bản ghi DNS — Bảng tra cứu nhanh](#6-các-bản-ghi-dns--bảng-tra-cứu-nhanh)
  - [6.1 Bản ghi cốt lõi (Web & Mail)](#61-bản-ghi-cốt-lõi-web--mail)
  - [6.2 TXT record cho xác thực email — bộ ba SPF/DKIM/DMARC](#62-txt-record-cho-xác-thực-email--bộ-ba-spfdkimdmarc)
  - [6.3 Các bản ghi khác (ít dùng trong vận hành thường ngày)](#63-các-bản-ghi-khác-ít-dùng-trong-vận-hành-thường-ngày)
- [7. Chiến lược TTL — kinh nghiệm thực chiến](#7-chiến-lược-ttl--kinh-nghiệm-thực-chiến)
- [8. DNS Propagation — vì sao đổi DNS chưa có hiệu lực ngay](#8-dns-propagation--vì-sao-đổi-dns-chưa-có-hiệu-lực-ngay)
- [9. Zone file mẫu — tổng hợp các bản ghi trong 1 domain thật](#9-zone-file-mẫu--tổng-hợp-các-bản-ghi-trong-1-domain-thật)
- [10. Split-horizon DNS](#10-split-horizon-dns)
- [11. Thực hành với Bind9 (Linux)](#11-thực-hành-với-bind9-linux)

---

## 1. DNS là gì?

DNS (Domain Name System) là hệ thống phân giải qua lại giữa **tên miền** (dễ nhớ, dạng chữ — `example.com`) và **địa chỉ IP** (dạng số — `192.168.136.131`) mà máy móc dùng để định tuyến traffic.

Bản chất: một database phân tán, phân cấp, không tồn tại ở một điểm trung tâm duy nhất — mỗi domain có name server thẩm quyền riêng, tự chịu trách nhiệm cho zone của mình và có thể delegate tiếp cho sub-domain. Cơ chế này giúp DNS có khả năng chịu lỗi và scale tốt ở quy mô toàn cầu.

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
- DNS hoạt động theo cơ chế **truy vấn đệ quy/lặp** giữa các resolver, không có một trung tâm duy nhất xử lý mọi truy vấn.
- Mỗi DNS server có hai vai trò song song:
  - **Resolve outbound**: phân giải tên cho client bên trong miền nó quản lý cả tên trong lẫn ngoài miền.
  - **Authoritative response**: trả lời các resolver bên ngoài hỏi về domain nó quản lý.
- Resolver **cache** lại kết quả đã phân giải theo TTL của bản ghi để tối ưu cho các truy vấn lặp lại.
- **Glue record**: khi name server của 1 domain lại nằm chính trong domain đó (VD `ns1.example.com` là NS cho `example.com`), sẽ xảy ra vòng lặp tham chiếu (cần biết IP của `ns1.example.com` để hỏi nó, nhưng lại phải hỏi nó để biết IP). Glue record là bản ghi A được khai báo kèm ngay ở TLD server để phá vòng lặp này — cấu hình ở registrar, không phải ở zone file.

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

> **Lưu ý dễ bị bỏ sót khi bật DNSSEC:** nếu domain đổi DNS provider hoặc đổi record mà **quên cập nhật DS record ở registrar**, toàn bộ domain sẽ "sập" (SERVFAIL) vì chain-of-trust bị gãy — đây là một trong những nguyên nhân outage nghiêm trọng nhất liên quan DNS, luôn double-check DS record sau khi thao tác DNSSEC.
>
> **DNS Amplification Attack:** DNS server cấu hình `recursion` mở công khai (open resolver) có thể bị lợi dụng làm bàn đạp cho tấn công DDoS khuếch đại (do response DNS thường lớn hơn nhiều lần so với query). Với server Bind9 tự dựng, luôn giới hạn `allow-recursion` chỉ cho dải mạng nội bộ, không để mặc định `any`.

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

> **Kinh nghiệm thực chiến:** khi mới bật DMARC cho domain đã hoạt động lâu, luôn bắt đầu bằng policy `p=none` (chỉ nhận report, không chặn) trong 1–2 tuần để rà soát toàn bộ nguồn gửi mail hợp lệ (CRM, marketing tool, ticket system...) trước khi chuyển dần lên `p=quarantine` rồi `p=reject`. Bật thẳng `p=reject` ngay từ đầu là nguyên nhân phổ biến khiến mail hệ thống nội bộ (không phải spam) bị chặn oan.

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

## 7. Chiến lược TTL — kinh nghiệm thực chiến

TTL (Time To Live) quyết định resolver sẽ **cache** bản ghi trong bao lâu trước khi hỏi lại authoritative server. Chọn TTL sai là nguyên nhân phổ biến gây downtime kéo dài khi migrate server.

| Tình huống | TTL khuyến nghị | Lý do |
|---|---|---|
| Domain hoạt động ổn định, ít đổi | 3600–14400s (1–4 giờ) | Giảm tải query, tăng tốc độ phân giải cho user |
| **Trước khi migrate server / đổi IP** (làm 24–48h trước) | Hạ xuống 300s (5 phút) | Khi đổi IP thật, resolver cũ hết hạn cache nhanh, giảm thời gian downtime cảm nhận |
| Sau khi migrate xong, đã ổn định | Tăng dần lại về 3600s+ | Cache lâu hơn để tối ưu hiệu năng, giảm tải DNS server |
| Bản ghi MX (mail) | 3600s+ | Mail ít khi đổi server đột ngột, ưu tiên ổn định |
| Bản ghi dùng cho failover tự động (multi-CDN, HA) | 60–300s | Cần cache ngắn để failover có hiệu lực nhanh khi 1 điểm chết |

> **Lưu ý:** hạ TTL **phải làm trước** thời điểm đổi bản ghi ít nhất bằng đúng giá trị TTL cũ (VD TTL cũ đang 86400s thì phải hạ trước tối thiểu 24h), vì bản thân giá trị TTL cũ vẫn còn hiệu lực cache ở các resolver trên toàn cầu cho đến khi nó tự hết hạn.

---

## 8. DNS Propagation — vì sao đổi DNS chưa có hiệu lực ngay

"Propagation" không phải do DNS "lan truyền chậm" theo nghĩa vật lý — mà do **hàng nghìn resolver trên thế giới cache bản ghi cũ theo TTL riêng của từng resolver**, và chỉ query lại authoritative server khi cache hết hạn.

**Thời gian thực tế phụ thuộc vào:**
- TTL của bản ghi cũ (xem mục 7) — yếu tố quyết định chính.
- Một số ISP/resolver **không tuân thủ đúng TTL** (cache lâu hơn tuyên bố) — hay gặp ở ISP tại Việt Nam.
- Registrar propagation (khi đổi **NS record** ở tầng registrar) có thể mất 24–48h vì phải chờ TLD server cập nhật, không kiểm soát được bằng TTL.

**Công cụ kiểm tra thực tế khi hỗ trợ khách hàng:**
```bash
# Tra cứu từ nhiều resolver khác nhau trên thế giới
# (khuyến nghị dùng công cụ web: dnschecker.org, whatsmydns.net)

# Kiểm tra từ chính server, so sánh với resolver public
dig domain.com @8.8.8.8 +short
dig domain.com @1.1.1.1 +short
dig domain.com @ns1.nhanhoa.com +short   # hỏi thẳng authoritative NS, luôn ra kết quả mới nhất
```

> **Mẹo xử lý ticket "tôi đổi DNS rồi mà web vẫn ra IP cũ":** luôn `dig` thẳng vào **authoritative NS** trước để xác nhận bản ghi mới đã đúng chưa. Nếu đúng rồi thì đó chỉ là vấn đề cache phía resolver của khách hàng — hướng dẫn họ flush cache local hoặc đợi hết TTL, không phải lỗi từ phía DNS server.

---

## 9. Zone file mẫu — tổng hợp các bản ghi trong 1 domain thật

Ví dụ zone file Bind9 hoàn chỉnh cho domain `example.com`, gộp các loại bản ghi đã học ở mục 6:

```bind
$TTL 3600
@   IN  SOA  ns1.example.com. admin.example.com. (
            2026071001  ; Serial (thường đặt theo YYYYMMDDnn)
            3600        ; Refresh
            900         ; Retry
            1209600     ; Expire
            3600 )      ; Minimum TTL

; Name Server
@       IN  NS      ns1.example.com.
@       IN  NS      ns2.example.com.

; A record — web
@       IN  A       103.159.51.228
www     IN  A       103.159.51.228

; Mail
@       IN  MX  10  mail.example.com.
mail    IN  A       103.159.51.230

; TXT — SPF
@                   IN  TXT  "v=spf1 mx a ip4:103.159.51.230 ~all"

; TXT — DKIM (selector "default")
default._domainkey  IN  TXT  "v=DKIM1; k=rsa; p=MIGfMA0GCSq..."

; TXT — DMARC
_dmarc              IN  TXT  "v=DMARC1; p=quarantine; rua=mailto:dmarc@example.com"

; CAA — chỉ cho Let's Encrypt cấp SSL
@       IN  CAA  0 issue "letsencrypt.org"

; PTR khai báo ở zone reverse riêng (in-addr.arpa), không nằm chung file này
```

---

## 10. Split-horizon DNS

**Split-horizon DNS** (hay split-brain DNS) là kỹ thuật trả về **kết quả phân giải khác nhau** cho cùng một domain, tùy theo nguồn truy vấn đến từ mạng nội bộ hay internet công cộng.

**Use case thực tế:**
- `app.company.com` khi truy vấn từ nhân viên trong mạng LAN công ty → trả về IP nội bộ `192.168.x.x` (đi thẳng qua LAN, nhanh hơn, không qua internet).
- Cùng domain đó khi truy vấn từ internet bên ngoài → trả về IP public thật của server.

**Cách triển khai với Bind9:** dùng `view` trong `named.conf`, phân theo dải IP nguồn truy vấn:

```bind
view "internal" {
    match-clients { 192.168.254.0/24; };
    zone "company.com" {
        type master;
        file "/etc/bind/db.company.com.internal";
    };
};

view "external" {
    match-clients { any; };
    zone "company.com" {
        type master;
        file "/etc/bind/db.company.com.external";
    };
};
```

> Lưu ý: `view` nội bộ phải khai **trước** `view` external trong file cấu hình — Bind9 xét theo thứ tự từ trên xuống, khớp view đầu tiên thỏa điều kiện là dừng.

---



## 11. Thực hành với Bind9 (Linux)
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

> **Bổ sung kiểm tra sau cùng khuyến nghị:** ngoài `dig`, nên chạy thêm `named-checkzone` để validate cú pháp zone file trước khi reload, tránh lỗi khiến Bind9 crash toàn bộ zone:
> ```bash
> named-checkzone domain.com /etc/bind/db.domain.com
> named-checkconf   # validate named.conf trước khi reload
> systemctl reload bind9
> ```

---
