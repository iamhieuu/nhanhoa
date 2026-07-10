# 3: NHÀ CUNG CẤP CHỨNG CHỈ, XÁC MINH DOMAIN & VẬN HÀNH SSL

---

## MỤC LỤC

1. [Certificate Authority (CA) là gì?](#1-certificate-authority-ca-là-gì)
   - 1.1. [Vai trò của CA](#11-vai-trò-của-ca)
   - 1.2. [So sánh các CA phổ biến](#12-so-sánh-các-ca-phổ-biến)
2. [Let's Encrypt](#2-lets-encrypt)
   - 2.1. [Ưu điểm](#21-ưu-điểm)
   - 2.2. [Quy trình hoạt động (khái niệm)](#22-quy-trình-hoạt-động-khái-niệm)
   - 2.3. [Hạn chế](#23-hạn-chế)
3. [Các phương pháp xác minh Domain (ACME Challenge)](#3-các-phương-pháp-xác-minh-domain-acme-challenge)
   - 3.1. [HTTP-01 Challenge](#31-http-01-challenge)
   - 3.2. [DNS-01 Challenge](#32-dns-01-challenge)
   - 3.3. [TLS-ALPN-01 Challenge](#33-tls-alpn-01-challenge)
   - 3.4. [Bảng so sánh 3 phương pháp](#34-bảng-so-sánh-3-phương-pháp)
4. [CAA Record](#4-caa-record)
5. [HSTS (HTTP Strict Transport Security)](#5-hsts-http-strict-transport-security)
   - 5.1. [Vấn đề: SSL Stripping](#51-vấn-đề-ssl-stripping)
   - 5.2. [HSTS giải quyết vấn đề như thế nào](#52-hsts-giải-quyết-vấn-đề-như-thế-nào)
   - 5.3. [Các tham số trong header HSTS](#53-các-tham-số-trong-header-hsts)
   - 5.4. [HSTS Preload List](#54-hsts-preload-list)
6. [OCSP và OCSP Stapling](#6-ocsp-và-ocsp-stapling)
   - 6.1. [Vấn đề: xác minh chứng chỉ có bị thu hồi hay không](#61-vấn-đề-xác-minh-chứng-chỉ-có-bị-thu-hồi-hay-không)
   - 6.2. [OCSP là gì](#62-ocsp-là-gì)
   - 6.3. [OCSP Stapling – cải tiến của OCSP](#63-ocsp-stapling--cải-tiến-của-ocsp)
7. [Công cụ đánh giá cấu hình SSL](#7-công-cụ-đánh-giá-cấu-hình-ssl)
   - 7.1. [SSL Labs](#71-ssl-labs)
   - 7.2. [SSL Checker](#72-ssl-checker)
   - 7.3. [So sánh SSL Labs và SSL Checker](#73-so-sánh-ssl-labs-và-ssl-checker)
8. [Quy trình Audit SSL – 8 bước (khái niệm)](#8-quy-trình-audit-ssl--8-bước-khái-niệm)
9. [Các lỗi SSL thường gặp trong vận hành](#9-các-lỗi-ssl-thường-gặp-trong-vận-hành)
   - 9.1. [Mixed Content](#91-mixed-content)
   - 9.2. [ERR_CERT_COMMON_NAME_INVALID](#92-err_cert_common_name_invalid)
   - 9.3. [SSL Handshake Failure](#93-ssl-handshake-failure)
   - 9.4. [Expired Certificate](#94-expired-certificate)
   - 9.5. [Missing Intermediate Certificate](#95-missing-intermediate-certificate)


---

## 1. Certificate Authority (CA) là gì?

**Certificate Authority (CA)** là tổ chức được ủy quyền để cấp và xác thực chứng chỉ SSL/TLS.

### 1.1. Vai trò của CA

- Xác minh danh tính chủ sở hữu domain hoặc doanh nghiệp.
- Ký số lên chứng chỉ SSL để bảo lãnh cho tính xác thực của chứng chỉ đó.
- Giúp trình duyệt xác định website nào đáng tin cậy.

>  **Nếu không có CA:** Bất kỳ ai cũng có thể tự tạo một chứng chỉ giả mạo, và trình duyệt sẽ không có cách nào phân biệt website thật với website giả.
>
> **Khi có CA:** Trình duyệt chỉ tin tưởng các CA nằm trong Trusted Root Store. Chứng chỉ được một CA hợp lệ ký sẽ được chấp nhận; chứng chỉ không đáng tin cậy sẽ bị cảnh báo bảo mật ngay trên trình duyệt.

### 1.2. So sánh các CA phổ biến

| CA | Loại chứng chỉ | Chi phí | Thời hạn | Phù hợp |
| --- | --- | --- | --- | --- |
| **Let's Encrypt** | DV | Miễn phí | 90 ngày | Website thông thường, hosting |
| **ZeroSSL** | DV | Miễn phí (giới hạn) | 90 ngày | Giải pháp thay thế Let's Encrypt |
| **Sectigo** | DV, OV, EV, Wildcard | 10 – 500 USD | 1 – 2 năm | Doanh nghiệp |
| **DigiCert** | DV, OV, EV | 200 – 2000 USD | 1 – 2 năm | Enterprise, ngân hàng |
| **GlobalSign** | DV, OV, EV | 150 – 1000 USD | 1 – 2 năm | Enterprise |
| **Nhân Hòa SSL** | DV, OV, Wildcard | Liên hệ | 1 – 2 năm | Khách hàng Nhân Hòa |

---

## 2. Let's Encrypt

Let's Encrypt là một CA phi lợi nhuận, nổi bật với việc cấp chứng chỉ DV **hoàn toàn miễn phí và tự động**.

### 2.1. Ưu điểm

- Miễn phí hoàn toàn.
- Tự động hóa toàn bộ quy trình cấp phát chứng chỉ.
- Hỗ trợ tự động gia hạn.
- Được hầu hết trình duyệt hiện đại tin cậy.

### 2.2. Quy trình hoạt động (khái niệm)

1. Client (thường là công cụ ACME như Certbot) gửi yêu cầu cấp chứng chỉ tới Let's Encrypt.
2. Let's Encrypt gửi lại một "thử thách" (challenge) để xác minh quyền sở hữu domain.
3. Client thực hiện xác minh theo phương pháp challenge được chọn (xem Mục 3).
4. Sau khi xác minh thành công, chứng chỉ được cấp phát.

### 2.3. Hạn chế

- Chỉ hỗ trợ chứng chỉ mức **DV** — không cấp được OV hoặc EV.
- Thời hạn chứng chỉ ngắn: chỉ **90 ngày** (buộc phải có cơ chế tự động gia hạn).
- Có giới hạn số lượng chứng chỉ được cấp trong một khoảng thời gian nhất định (rate limit).
- Không phù hợp cho các hệ thống yêu cầu xác thực tổ chức ở mức OV hoặc EV (ví dụ ngân hàng, tài chính).

---

## 3. Các phương pháp xác minh Domain (ACME Challenge)

Trước khi cấp chứng chỉ, CA cần một cách để xác minh rằng bên yêu cầu thực sự sở hữu/kiểm soát domain đó. Đây là vai trò của các phương pháp **ACME Challenge**.

### 3.1. HTTP-01 Challenge

CA kiểm tra sự tồn tại của một file xác thực đặc biệt trên web server của domain, thông qua kết nối HTTP thông thường. Yêu cầu **port 80 phải mở** và website phải có địa chỉ public.

| Phù hợp | Không phù hợp |
| --- | --- |
| Website thông thường | Wildcard SSL |
| | Internal Server (không có IP public) |

### 3.2. DNS-01 Challenge

CA kiểm tra sự tồn tại của một bản ghi **TXT** trong DNS zone của domain, ví dụ:

```
_acme-challenge.congty.vn TXT "TOKEN"
```

| Phù hợp |
| --- |
| Wildcard SSL |
| Internal Server |
| Server không có IP public |

### 3.3. TLS-ALPN-01 Challenge

Xác minh thông qua chính kết nối TLS trên **port 443**, thường được dùng khi port 80 bị chặn hoặc không khả dụng.

### 3.4. Bảng so sánh 3 phương pháp

| Phương pháp | Ưu điểm | Nhược điểm |
| --- | --- | --- |
| **HTTP-01** | Cấu hình đơn giản, được hầu hết công cụ ACME (Certbot...) hỗ trợ tốt, phổ biến nhất | Yêu cầu mở port 80, không hỗ trợ Wildcard SSL |
| **DNS-01** | Hỗ trợ Wildcard SSL, không cần web server chạy, phù hợp cho Internal Server | Cấu hình DNS phức tạp hơn, cần chờ thời gian DNS propagate |
| **TLS-ALPN-01** | Không cần mở port 80, chỉ cần port 443 | Ít được hỗ trợ rộng rãi hơn, cấu hình phức tạp hơn HTTP-01 |

---

## 4. CAA Record

**CAA (Certification Authority Authorization)** là một loại bản ghi DNS quy định **CA nào được phép cấp SSL** cho một domain cụ thể.

**Ví dụ:**

```
congty.vn. CAA 0 issue "letsencrypt.org"
congty.vn. CAA 0 issue "sectigo.com"
```

**Lợi ích của CAA Record:**

- Ngăn chặn việc cấp SSL trái phép từ các CA không được ủy quyền.
- Tăng cường bảo mật cho domain.
- Giảm nguy cơ giả mạo website thông qua chứng chỉ cấp sai.

---

## 5. HSTS (HTTP Strict Transport Security)

### 5.1. Vấn đề: SSL Stripping

Redirect HTTP → HTTPS thông thường (mã 301) có một khoảng hở bảo mật: **request HTTP đầu tiên vẫn phải được gửi đi** trước khi server kịp trả về lệnh chuyển hướng. Trong khoảng thời gian cực ngắn đó, dữ liệu chưa được mã hóa — đây là cơ hội cho kẻ tấn công đứng giữa trên cùng mạng (ví dụ Wi-Fi công cộng) thực hiện kỹ thuật gọi là **SSL Stripping**: chặn request HTTP gốc trước khi nó đến được server thật, tự đóng vai server và trả lời bằng HTTP (không bao giờ để redirect 301 xảy ra), khiến trình duyệt nạn nhân tin rằng site chỉ có HTTP — từ đó dữ liệu nhạy cảm (mật khẩu, session) bị truyền không mã hóa và có thể bị đọc trộm.

### 5.2. HSTS giải quyết vấn đề như thế nào

**HSTS** giải quyết lỗ hổng này bằng cách: sau lần truy cập HTTPS đầu tiên, server gửi một header đặc biệt yêu cầu trình duyệt **tự ghi nhớ vĩnh viễn** rằng domain này chỉ được truy cập qua HTTPS. Từ đó, trình duyệt sẽ **không bao giờ gửi request HTTP ra mạng nữa** — nó tự động đổi sang HTTPS ngay tại tầng trình duyệt, trước khi bất kỳ gói tin nào rời khỏi máy người dùng.

>  **Khác biệt cốt lõi với redirect 301:** Redirect 301 xử lý ở tầng server, **sau khi** request đã ra mạng (vẫn còn khoảng hở HTTP ban đầu). HSTS xử lý ngay tại tầng trình duyệt, loại bỏ hoàn toàn khoảng hở đó.

### 5.3. Các tham số trong header HSTS

```
Strict-Transport-Security: max-age=31536000; includeSubDomains; preload
```

| Tham số | Ý nghĩa |
| --- | --- |
| `max-age=31536000` | Thời gian (tính bằng giây) trình duyệt sẽ ghi nhớ quy tắc này — 31.536.000 giây = 365 ngày |
| `includeSubDomains` | Áp dụng quy tắc HSTS cho **mọi subdomain** (`mail.congty.vn`, `shop.congty.vn`...), không chỉ riêng domain chính |
| `preload` | Đăng ký tham gia danh sách preload toàn cầu (xem Mục 5.4) |

### 5.4. HSTS Preload List

Vấn đề còn sót lại của HSTS thông thường: **lần truy cập đầu tiên tuyệt đối** vẫn phải đi qua HTTP trước khi trình duyệt nhận được header HSTS lần đầu tiên — đây vẫn là một khoảng hở nhỏ.

**HSTS Preload List** giải quyết triệt để vấn đề này: domain được đăng ký vào một danh sách trung tâm, danh sách này được **đóng gói cứng sẵn ngay trong trình duyệt**. Nhờ vậy, trình duyệt biết ngay từ lần truy cập đầu tiên rằng domain này bắt buộc phải dùng HTTPS, không cần chờ nhận header từ server.

>  **Lưu ý quan trọng:** Domain đã được đưa vào preload list **rất khó rút ra** — có thể mất nhiều tháng để các phiên bản trình duyệt cũ loại bỏ domain khỏi danh sách cứng đã cài sẵn. Vì vậy chỉ nên bật `preload` khi đã chắc chắn **toàn bộ domain và mọi subdomain** đều sẵn sàng chạy HTTPS vĩnh viễn.

---

## 6. OCSP và OCSP Stapling

### 6.1. Vấn đề: xác minh chứng chỉ có bị thu hồi hay không

Một chứng chỉ có thể bị **thu hồi (revoke)** trước khi hết hạn tự nhiên — ví dụ khi private key của nó bị lộ. Trình duyệt cần một cách để biết chứng chỉ đang sử dụng có bị thu hồi hay không, thay vì chỉ dựa vào ngày hết hạn (`Not After`).

### 6.2. OCSP là gì

**OCSP (Online Certificate Status Protocol)** là cơ chế để trình duyệt hỏi trực tiếp CA: "chứng chỉ này còn hợp lệ không?" — thay thế cho cơ chế cũ hơn là **CRL (Certificate Revocation List)**, vốn phải tải về cả một danh sách dài rồi tự tra cứu, rất nặng và kém hiệu quả.

```
Browser                                    CA's OCSP Server
   │                                              │
   │--- "Chứng chỉ ABC còn hợp lệ không?" ------->│
   │                                              │
   │<-- "Có" / "Không (đã bị thu hồi)" -----------│
   │
   ▼
Mới tiếp tục tải trang web
```

### 6.3. OCSP Stapling – cải tiến của OCSP

**OCSP Stapling** đảo ngược trách nhiệm kiểm tra: thay vì để **trình duyệt** tự đi hỏi CA (gây độ trễ và phụ thuộc vào tính khả dụng của OCSP server), **chính web server** sẽ định kỳ tự hỏi CA trước, lưu cache lại câu trả lời, rồi **đính kèm thẳng** câu trả lời đó (đã được CA ký) vào ngay trong quá trình TLS Handshake gửi cho trình duyệt.

```
Web Server                  CA's OCSP Server
    │                              │
    │--- Hỏi trước (định kỳ) ----->│
    │<-- OCSP Response (đã ký) ----│
    │   (cache lại trong vài giờ)
    │
    ▼
Browser                     Web Server
   │--- TLS Handshake -------------->│
   │<-- Certificate + OCSP Response--│
   │    (verify chữ ký CA tại đây, không cần tự gọi CA)
```

>  **Lợi ích của OCSP Stapling:** Giảm độ trễ cho người dùng cuối (không phải chờ trình duyệt tự kết nối tới OCSP server của CA), đồng thời giảm tải cho hạ tầng OCSP của CA.

---

## 7. Công cụ đánh giá cấu hình SSL

Trong vận hành thực tế, kỹ thuật viên cần một công cụ **tổng hợp toàn bộ** các tiêu chí (chain, cipher, protocol, HSTS, OCSP...) thành một bản đánh giá duy nhất, dễ trình bày cho khách hàng hoặc dùng để audit định kỳ. Đây là vai trò của SSL Labs và các SSL Checker.

### 7.1. SSL Labs

**SSL Labs** là công cụ kiểm tra SSL/TLS được công nhận rộng rãi nhất trong ngành, chấm điểm cấu hình SSL của một domain theo thang hạng chữ.

**Thang điểm SSL Rating:**

| Hạng | Ý nghĩa chung |
| --- | --- |
| **A+** | Cấu hình xuất sắc: chỉ dùng TLS 1.2/1.3, cipher mạnh, HSTS bật đúng, chain đầy đủ, OCSP Stapling hoạt động |
| **A** | Tốt, đạt chuẩn bảo mật hiện hành nhưng thiếu một số điểm cộng (ví dụ chưa bật HSTS) |
| **B** | Còn hỗ trợ giao thức/cipher cũ (ví dụ TLS 1.0/1.1) gây giảm điểm |
| **C** | Có vấn đề rõ ràng hơn: cipher yếu, chain thiếu sót, hoặc cấu hình không tối ưu |
| **F** | Lỗi nghiêm trọng: lỗ hổng đã biết (ví dụ Heartbleed cũ), chain hoàn toàn sai, hoặc cấu hình sai căn bản |



### 7.2. SSL Checker

Là nhóm công cụ nhẹ hơn SSL Labs, tập trung vào kiểm tra nhanh các thông tin cơ bản, không chấm điểm tổng thể chi tiết như SSL Labs. Thường được tích hợp sẵn trong nhiều đại lý bán SSL (kể cả Nhân Hòa thường có sẵn link kiểm tra nhanh sau khi khách hàng cài đặt chứng chỉ).

Các nhóm chức năng thường có:

| Chức năng | Mô tả |
| --- | --- |
| **Kiểm tra chứng chỉ** | Xác nhận chứng chỉ đang chạy đúng domain, đúng CA cấp, hiển thị thông tin Subject/Issuer ở dạng rút gọn, dễ đọc cho người không chuyên kỹ thuật |
| **Kiểm tra Chain** | Liệt kê trực quan từng chứng chỉ trong chain, cảnh báo rõ nếu thiếu Intermediate CA — công cụ nhanh nhất để xác nhận lỗi "Incomplete Chain" |
| **Kiểm tra ngày hết hạn** | Hiển thị rõ ngày hết hạn và số ngày còn lại — hữu ích khi cần theo dõi nhanh nhiều domain khách hàng cùng lúc |

### 7.3. So sánh SSL Labs và SSL Checker

| Tiêu chí | SSL Labs | SSL Checker (nhẹ) |
| --- | --- | --- |
| Độ chi tiết | Rất chi tiết, chấm điểm A+ → F | Cơ bản: chứng chỉ, chain, ngày hết hạn |
| Thời gian quét | 1 – 3 phút | Vài giây |
| Phù hợp | Audit định kỳ, demo cho khách hàng, kiểm tra trước go-live | Kiểm tra nhanh hằng ngày, theo dõi nhiều domain |
| Test với nhiều OS/Browser | Có (Handshake Simulation) | Không |

---

## 8. Quy trình Audit SSL – 8 bước (khái niệm)

Trong công việc thực tế của System Administrator, các thành phần kỹ thuật của SSL (khóa, CSR, định dạng, chain, redirect, HSTS, OCSP...) không được kiểm tra rời rạc, mà cần một **quy trình audit có thứ tự, lặp lại được**, áp dụng cho mọi domain trong hạ tầng — đặc biệt quan trọng khi quản lý nhiều domain khách hàng cùng lúc.

Quy trình dưới đây tổng hợp toàn bộ kiến thức thành một checklist hành động theo đúng thứ tự ưu tiên xử lý vấn đề: **bước 1–4 kiểm tra tính đúng đắn cấu trúc** của chứng chỉ, **bước 5–7 kiểm tra cấu hình vận hành**, **bước 8 là bước tổng hợp xác nhận** lại toàn bộ.

| # | Bước | Mục tiêu kiểm tra | Tiêu chí đạt (pass) |
| --- | --- | --- | --- |
| 1 | **Certificate** | Đúng domain, đúng SAN, còn hiệu lực, đúng Issuer | CN/SAN khớp đúng domain đang chạy, `Not After` còn hạn, Issuer là CA hợp lệ |
| 2 | **Chain** | Chain đầy đủ tới Root CA, không thiếu Intermediate | Server trả về đủ website cert + Intermediate CA, không báo lỗi "unable to verify" |
| 3 | **Cipher** | Không còn cipher yếu (3DES, RC4) | Danh sách cipher không chứa cipher gắn nhãn yếu/không an toàn |
| 4 | **TLS Version** | Đã loại bỏ SSL 2.0/3.0, TLS 1.0/1.1 | Chỉ TLS 1.2/1.3 kết nối thành công, các phiên bản cũ bị từ chối |
| 5 | **HSTS** | Header tồn tại và cấu hình đúng | Có header `Strict-Transport-Security` với `max-age` đủ dài |
| 6 | **OCSP** | OCSP Stapling đang hoạt động | Phản hồi OCSP xác nhận chứng chỉ hợp lệ (không phải "no response") |
| 7 | **Expiration** | Chứng chỉ chưa gần hết hạn, cơ chế tự động gia hạn hoạt động | Còn trên 15–30 ngày trước hết hạn (với Let's Encrypt 90 ngày, đây là ngưỡng cảnh báo cần theo dõi sát) |
| 8 | **SSL Labs** | Đánh giá tổng hợp lại toàn bộ 7 bước trên | Đạt hạng tối thiểu A, tốt nhất là A+ |

>   Khi audit hàng loạt domain trên một server hosting, có thể tự động hóa 7 bước đầu tiên để chạy lặp qua danh sách domain, chỉ những domain nào không đạt mới cần vào SSL Labs kiểm tra sâu — tránh phải chạy SSL Labs thủ công cho từng domain một.

---

## 9. Các lỗi SSL thường gặp trong vận hành

Trong môi trường production, các lỗi SSL/TLS là nguyên nhân hàng đầu gây gián đoạn dịch vụ và ảnh hưởng niềm tin khách hàng. Bảng dưới đây tổng hợp các nhóm lỗi phổ biến nhất theo cách tiếp cận **nhận diện → nguyên nhân → hướng xử lý**.

| Lỗi | Nguyên nhân chính |
| --- | --- |
| **Mixed Content** | Trang HTTPS tải tài nguyên (ảnh, script, CSS, iframe) qua HTTP |
| **ERR_CERT_COMMON_NAME_INVALID** | Domain người dùng truy cập không khớp với SAN trong chứng chỉ |
| **SSL Handshake Failure** | Cipher/protocol không tương thích giữa client và server, hoặc chứng chỉ có vấn đề |
| **Expired Certificate** | Chứng chỉ đã hết hạn do không được gia hạn kịp thời |
| **Missing Intermediate** | Server không gửi kèm Intermediate CA certificate |

### 9.1. Mixed Content

Xảy ra khi một trang HTTPS tải tài nguyên (ảnh, script, CSS, iframe...) qua đường dẫn HTTP thay vì HTTPS. Trình duyệt sẽ **chặn (block) hoặc cảnh báo (warn)** loại nội dung này, ảnh hưởng trực tiếp đến trải nghiệm người dùng (UX) và cả thứ hạng SEO.

> **Hướng xử lý khái niệm:** Rà soát toàn bộ tài nguyên được nhúng trong trang để đảm bảo tất cả đều trỏ về đường dẫn HTTPS; đồng thời cấu hình redirect HTTP → HTTPS ở tầng server và bật HSTS để ngăn tình trạng tái diễn.

### 9.2. ERR_CERT_COMMON_NAME_INVALID

Lỗi này xuất hiện khi domain trong chứng chỉ không khớp với domain mà người dùng đang truy cập. Ví dụ: chứng chỉ được cấp cho `example.com` nhưng người dùng lại truy cập `www.example.com` hoặc `sub.example.com` — trong khi các domain này không nằm trong danh sách SAN của chứng chỉ.

> **Hướng xử lý khái niệm:** Xác định đầy đủ danh sách domain/subdomain thực tế đang được sử dụng, sau đó yêu cầu cấp lại chứng chỉ có SAN bao phủ đầy đủ toàn bộ các domain đó.

### 9.3. SSL Handshake Failure

Đây là nhóm lỗi phức tạp nhất, có thể do nhiều nguyên nhân: cipher suite giữa client và server không khớp, phiên bản giao thức (protocol) không tương thích, chứng chỉ bị từ chối, cấu hình SNI sai, hoặc kết nối bị timeout.

> **Hướng xử lý khái niệm:** Cần chẩn đoán theo từng lớp — trước tiên xác nhận kết nối TLS cơ bản có thiết lập được không, sau đó kiểm tra xem phiên bản TLS và cipher suite của server có phù hợp với client hay không, và cuối cùng xác nhận chain chứng chỉ có đầy đủ hay không, vì thiếu Intermediate CA cũng có thể biểu hiện ra thành lỗi handshake.

### 9.4. Expired Certificate

Chứng chỉ đã hết hạn hiệu lực (quá ngày `Not After`) nhưng chưa được gia hạn kịp thời — nguyên nhân phổ biến nhất là do cơ chế tự động gia hạn (đặc biệt với Let's Encrypt, chu kỳ chỉ 90 ngày) không hoạt động đúng như kỳ vọng.

> **Hướng xử lý khái niệm:** Cần có cơ chế giám sát chủ động (theo dõi số ngày còn lại trước khi hết hạn) và xác nhận định kỳ rằng tiến trình gia hạn tự động đang hoạt động bình thường, thay vì chỉ phát hiện sự cố sau khi chứng chỉ đã hết hạn.

### 9.5. Missing Intermediate Certificate

Server chỉ cấu hình gửi chứng chỉ của website mà thiếu Intermediate CA certificate đi kèm — đây chính là lỗi "Incomplete Chain" đã đề cập ở Chuyên đề 2, Mục 7.3.

> **Hướng xử lý khái niệm:** Đảm bảo cấu hình server luôn gửi kèm đầy đủ chứng chỉ website và Intermediate CA (thường ghép chung thành một file "fullchain"), vì nhiều loại client (ứng dụng di động, thư viện Java, các công cụ dòng lệnh) không tự động bổ sung Intermediate CA còn thiếu như một số trình duyệt desktop hiện đại vẫn làm được.

---



