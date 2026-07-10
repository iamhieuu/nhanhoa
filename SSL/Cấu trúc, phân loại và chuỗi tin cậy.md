# SSL/TLS CHUYÊN ĐỀ 2: CHỨNG CHỈ SSL – CẤU TRÚC, PHÂN LOẠI & CHUỖI TIN CẬY

> **Loại tài liệu:** Tài liệu định nghĩa chuyên đề – dùng cho đào tạo nội bộ (seminar intern) và báo cáo thực tập
> **Phạm vi:** Khái niệm về chứng chỉ SSL, CSR, định dạng chứng chỉ, chuỗi chứng chỉ và mô hình CA — không bao gồm hướng dẫn cài đặt hoặc thao tác dòng lệnh
> **Nguồn tham khảo:** Tổng hợp từ tài liệu thực tập ngày 42 và ngày 44 (SSL Termination)

---

## MỤC LỤC

1. [Chứng chỉ SSL là gì?](#1-chứng-chỉ-ssl-là-gì)
   - 1.1. [Khái niệm và mục đích](#11-khái-niệm-và-mục-đích)
   - 1.2. [Cấu trúc cốt lõi của một chứng chỉ SSL](#12-cấu-trúc-cốt-lõi-của-một-chứng-chỉ-ssl)
   - 1.3. [Common Name (CN) và Subject Alternative Name (SAN)](#13-common-name-cn-và-subject-alternative-name-san)
2. [Phân loại chứng chỉ SSL theo mức xác thực](#2-phân-loại-chứng-chỉ-ssl-theo-mức-xác-thực)
   - 2.1. [DV – Domain Validation](#21-dv--domain-validation)
   - 2.2. [OV – Organization Validation](#22-ov--organization-validation)
   - 2.3. [EV – Extended Validation](#23-ev--extended-validation)
   - 2.4. [Bảng so sánh nhanh DV / OV / EV](#24-bảng-so-sánh-nhanh-dv--ov--ev)
3. [Phân loại chứng chỉ SSL theo phạm vi domain](#3-phân-loại-chứng-chỉ-ssl-theo-phạm-vi-domain)
   - 3.1. [Single Domain Certificate](#31-single-domain-certificate)
   - 3.2. [Wildcard SSL](#32-wildcard-ssl)
   - 3.3. [SAN SSL / Multi-Domain Certificate](#33-san-ssl--multi-domain-certificate)
4. [Self-Signed Certificate và CA-Signed Certificate](#4-self-signed-certificate-và-ca-signed-certificate)
5. [CSR – Certificate Signing Request](#5-csr--certificate-signing-request)
   - 5.1. [CSR là gì?](#51-csr-là-gì)
   - 5.2. [Vì sao phải tạo CSR trước khi xin SSL?](#52-vì-sao-phải-tạo-csr-trước-khi-xin-ssl)
   - 5.3. [CSR chứa những thông tin gì?](#53-csr-chứa-những-thông-tin-gì)
   - 5.4. [CA xử lý CSR như thế nào?](#54-ca-xử-lý-csr-như-thế-nào)
6. [Định dạng chứng chỉ SSL](#6-định-dạng-chứng-chỉ-ssl)
   - 6.1. [Các định dạng phổ biến](#61-các-định-dạng-phổ-biến)
   - 6.2. [Bảng so sánh định dạng](#62-bảng-so-sánh-định-dạng)
   - 6.3. [Hệ thống nào dùng định dạng nào?](#63-hệ-thống-nào-dùng-định-dạng-nào)
7. [Certificate Chain – Chuỗi chứng chỉ](#7-certificate-chain--chuỗi-chứng-chỉ)
   - 7.1. [Định nghĩa](#71-định-nghĩa)
   - 7.2. [Vì sao cần Certificate Chain?](#72-vì-sao-cần-certificate-chain)
   - 7.3. [Các lỗi Certificate Chain thường gặp](#73-các-lỗi-certificate-chain-thường-gặp)
8. [Intermediate CA và Root CA](#8-intermediate-ca-và-root-ca)
   - 8.1. [Định nghĩa](#81-định-nghĩa)
   - 8.2. [Vì sao Root CA không ký trực tiếp cho khách hàng](#82-vì-sao-root-ca-không-ký-trực-tiếp-cho-khách-hàng)
   - 8.3. [Ví dụ thực tế mô hình CA](#83-ví-dụ-thực-tế-mô-hình-ca)
   - 8.4. [Cơ chế Trust Chain của trình duyệt/hệ điều hành](#84-cơ-chế-trust-chain-của-trình-duyệthệ-điều-hành)

---

## 1. Chứng chỉ SSL là gì?

### 1.1. Khái niệm và mục đích

Có thể hình dung **chứng chỉ SSL** như CMND/CCCD của một website. Nó chứng minh 3 điều:

1. Website này thuộc về ai.
2. Ai đã cấp chứng chỉ.
3. Chứng chỉ còn hiệu lực đến khi nào.

### 1.2. Cấu trúc cốt lõi của một chứng chỉ SSL

Chứng chỉ SSL (chuẩn X.509) gồm 5 phần cốt lõi:

| Thành phần | Ý nghĩa | Ví dụ |
| --- | --- | --- |
| **Subject** | Chủ sở hữu chứng chỉ: tên miền (CN), tổ chức (O), quốc gia (C) | `CN=congty.vn, O=Cong Ty ABC, C=VN` |
| **Issuer** | CA đã ký và bảo lãnh cho chứng chỉ | Let's Encrypt, Sectigo, DigiCert... |
| **Validity** | Thời hạn hiệu lực (Not Before / Not After) | Let's Encrypt: 90 ngày; chứng chỉ thương mại: 1–2 năm |
| **Public Key** | Khóa công khai dùng để mã hóa, đi kèm thuật toán ký (Signature Algorithm) | RSA 2048-bit hoặc ECDSA 256-bit |
| **SAN (Subject Alternative Name)** | Danh sách domain thực sự được chứng chỉ bảo vệ | `DNS: congty.vn`, `DNS: www.congty.vn` |

### 1.3. Common Name (CN) và Subject Alternative Name (SAN)

- **Common Name (CN):** Là tên miền chính mà chứng chỉ đại diện. Trước đây, trình duyệt chỉ dựa vào CN để xác định chứng chỉ có hợp lệ với domain đang truy cập hay không.
- **SAN (Subject Alternative Name):** Kể từ Chrome 58, trình duyệt **bỏ qua hoàn toàn CN** khi xác thực — chỉ còn dựa vào SAN. CN giờ chỉ mang tính tham khảo, không còn giá trị xác thực kỹ thuật thực sự.

>  Khách hàng báo "SSL bị lỗi" nhưng nguyên nhân là domain họ đang truy cập (ví dụ `shop.congty.vn`) **không nằm trong danh sách SAN** của chứng chỉ, dù domain chính (`congty.vn`) đã có chứng chỉ hợp lệ. Cần luôn kiểm tra SAN, không chỉ CN, khi chẩn đoán lỗi loại này.

---

## 2. Phân loại chứng chỉ SSL theo mức xác thực

### 2.1. DV – Domain Validation

Mức xác thực **thấp nhất và phổ biến nhất**. CA chỉ xác minh một điều duy nhất: **bạn có quyền sở hữu domain đó hay không**. CA hoàn toàn không xác minh công ty đứng sau domain có hợp pháp hay có thật hay không.

| Đặc điểm | Giá trị |
| --- | --- |
| Mức xác thực | Thấp |
| Thời gian cấp | Vài phút |
| Chi phí | Miễn phí (Let's Encrypt) đến khoảng 10 USD/năm |
| Hiển thị | HTTPS và biểu tượng ổ khóa |
| Phù hợp | Blog cá nhân, landing page, website SMB, website giới thiệu công ty |
| Không phù hợp | Ngân hàng, thanh toán trực tuyến, hệ thống yêu cầu độ tin cậy cao |

### 2.2. OV – Organization Validation

Mức xác thực **trung bình**. Ngoài xác minh quyền sở hữu domain, CA còn xác minh thêm: công ty có đăng ký kinh doanh hợp lệ không, địa chỉ công ty có thật không, số điện thoại có xác minh được không.

| Đặc điểm | Giá trị |
| --- | --- |
| Mức xác thực | Trung bình |
| Thời gian cấp | 1 – 3 ngày |
| Chi phí | 50 – 200 USD/năm |
| Hiển thị | HTTPS, kèm thông tin doanh nghiệp trong chi tiết chứng chỉ |
| Phù hợp | Website doanh nghiệp, cổng thông tin nội bộ, hệ thống B2B, API cho đối tác |

### 2.3. EV – Extended Validation

Mức xác thực **cao nhất**. CA thực hiện xác minh rất kỹ: giấy phép kinh doanh, lịch sử hoạt động doanh nghiệp, xác minh qua điện thoại từ bên thứ ba, kiểm tra thông tin WHOIS của domain.

| Đặc điểm | Giá trị |
| --- | --- |
| Mức xác thực | Cao nhất |
| Thời gian cấp | 1 – 2 tuần |
| Chi phí | 100 – 500 USD/năm |
| Hiển thị | HTTPS, kèm thông tin doanh nghiệp chi tiết trong chứng chỉ |
| Phù hợp | Ngân hàng, tài chính, cổng thanh toán, website chính phủ, hoặc bất kỳ nơi nào yêu cầu mức độ tin tưởng cao nhất |

### 2.4. Bảng so sánh nhanh DV / OV / EV

| Loại SSL | Xác minh gì | Thời gian cấp | Phù hợp |
| --- | --- | --- | --- |
| **DV** | Quyền sở hữu domain | Vài phút | Website cá nhân, blog, website thông thường |
| **OV** | Domain + tổ chức tồn tại hợp pháp | 1 – 3 ngày | Website doanh nghiệp nhỏ và vừa |
| **EV** | Domain + tổ chức + xác minh pháp lý chuyên sâu | Vài ngày đến vài tuần | Thương mại điện tử, ngân hàng, tài chính |

---

## 3. Phân loại chứng chỉ SSL theo phạm vi domain

### 3.1. Single Domain Certificate

Chỉ bảo vệ **đúng một domain** (và tùy chọn thêm phiên bản `www.` nếu được khai báo riêng trong SAN). Đây là loại chứng chỉ đơn giản và phổ biến nhất khi chỉ có 1 website cần bảo vệ.

### 3.2. Wildcard SSL

Bảo vệ toàn bộ **subdomain cấp 1** của một domain gốc, ví dụ chứng chỉ cho `*.congty.vn` sẽ bảo vệ được:

- `mail.congty.vn`
- `shop.congty.vn`
- `admin.congty.vn`
- `api.congty.vn`

>  **Giới hạn quan trọng:** Wildcard **không** bảo vệ subdomain cấp 2, ví dụ `sub.mail.congty.vn` sẽ **không** được cover bởi chứng chỉ `*.congty.vn`.

| Đặc điểm | Giá trị |
| --- | --- |
| Giá tham khảo | 80 – 300 USD/năm |
| Dùng khi | Có nhiều subdomain, không muốn mua/quản lý từng chứng chỉ riêng lẻ |

### 3.3. SAN SSL / Multi-Domain Certificate

Bảo vệ **nhiều domain khác nhau** (không nhất thiết cùng domain gốc) chỉ trong một chứng chỉ duy nhất, ví dụ:

```
congty.vn
congty.com
shop.congty.vn
api.partners.com
```

| Đặc điểm | Giá trị |
| --- | --- |
| Dùng khi | Một công ty sở hữu nhiều domain khác nhau cần bảo vệ cùng lúc |
| Lợi ích | Tiết kiệm công quản lý — chỉ cần theo dõi 1 chứng chỉ thay vì nhiều chứng chỉ riêng biệt |

---

## 4. Self-Signed Certificate và CA-Signed Certificate

| Tiêu chí | Self-Signed Certificate | CA-Signed Certificate |
| --- | --- | --- |
| Đơn vị ký chứng chỉ | Tự tạo và tự ký | Được ký bởi một CA uy tín |
| Chi phí | Miễn phí | Miễn phí (Let's Encrypt) hoặc có phí |
| Mức độ tin cậy | Thấp | Cao |
| Trình duyệt có tin tưởng không | Không | Có |
| Cảnh báo trình duyệt | Hiển thị cảnh báo bảo mật | Không cảnh báo |
| Xác thực danh tính | Không | Có |
| Môi trường sử dụng phù hợp | Lab, môi trường test, hệ thống nội bộ | Production, website công khai |
| Độ phức tạp triển khai | Đơn giản | Cần trải qua quy trình xác thực với CA |

---

## 5. CSR – Certificate Signing Request

### 5.1. CSR là gì?

**CSR (Certificate Signing Request)** là một tệp yêu cầu cấp chứng chỉ SSL, tuân theo chuẩn **PKCS#10**. CSR đóng vai trò là "đơn xin cấp chứng chỉ" mà server gửi cho CA.

CSR chứa:

- Public Key của server.
- Thông tin định danh domain hoặc doanh nghiệp.
- Chữ ký được tạo từ Private Key tương ứng (chứng minh server thực sự sở hữu cặp khóa này).

>  Private Key **luôn được giữ lại trên server**, không bao giờ gửi đi. CA chỉ nhận CSR — vì vậy CSR có thể được gửi qua Internet một cách an toàn mà không lo lộ khóa bí mật.

### 5.2. Vì sao phải tạo CSR trước khi xin SSL?

Khi cấp chứng chỉ SSL, CA cần xác minh 2 thông tin:

1. **Ai** đang yêu cầu cấp chứng chỉ.
2. **Public Key nào** sẽ được gắn vào chứng chỉ.

CSR đóng gói cả hai thông tin này (thông tin domain/tổ chức + public key + chữ ký số) trong cùng một tệp duy nhất, giúp CA xử lý yêu cầu một cách nhất quán và có thể xác minh được.

### 5.3. CSR chứa những thông tin gì?

| Trường | Ý nghĩa | Ví dụ |
| --- | --- | --- |
| **CN (Common Name)** | Domain chính cần cấp SSL | `congty.vn` |
| **O (Organization)** | Tên tổ chức | `Cong Ty ABC` |
| **OU (Organizational Unit)** | Phòng ban (tùy chọn) | `IT Department` |
| **C (Country)** | Mã quốc gia | `VN` |
| **ST (State/Province)** | Tỉnh/Thành phố | `Ha Noi` |
| **L (Locality)** | Quận/Huyện | `Dong Da` |
| **SAN** | Danh sách domain bổ sung | `www.congty.vn`, `mail.congty.vn` |

### 5.4. CA xử lý CSR như thế nào?

Sau khi nhận CSR, CA thực hiện tuần tự các bước sau:

1. Đọc thông tin định danh trong CSR.
2. Xác minh quyền sở hữu domain (theo một trong các phương pháp ACME Challenge — xem chi tiết ở Chuyên đề 3).
3. Lấy Public Key từ CSR.
4. Tạo Certificate gắn với Public Key đó.
5. Ký Certificate bằng Private Key của chính CA (hoặc của Intermediate CA thuộc CA đó).

Kết quả cuối cùng là một chứng chỉ SSL hoàn chỉnh, được cài đặt trên server và tiếp tục sử dụng **chính cặp khóa (Private Key) ban đầu** đã dùng để tạo CSR.

---

## 6. Định dạng chứng chỉ SSL

### 6.1. Các định dạng phổ biến

Chứng chỉ SSL đều tuân theo chuẩn **X.509**, nhưng có thể được lưu trữ dưới nhiều định dạng file khác nhau tùy theo hệ điều hành và ứng dụng sử dụng:

| Định dạng | Mô tả |
| --- | --- |
| **PEM** | Dạng văn bản (Base64), phổ biến nhất trên Linux, dùng cho Apache/Nginx; có thể chứa Certificate, Private Key, hoặc cả Chain |
| **CRT** | Thường là Certificate dạng PEM, không chứa Private Key, phổ biến trên Linux/Unix |
| **CER** | Tương tự CRT nhưng thường dùng trên Windows, có thể ở dạng PEM hoặc DER |
| **DER** | Dạng nhị phân (binary), không đọc được bằng text editor, thường dùng cho Java hoặc một số ứng dụng Windows |
| **PFX / P12** | Chuẩn PKCS#12, chứa cả Certificate + Private Key + Chain, được bảo vệ bằng mật khẩu, thường dùng trên IIS, Windows và Java |

### 6.2. Bảng so sánh định dạng

| Định dạng | Dạng lưu trữ | Chứa Private Key | Thường dùng |
| --- | --- | --- | --- |
| PEM | Text | Có thể | Apache, Nginx, Linux |
| CRT | PEM | Không | Linux/Unix |
| CER | PEM hoặc DER | Không | Windows |
| DER | Binary | Có thể | Java, Windows |
| PFX | Binary | Có | IIS, Windows |
| P12 | Binary | Có | Java, macOS |

### 6.3. Hệ thống nào dùng định dạng nào?

| Hệ thống | Định dạng |
| --- | --- |
| Apache | PEM (`.crt` + `.key`) |
| Nginx | PEM (`.crt` + `.key`) |
| IIS | PFX |
| Windows (nói chung) | CER, PFX |
| Java | P12, JKS |

>  Khi hỗ trợ khách hàng chuyển đổi giữa các nền tảng hosting (ví dụ từ Windows/IIS sang Linux/Nginx), việc đầu tiên cần xác định là **định dạng chứng chỉ hiện có** và **định dạng mà hệ thống đích yêu cầu**, vì đây là nguyên nhân phổ biến gây lỗi không cài được SSL khi di chuyển hạ tầng.

---

## 7. Certificate Chain – Chuỗi chứng chỉ

### 7.1. Định nghĩa

**Certificate Chain** là một dãy các chứng chỉ liên kết với nhau bằng quan hệ ký số, bắt đầu từ chứng chỉ của website, đi qua một hoặc nhiều Intermediate CA, và kết thúc tại một Root CA mà trình duyệt/hệ điều hành đã tin tưởng sẵn từ trước:

```
Website Certificate   (cert của congty.vn, được ký bởi Intermediate CA)
        │
        ▼
Intermediate CA        (cert của CA trung gian, được ký bởi Root CA)
        │
        ▼
Root CA                (tự ký, đã có sẵn trong Trusted Root Store của OS/Browser)
```

### 7.2. Vì sao cần Certificate Chain?

Trình duyệt **không tin trực tiếp** chứng chỉ của website. Nó phải xác minh được rằng chứng chỉ đó được ký bởi một CA mà nó tin tưởng, bằng cách lần theo chuỗi:

```
Website Cert → Intermediate CA → Root CA → Trusted Root Store → Được tin cậy
```

Nếu lần được tới một Root CA đáng tin cậy, website được coi là hợp lệ. Nếu chuỗi bị đứt đoạn ở bất kỳ mắt xích nào, trình duyệt sẽ báo lỗi tin cậy.

### 7.3. Các lỗi Certificate Chain thường gặp

| Lỗi | Nguyên nhân | Đặc điểm nhận biết |
| --- | --- | --- |
| **Incomplete Chain** | Server chỉ gửi Website Certificate nhưng thiếu Intermediate CA | Trình duyệt Chrome đôi khi vẫn hoạt động , nhưng mobile app, Java, hoặc `curl` thường báo lỗi rõ ràng |
| **Unknown Issuer** | Trình duyệt không tìm được CA đã ký chứng chỉ | Thường do thiếu Intermediate CA, hoặc CA không nằm trong Trusted Root Store |
| **Certificate Not Trusted** | Trình duyệt không tin chứng chỉ | Nguyên nhân có thể là: chứng chỉ self-signed, chain không đầy đủ, hoặc CA không được tin cậy |

>  Incomplete Chain là lỗi phổ biến nhất trong công việc support hosting — cần luôn cấu hình server gửi kèm **cả Website Certificate và Intermediate CA** (thường ghép thành file `fullchain`), Root CA thì thường **không cần gửi** vì trình duyệt/OS đã có sẵn.

---

## 8. Intermediate CA và Root CA

### 8.1. Định nghĩa

- **Root CA:** Là tổ chức chứng thực gốc, tự ký chứng chỉ của chính mình (self-signed), và chứng chỉ đó được cài đặt sẵn trong Trusted Root Store của hệ điều hành và trình duyệt.
- **Intermediate CA:** Là tổ chức chứng thực trung gian, được Root CA ký cấp chứng chỉ, và chính Intermediate CA này mới là bên **trực tiếp** ký chứng chỉ cho website/khách hàng cuối.

```
Root CA            (self-signed, có sẵn trong Trusted Root Store)
   │
   │ (ký cấp)
   ▼
Intermediate CA     (không tự ký, được Root CA bảo lãnh)
   │
   │ (ký cấp)
   ▼
Website Certificate (cert của congty.vn)
```

### 8.2. Vì sao Root CA không ký trực tiếp cho khách hàng

Private key của Root CA là tài sản có giá trị tin cậy cao nhất trong toàn hệ thống PKI — nếu private key này bị lộ, toàn bộ chứng chỉ đã từng phát hành trên thế giới dựa vào Root CA đó đều mất giá trị tin cậy. Vì vậy, private key của Root CA thường được:

- Lưu trữ **offline (air-gapped)**, không kết nối Internet.
- Chỉ dùng trong những sự kiện ký cấp hiếm hoi, được kiểm soát nghiêm ngặt (gọi là **key ceremony**).

Việc dùng Intermediate CA làm lớp đệm mang lại các lợi ích:

| Lợi ích | Giải thích |
| --- | --- |
| Giảm rủi ro | Nếu Intermediate CA bị lộ key, chỉ cần thu hồi (revoke) Intermediate đó, Root CA vẫn an toàn |
| Vận hành linh hoạt | Intermediate CA có thể hoạt động online để ký hàng loạt chứng chỉ cho khách hàng mỗi ngày |
| Phân lớp trách nhiệm | Mỗi Intermediate CA có thể giới hạn phạm vi theo loại chứng chỉ, khu vực, hoặc mục đích sử dụng |

### 8.3. Ví dụ thực tế mô hình CA

| CA | Mô hình |
| --- | --- |
| **Let's Encrypt** | Root: ISRG Root X1 → Intermediate: R10/R11 → Chứng chỉ website |
| **DigiCert** | Root: DigiCert Global Root → nhiều Intermediate theo từng dòng sản phẩm |
| **Sectigo** | Root: USERTrust RSA/ECC → Intermediate: Sectigo RSA/ECC Domain Validation |

### 8.4. Cơ chế Trust Chain của trình duyệt/hệ điều hành

Chrome, Firefox, Windows, macOS đều duy trì danh sách Root CA tin cậy riêng. Khi nhận một chứng chỉ, trình duyệt thực hiện tuần tự:

1. Đọc thông tin **Issuer** trên chứng chỉ website → tìm Intermediate CA tương ứng.
2. Đọc **Issuer** của chính Intermediate CA đó → lần tiếp lên Root CA.
3. Nếu Root CA cuối cùng nằm trong Trusted Root Store → **tin cậy toàn bộ chuỗi**.
4. Nếu không tìm được, hoặc chuỗi bị đứt đoạn ở bất kỳ mắt xích nào → báo lỗi `Unknown Issuer` hoặc `Not Trusted`.

---
