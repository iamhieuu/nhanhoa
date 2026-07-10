# SSL/TLS CHUYÊN ĐỀ 1: ĐỊNH NGHĨA SSL, TLS, HTTPS VÀ NGUYÊN LÝ HOẠT ĐỘNG

> **Loại tài liệu:** Tài liệu định nghĩa chuyên đề – dùng cho đào tạo nội bộ (seminar intern) và báo cáo thực tập
> **Phạm vi:** Khái niệm nền tảng – không bao gồm hướng dẫn cài đặt hoặc thao tác dòng lệnh
> **Nguồn tham khảo:** Tổng hợp từ tài liệu thực tập ngày 42 (SSL Termination) và DigiCert – *What is SSL, TLS and HTTPS?*

---

## MỤC LỤC

1. [SSL là gì? TLS là gì?](#1-ssl-là-gì-tls-là-gì)
   - 1.1. [Định nghĩa SSL](#11-định-nghĩa-ssl)
   - 1.2. [Định nghĩa TLS](#12-định-nghĩa-tls)
   - 1.3. [Mối quan hệ giữa SSL và TLS](#13-mối-quan-hệ-giữa-ssl-và-tls)
   - 1.4. [Lịch sử và các phiên bản](#14-lịch-sử-và-các-phiên-bản)
2. [HTTPS là gì?](#2-https-là-gì)
   - 2.1. [Định nghĩa HTTPS](#21-định-nghĩa-https)
   - 2.2. [Mối quan hệ giữa HTTP, SSL/TLS và HTTPS](#22-mối-quan-hệ-giữa-http-ssltls-và-https)
3. [Vì sao cần SSL/TLS?](#3-vì-sao-cần-ssltls)
   - 3.1. [Các nhóm lợi ích chính](#31-các-nhóm-lợi-ích-chính)
   - 3.2. [SSL và SEO / niềm tin người dùng](#32-ssl-và-seo--niềm-tin-người-dùng)
   - 3.3. [Ba trụ cột bảo mật mà SSL/TLS cung cấp](#33-ba-trụ-cột-bảo-mật-mà-ssltls-cung-cấp)
4. [Nguyên lý mã hóa dùng trong SSL/TLS](#4-nguyên-lý-mã-hóa-dùng-trong-ssltls)
   - 4.1. [Mã hóa đối xứng (Symmetric Encryption)](#41-mã-hóa-đối-xứng-symmetric-encryption)
   - 4.2. [Mã hóa bất đối xứng (Asymmetric Encryption)](#42-mã-hóa-bất-đối-xứng-asymmetric-encryption)
   - 4.3. [Vì sao SSL/TLS kết hợp cả hai loại mã hóa](#43-vì-sao-ssltls-kết-hợp-cả-hai-loại-mã-hóa)
5. [TLS Handshake – Quy trình bắt tay thiết lập kết nối an toàn](#5-tls-handshake--quy-trình-bắt-tay-thiết-lập-kết-nối-an-toàn)
   - 5.1. [Mục tiêu của TLS Handshake](#51-mục-tiêu-của-tls-handshake)
   - 5.2. [Các giai đoạn chính (khái niệm)](#52-các-giai-đoạn-chính-khái-niệm)
6. [Public Key và Private Key](#6-public-key-và-private-key)
   - 6.1. [Định nghĩa](#61-định-nghĩa)
   - 6.2. [Vai trò trong SSL/TLS](#62-vai-trò-trong-ssltls)
   - 6.3. [So sánh RSA và ECC](#63-so-sánh-rsa-và-ecc)
7. [Thuật toán mã hóa và trao đổi khóa](#7-thuật-toán-mã-hóa-và-trao-đổi-khóa)
   - 7.1. [Thuật toán đối xứng phổ biến](#71-thuật-toán-đối-xứng-phổ-biến)
   - 7.2. [Thuật toán trao đổi khóa phổ biến](#72-thuật-toán-trao-đổi-khóa-phổ-biến)
8. [Bảng thuật ngữ (Glossary)](#8-bảng-thuật-ngữ-glossary)
9. [Tài liệu tham khảo](#9-tài-liệu-tham-khảo)
---

## 1. SSL là gì? TLS là gì?

### 1.1. Định nghĩa SSL

**SSL (Secure Sockets Layer)** là một giao thức chuẩn dùng để bảo mật kết nối Internet bằng cách mã hóa dữ liệu được truyền giữa một website và trình duyệt hoặc giữa hai server với nhau. Mục tiêu chính của SSL là ngăn chặn hacker xem hoặc đánh cắp bất kỳ thông tin nào được truyền đi, bao gồm cả dữ liệu cá nhân hoặc tài chính.

>   Chứng chỉ SSL vẫn là cách gọi phổ biến nhất, nhưng về mặt kỹ thuật, các chứng chỉ hiện đại được các CA cấp ngày nay thực chất đều là **chứng chỉ TLS** – SSL với tư cách giao thức gần như không còn được sử dụng trong thực tế do đã lỗi thời và mất an toàn.

### 1.2. Định nghĩa TLS

**TLS (Transport Layer Security)** là phiên bản kế thừa, cập nhật và bảo mật hơn của SSL. TLS giữ nguyên mục tiêu của SSL (mã hóa, xác thực dữ liệu truyền qua mạng) nhưng khắc phục các lỗ hổng bảo mật đã được phát hiện trong các phiên bản SSL cũ.

### 1.3. Mối quan hệ giữa SSL và TLS

| Khía cạnh | SSL | TLS |
| --- | --- | --- |
| Vai trò | Giao thức tiền nhiệm | Giao thức kế thừa, đang được sử dụng thực tế |
| Tình trạng sử dụng hiện tại | Đã ngừng sử dụng do lỗ hổng bảo mật | Đang là chuẩn bắt buộc (TLS 1.2, TLS 1.3) |
| Cách gọi trong ngành | Vẫn quen gọi là "SSL certificate" | Về bản chất kỹ thuật, đây là "TLS certificate" |

### 1.4. Lịch sử và các phiên bản

| Phiên bản | Tình trạng |
| --- | --- |
| SSL 1.0 | Không bao giờ được phát hành công khai (lỗi bảo mật nghiêm trọng ngay từ thiết kế) |
| SSL 2.0 | Đã bị loại bỏ hoàn toàn do nhiều lỗ hổng |
| SSL 3.0 | Đã bị loại bỏ hoàn toàn |
| TLS 1.0 / TLS 1.1 | Lỗi thời, không còn được khuyến nghị sử dụng trong production |
| TLS 1.2 | Vẫn được sử dụng rộng rãi, đạt chuẩn bảo mật hiện hành |
| TLS 1.3 | Phiên bản mới nhất, tối ưu tốc độ handshake và bảo mật cao nhất hiện nay |

---

## 2. HTTPS là gì?

### 2.1. Định nghĩa HTTPS

**HTTPS (HyperText Transfer Protocol Secure)** xuất hiện trên thanh địa chỉ URL khi một website được bảo mật bằng chứng chỉ SSL/TLS. Người dùng có thể xem chi tiết chứng chỉ – bao gồm CA cấp chứng chỉ và tên tổ chức sở hữu website – bằng cách nhấn vào biểu tượng ổ khóa trên thanh trình duyệt.

### 2.2. Mối quan hệ giữa HTTP, SSL/TLS và HTTPS
 **HTTPS = HTTP +  SSL/TLS**  

HTTP là giao thức truyền tải nội dung web ở dạng thuần văn bản (plaintext), dễ bị nghe lén hoặc chỉnh sửa giữa đường (man-in-the-middle). Khi kết hợp với SSL/TLS, toàn bộ dữ liệu trao đổi được mã hóa trước khi truyền đi, tạo thành HTTPS.

---

## 3. Vì sao cần SSL/TLS?

### 3.1. Các nhóm lợi ích chính

SSL không chỉ dành riêng cho các website thương mại điện tử – nó bảo vệ mọi loại thông tin được truyền đến và đi từ website:

| Nhóm | Lý do cần SSL |
| --- | --- |
| **Trang thanh toán (Checkout)** | Khách hàng có xu hướng hoàn tất giao dịch nhiều hơn khi biết khu vực thanh toán và thông tin thẻ được bảo mật |
| **Form đăng nhập & biểu mẫu** | SSL mã hóa và bảo vệ username, password, cũng như các form gửi thông tin cá nhân, tài liệu, hình ảnh |
| **Blog & trang thông tin** | Ngay cả website không thu thập thanh toán hay dữ liệu nhạy cảm vẫn cần HTTPS để giữ riêng tư hoạt động của người dùng |

### 3.2. SSL và SEO / niềm tin người dùng

Từ năm 2014, Google đã kêu gọi triển khai HTTPS trên toàn bộ web nhằm cải thiện bảo mật, và thưởng thứ hạng tìm kiếm cao hơn cho các website có SSL. Đến năm 2018, Google tiến xa hơn khi đánh dấu cảnh báo "Not secure" cho các website không có SSL ngay trên trình duyệt Chrome.

### 3.3. Ba trụ cột bảo mật mà SSL/TLS cung cấp

Theo DigiCert, một phiên làm việc HTTPS an toàn được thiết lập qua 3 giai đoạn:

| Giai đoạn | Nội dung |
| --- | --- |
| **1. Authentication (Xác thực)** | Mỗi phiên làm việc mới, trình duyệt và server trao đổi và xác thực chứng chỉ SSL của nhau |
| **2. Encryption (Mã hóa)** | Server chia sẻ public key cho trình duyệt; trình duyệt dùng public key này để tạo và mã hóa một pre-master key – đây gọi là bước trao đổi khóa (key exchange) |
| **3. Decryption (Giải mã)** | Server giải mã pre-master key bằng private key của mình, từ đó thiết lập kết nối mã hóa an toàn cho toàn bộ phiên làm việc |

---

## 4. Nguyên lý mã hóa dùng trong SSL/TLS

### 4.1. Mã hóa đối xứng (Symmetric Encryption)

Là phương pháp mã hóa trong đó **cùng một khóa** được dùng cho cả quá trình mã hóa và giải mã dữ liệu.

- **Ưu điểm:** Tốc độ xử lý nhanh, phù hợp mã hóa khối lượng dữ liệu lớn.
- **Nhược điểm:** Cả hai bên phải cùng sở hữu khóa bí mật giống nhau — nếu khóa bị lộ trong quá trình trao đổi, toàn bộ dữ liệu có nguy cơ bị giải mã.

### 4.2. Mã hóa bất đối xứng (Asymmetric Encryption)

Là phương pháp mã hóa sử dụng **một cặp khóa** gồm public key (khóa công khai) và private key (khóa riêng tư): dữ liệu mã hóa bằng public key chỉ có thể giải mã bằng private key tương ứng, và ngược lại.

- **Ưu điểm:** Không cần trao đổi khóa bí mật qua kênh không an toàn — public key có thể chia sẻ công khai mà không lo rò rỉ.
- **Nhược điểm:** Tốc độ xử lý chậm hơn nhiều so với mã hóa đối xứng, không phù hợp mã hóa khối lượng dữ liệu lớn.

### 4.3. Vì sao SSL/TLS kết hợp cả hai loại mã hóa

SSL/TLS tận dụng ưu điểm của cả hai phương pháp:

1. Dùng **mã hóa bất đối xứng** trong bước TLS Handshake để trao đổi an toàn một khóa phiên (session key) mà không cần lộ khóa bí mật qua mạng.
2. Sau khi đã có khóa phiên, chuyển sang dùng **mã hóa đối xứng** để mã hóa toàn bộ dữ liệu thực tế trong suốt phiên làm việc, nhờ tốc độ xử lý nhanh hơn.

> Đây chính là lý do SSL/TLS vừa an toàn  vừa nhanh.

---

## 5. TLS Handshake – Quy trình bắt tay thiết lập kết nối an toàn

### 5.1. Mục tiêu của TLS Handshake

TLS Handshake là quy trình đàm phán được thực hiện **trước khi** dữ liệu thực tế được truyền đi, nhằm:

- Xác thực danh tính của server (và tùy chọn cả client).
- Thống nhất phiên bản giao thức TLS và bộ cipher suite sẽ sử dụng.
- Tạo ra khóa phiên (session key) dùng chung để mã hóa đối xứng cho toàn bộ phiên làm việc.

### 5.2. Các giai đoạn chính 

| Giai đoạn | Nội dung khái quát |
| --- | --- |
| **Client Hello** | Trình duyệt gửi yêu cầu kết nối, kèm theo danh sách phiên bản TLS và cipher suite mà nó hỗ trợ |
| **Server Hello + gửi chứng chỉ** | Server phản hồi, chọn phiên bản TLS/cipher phù hợp, và gửi chứng chỉ SSL/TLS của mình (chứa public key) |
| **Xác thực chứng chỉ** | Trình duyệt kiểm tra chứng chỉ server có hợp lệ, còn hạn, và được CA đáng tin cậy cấp hay không |
| **Trao đổi khóa (Key Exchange)** | Hai bên phối hợp tạo ra pre-master secret / session key bằng thuật toán trao đổi khóa đã thống nhất |
| **Hoàn tất bắt tay** | Cả hai bên xác nhận đã sẵn sàng, bắt đầu truyền dữ liệu thực tế bằng mã hóa đối xứng với khóa phiên vừa tạo |


---

## 6. Public Key và Private Key

### 6.1. Định nghĩa

- **Public Key (Khóa công khai):** Có thể chia sẻ tự do, dùng để mã hóa dữ liệu hoặc xác minh chữ ký số. Public key được nhúng trong chứng chỉ SSL/TLS và gửi cho bất kỳ ai kết nối tới server.
- **Private Key (Khóa riêng tư):** Phải được giữ bí mật tuyệt đối, chỉ server sở hữu, dùng để giải mã dữ liệu đã được mã hóa bằng public key tương ứng, hoặc để tạo chữ ký số.

### 6.2. Vai trò trong SSL/TLS

Cặp khóa Public/Private là nền tảng của mã hóa bất đối xứng, được sử dụng trong bước TLS Handshake để trao đổi khóa phiên một cách an toàn mà không cần truyền private key qua mạng.

> Nếu private key bị mất hoặc lộ, chứng chỉ tương ứng phải được thu hồi và cấp lại, không có cách nào khôi phục private key gốc.

### 6.3. So sánh RSA và ECC

| Tiêu chí | RSA | ECC (Elliptic Curve Cryptography) |
| --- | --- | --- |
| Nguyên lý toán học | Dựa trên độ khó phân tích thừa số nguyên tố của số lớn | Dựa trên bài toán logarit rời rạc trên đường cong elliptic |
| Độ dài khóa để đạt cùng mức bảo mật | Dài hơn (ví dụ 2048-bit, 3072-bit) | Ngắn hơn đáng kể cho cùng mức bảo mật (ví dụ 256-bit ECC ≈ 3072-bit RSA) |
| Tốc độ xử lý | Chậm hơn với khóa dài | Nhanh hơn, tốn ít tài nguyên tính toán hơn |
| Mức độ phổ biến | Rất phổ biến, tương thích rộng rãi với hệ thống cũ | Ngày càng phổ biến, đặc biệt trên thiết bị di động/IoT |

---

## 7. Thuật toán mã hóa và trao đổi khóa

### 7.1. Thuật toán đối xứng phổ biến

| Thuật toán | Ghi chú |
| --- | --- |
| **AES (Advanced Encryption Standard)** | Chuẩn mã hóa đối xứng hiện đại, được khuyến nghị sử dụng rộng rãi (AES-128, AES-256) |
| **3DES** | Thuật toán cũ, hiện được xem là **cipher yếu**, không nên còn xuất hiện trong cấu hình production |
| **RC4** | Thuật toán cũ, đã được chứng minh có lỗ hổng, cần loại bỏ hoàn toàn |
| **ChaCha20** | Thuật toán hiện đại, hiệu năng tốt trên thiết bị không có tăng tốc phần cứng AES (ví dụ mobile) |

### 7.2. Thuật toán trao đổi khóa phổ biến

| Thuật toán | Ghi chú |
| --- | --- |
| **RSA Key Exchange** | Cách trao đổi khóa truyền thống, không hỗ trợ Forward Secrecy nếu dùng độc lập |
| **ECDHE (Elliptic Curve Diffie-Hellman Ephemeral)** | Hỗ trợ **Forward Secrecy** – mỗi phiên tạo khóa tạm thời riêng, dù private key gốc bị lộ sau này cũng không giải mã được các phiên cũ đã ghi lại |
| **DHE (Diffie-Hellman Ephemeral)** | Tương tự ECDHE nhưng dựa trên toán học cổ điển thay vì đường cong elliptic, tốc độ chậm hơn |


---

## 8. Bảng thuật ngữ 

| Thuật ngữ | Định nghĩa ngắn gọn |
| --- | --- |
| **256-bit encryption** | Quá trình mã hóa dữ liệu bằng thuật toán có độ dài khóa 256 bit — khóa càng dài, độ an toàn càng cao |
| **Asymmetric cryptography** | Nhóm thuật toán mã hóa dùng một cặp khóa (public/private) cho quá trình mã hóa và giải mã |
| **CSR (Certificate Signing Request)** | Bản yêu cầu cấp chứng chỉ ở dạng máy có thể đọc được, thường chứa public key và tên định danh của bên yêu cầu |
| **CA (Certificate Authority)** | Tổ chức được ủy quyền cấp, tạm ngưng, gia hạn hoặc thu hồi chứng chỉ |
| **Cipher suite** | Tập hợp các thuật toán (trao đổi khóa, xác thực, mã hóa, xác thực thông điệp) được dùng trong một phiên TLS |
| **Common Name (CN)** | Thuộc tính trong tên định danh của chứng chỉ; với chứng chỉ SSL, đây là tên miền (DNS host name) cần bảo mật |
| **DV – Domain Validation** | Mức xác thực chứng chỉ cơ bản nhất, chỉ xác minh quyền sở hữu tên miền |
| **OV – Organization Validation** | Mức xác thực xác nhận cả quyền sở hữu tên miền lẫn sự tồn tại hợp pháp của tổ chức |
| **EV – Extended Validation** | Mức xác thực toàn diện nhất, yêu cầu quy trình xác minh nghiêm ngặt về tổ chức |
| **ECC** | Phương pháp tạo khóa mã hóa dựa trên các điểm trên đường cong elliptic, khó bị bẻ khóa bằng brute-force và hiệu quả hơn RSA thuần |
| **Encryption** | Quá trình chuyển đổi dữ liệu ở dạng đọc được (plaintext) thành dạng không thể đọc được (ciphertext) |
| **Key exchange** | Quy trình mà client và server cùng thiết lập một pre-master secret an toàn cho phiên làm việc |
| **PKI (Public Key Infrastructure)** | Tổng thể kiến trúc, tổ chức, kỹ thuật và quy trình hỗ trợ vận hành hệ thống mã hóa khóa công khai dựa trên chứng chỉ |
| **SAN (Subject Alternative Name) certificate** | Loại chứng chỉ cho phép bảo mật nhiều tên miền khác nhau chỉ với một chứng chỉ duy nhất |
| **Wildcard certificate** | Loại chứng chỉ dùng để bảo mật nhiều subdomain của cùng một domain gốc |
| **Symmetric encryption** | Phương pháp mã hóa dùng chung một khóa cho cả mã hóa và giải mã |

---

## 9. Tài liệu tham khảo

- DigiCert – *What is SSL, TLS and HTTPS?*: `https://www.digicert.com/what-is-ssl-tls-and-https`

---

*Tài liệu được biên soạn phục vụ mục đích đào tạo nội bộ và báo cáo thực tập, tổng hợp và diễn giải lại từ tài liệu chính thức của DigiCert và nội dung thực tập cá nhân.*
