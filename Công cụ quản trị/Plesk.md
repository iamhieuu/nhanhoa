# Plesk – Tài liệu tổng hợp

## Mục lục

- [Plesk – Tổng quan](#plesk-tổng-quan)
  - [Giới thiệu](#giới-thiệu)
  - [Ưu điểm / Nhược điểm](#ưu-điểm-nhược-điểm)
  - [Mô hình người dùng (3 cấp)](#mô-hình-người-dùng-3-cấp)
  - [Chế độ giao diện](#chế-độ-giao-diện)
  - [Bố cục giao diện chung](#bố-cục-giao-diện-chung)
  - [Các khái niệm cốt lõi](#các-khái-niệm-cốt-lõi)
- [Plesk Admin (Service Provider View)](#plesk-admin-service-provider-view)
  - [Home](#home)
  - [Hosting Services](#hosting-services)
  - [Links to Additional Services](#links-to-additional-services)
  - [Server Management](#server-management)
  - [My Profile](#my-profile)
  - [Ví dụ: Tạo tài khoản Reseller giới hạn quyền](#ví-dụ-tạo-tài-khoản-reseller-giới-hạn-quyền)
- [Plesk Reseller (Service Provider View)](#plesk-reseller-service-provider-view)
  - [Home](#home)
  - [Hosting Services](#hosting-services)
  - [Links to Additional Services](#links-to-additional-services)
  - [My Profile](#my-profile)
- [Plesk Customer (End User)](#plesk-customer-end-user)
  - [Websites and Domains](#websites-and-domains)
  - [Mail](#mail)
  - [Applications](#applications)
  - [Files](#files)
  - [Databases](#databases)
  - [Statistics](#statistics)
  - [Users](#users)
  - [Account](#account)
  - [WordPress (WP Toolkit)](#wordpress-wp-toolkit)
  - [SEO (SEO Toolkit)](#seo-seo-toolkit)
  - [Laravel](#laravel)

---

## Plesk – Tổng quan

### Giới thiệu
- Plesk là bảng điều khiển quản trị web hosting (hosting control panel) phổ biến thứ 2 thế giới, hỗ trợ cả Linux và Windows, triển khai trên hơn 370.000 máy chủ, phục vụ hơn 12 triệu website.
- Điểm khác biệt: tích hợp toàn diện — quản trị hosting, thiết kế website, storefront SaaS, và hệ thống hóa đơn/thanh toán trong cùng một nền tảng.
- Hai phiên bản: **Plesk Onyx** (cơ bản: quản lý file, bảo mật, hiệu suất, tự động cập nhật) và **Plesk Obsidian** (mới nhất từ 25/09/2019, tập trung bảo mật – hiệu suất – tiện dụng, bổ sung Composer, Docker, MongoDB, SSL It!, WP Toolkit...).

### Ưu điểm / Nhược điểm
- **Ưu điểm**: dễ dùng trên Windows & Linux, ổn định, đầy đủ tính năng, giao diện thân thiện, tích hợp thiết kế web + billing + storefront, hỗ trợ nhiều FTP/hosting cùng lúc.
- **Nhược điểm**: lịch sử có lỗ hổng bảo mật; backup/restore tốn nhiều dung lượng đĩa; cài đặt phức tạp hơn cPanel (không có wizard một chạm).

### Mô hình người dùng (3 cấp)
| Vai trò | Vai trò chính |
|---|---|
| **Administrator** | Toàn quyền quản lý server, tạo/quản lý reseller, customer, dịch vụ hosting |
| **Reseller** | Mua tài nguyên từ Admin, tạo và quản lý nhiều Customer |
| **Customer (End User)** | Chủ website, quản lý hosting/email/database/FTP trong phạm vi được cấp |

### Chế độ giao diện
- **Service Provider View**: đầy đủ chức năng quản lý reseller/customer/gói dịch vụ (dùng cho tài liệu này).
- **Power User View**: giao diện đơn giản cho cá nhân/doanh nghiệp nhỏ, chỉ quản lý website riêng.

### Bố cục giao diện chung
- **Header**: tìm kiếm nhanh, thông tin tài khoản, thông báo server, Plesk360, Advisor, Help, chuyển Light/Dark mode.
- **Navigation Panel** (trái): Home, Hosting Services, Server Management, Tools & Settings, Extensions...
- **Main Display Area**: nội dung tương ứng mục đang chọn.
- **View Switcher**: chuyển Service Provider View ⇄ Power User View.

### Các khái niệm cốt lõi
- **Customer**: người dùng cuối, sở hữu 1+ Subscription, quản lý website/email/DB/FTP.
- **Reseller**: trung gian Admin–Customer, được cấp tài nguyên để phân phối lại, tạo/quản lý nhiều Customer.
- **Domain**: địa chỉ website gán vào 1 Subscription (1 Subscription có thể có nhiều domain/subdomain).
- **Subscription**: không gian lưu trữ + cấu hình dịch vụ (domain, email, DB, PHP, SSL...) cho 1 khách hàng, tạo dựa trên 1 Service Plan.
- **Service Plan**: mẫu cấu hình tài nguyên (giới hạn dung lượng, email, DB...) do Admin tạo, dùng để cấp Subscription cho Customer/Reseller.

**Quan hệ**: `Admin → tạo Service Plan → cấp cho Reseller/Customer → tạo Subscription → gán Domain`

**Ví dụ**: Admin tạo Service Plan `Basic Hosting` (5GB, 10 email, 1 DB) → tạo Customer `Nguyen A` → cấp Subscription theo gói này → gán domain `example.io.vn` → Customer đăng nhập Plesk quản lý website/email.


---

## Plesk Admin (Service Provider View)

Admin View là chế độ mạnh nhất của Plesk, quản lý toàn bộ hệ thống: tài nguyên máy chủ, người dùng, tên miền, email, bảo mật. Các khu vực chính: **Home**, **Hosting Services** (Customers, Resellers, Domains, Subscriptions, Service Plans), **Links to Additional Services**, **Server Management** (Tools & Settings, Statistics, Extensions, WordPress/Laravel, Monitoring), **My Profile**.

### Home
Dashboard tổng quan gồm: thông tin cập nhật hệ thống (Last updated, Allow Automatic Updates, Add/Remove Components); thông tin hệ thống (Hostname, IP, OS, Uptime); lịch backup (Schedule Daily, Create Backup); danh sách cập nhật phần mềm gần đây; tình trạng SSL của các website; danh sách tên miền (Add new); biểu đồ CPU/Memory 24h; nút Customize để tuỳ biến dashboard.

### Hosting Services

#### Customers
Quản lý tài khoản khách hàng — mỗi Customer có thể sở hữu nhiều Subscription.
- **Chức năng chính**: Add Customer, Convert to Reseller, Move to (đổi provider), Change Status, Remove.
- **Danh sách**: Customer Name, Subscription, Setup Date, Provider, Login as Customer (đăng nhập hộ để hỗ trợ).
- **Chi tiết 1 customer** (click vào tên): quản lý Domains/Subscriptions của khách (chi tiết cấu hình từng subscription xem tại tài liệu *Plesk Customer*), thông tin liên hệ (Edit Contact Info, Convert to Reseller, Status/Suspend, Change login info, Owner's description), IIS Application Pool riêng (Windows), Remove customer.
- **Ví dụ quy trình**: Hosting Services → Customers → Add a Customer → nhập contact info + thông tin bổ sung (company, phone, address...) + tài khoản đăng nhập Plesk → tạo Subscription đi kèm (domain name, service plan, IP, username/password) → Save.

#### Reseller
Tương tự Customers nhưng quản lý các tài khoản Reseller: Add Reseller, Change Status, Remove; danh sách gồm Reseller Name, Setup Date, số Customer quản lý. Click vào reseller để xem chi tiết tài nguyên, gói dịch vụ (Reseller Plan) đang dùng, danh sách Customer trực thuộc.

#### Domains
Danh sách toàn bộ tên miền trên server, chủ sở hữu (Customer/Reseller), Subscription, trạng thái, dung lượng/traffic sử dụng. Cho phép thao tác nhanh: Add Domain, Set Status, Remove.

#### Subscriptions
- **Chức năng**: Change Plan (đổi gói dịch vụ, thêm/gỡ add-on), Change Subscriber (chuyển chủ sở hữu, subscription chuyển sang trạng thái *Custom* — không còn ràng buộc gói cũ), Set Status, Remove.
- **Chi tiết 1 Subscription** — click vào tên mở đầy đủ các tab quản trị (**Websites and Domains, Mail, Applications, Files, Databases, Statistics, Users, Account, WordPress, SEO**). Nội dung các tab này **giống hệt** giao diện Customer khi tự quản lý subscription của mình — xem chi tiết đầy đủ tại tài liệu *Plesk Customer*. Admin/Reseller chỉ khác ở quyền truy cập: có thể vào trực tiếp từ danh sách Subscriptions mà không cần đăng nhập bằng tài khoản khách (hoặc dùng nút "Manage in Customer Panel"/"Login as Customer").

#### Service Plans
Mẫu cấu hình tài nguyên dùng để tạo Subscription. Gồm 2 loại:

**Hosting Plans** (cấp cho Customer) — cấu hình theo các tab:
- **Resources**: giới hạn dung lượng đĩa, băng thông, số domain/subdomain, mailbox, database, FTP account...
- **Permissions**: bật/tắt các quyền thao tác (quản lý DNS, SSL, backup, Git, WP Toolkit, database từ xa, IIS pool...).
- **Hosting Parameters**: loại hosting, PHP, script hỗ trợ.
- **PHP Settings, Mail, DNS, Performance, Logs & Statistics, Applications, Additional Services**: cấu hình mặc định tương ứng cho subscription tạo từ gói này.

**Reseller Plans** (cấp cho Reseller) — tương tự Hosting Plan nhưng có thêm:
- **Resources**: số lượng Customer tối đa, tổng dung lượng/băng thông/database được phép cấp phát lại.
- **Permissions**: các quyền quản trị Reseller được cấp (tạo customer, oversell tài nguyên, quản lý FTP/DB/backup/API/WP Toolkit/Laravel Toolkit/Sitejet...).
- **IP Addresses**: cấp IP dùng chung (shared) hoặc số lượng IPv4/IPv6 riêng để Reseller phân phối cho khách.
- **Application**: chọn cung cấp toàn bộ ứng dụng trong Application Catalog, hoặc chỉ một số ứng dụng chỉ định.

**Additional Services**: thêm nút liên kết dịch vụ ngoài (Add Service: tên, mô tả, URL, ảnh nền, có thể chèn biến động như `dom_id`, `cl_id`, `email`...) hiển thị trong Customer Panel; Make Available/Unavailable để bật tắt cho Reseller sử dụng.

### Links to Additional Services
- **SEO Toolkit**: công cụ phân tích & tối ưu SEO cấp server (chi tiết xem tài liệu Customer – mục SEO).
- **Process List**: xem danh sách tiến trình đang chạy trên máy chủ.

### Server Management

#### Tools & Settings
Trung tâm cấu hình hệ thống, gồm các nhóm:

**Security**: Security Policy (mật khẩu, FTPS, Enhanced Security Mode); Firewall (General bật/tắt + Panic Mode, ICMP Protocol, Firewall Rules – thêm rule theo port/protocol/IP); Web Application Firewall/ModSecurity (chế độ Off/Detection/On, rule set Atomic/OWASP/Custom); SSL/TLS Certificates cấp server (Let's Encrypt, upload, quản lý pool chứng chỉ); Restrict Creation of Subzones & các cấu hình hệ thống khác (hostname, tính dung lượng đĩa/traffic, cấm đổi tên miền chính, log IP...); Additional Administrator Accounts (tạo thêm admin, Restricted Mode, Force Power User view); Active Sessions (Plesk/FTP/Terminal), Session Idle Time, IP Access Restriction, Prohibited Domain Names.

**Assistance and Troubleshooting**: công cụ hỗ trợ kiểm tra/sửa lỗi hệ thống, truy cập tài liệu trợ giúp.

**Tools & Resources**: quản lý IP addresses, backup toàn server, gửi mail hàng loạt, tác vụ định kỳ (scheduler) cấp server, di chuyển dữ liệu (migration).

**General Settings**: Ngày giờ hệ thống + múi giờ + đồng bộ NTP; DNS Settings (Zone Records/Settings Template, Zone Transfers, Service-wide recursive query); FTP Settings (dải cổng passive); Website Preview (Quick preview/Preview theo domain nội bộ hoặc ngoài/Disable); PHP Settings (bật/tắt PHP handler theo domain); Customize Plesk URL (tuỳ biến domain truy cập Plesk).

**Server Management**: Info and Statistics (Overview CPU/RAM/disk, Domain list, Traffic usage theo domain/customer/reseller, Reports – Summary/Full, lên lịch gửi báo cáo qua email); Server Components (DNS server, Firewall, Mail Server/Webmail/Spam Filter/Antivirus, Node.js, MariaDB/MySQL Webadmin); Services Management (Start/Stop/Restart dịch vụ, đổi Startup type); Restart/Shut Down Server; Remote API (REST) – tài liệu API, ví dụ gọi curl.

**Mail**: Mail Server Settings (chọn phần mềm mail server), Antivirus, Spam Filter, Outgoing Mail Control, Webmail, Smarthost, External SMTP Server.

**Applications & Databases**: quản lý ứng dụng cài đặt cấp server, cấu hình database server, ODBC, IIS/ASP.NET (Windows).

**Plesk**: License Information (Plesk License Key, Retrieve/Install/Roll back key, license bổ sung), About Plesk, Cookies (Anonymous tracking / Personalization / Necessary functional – bật/tắt từng loại).

**Plesk Appearance**: Branding (custom title, logo, favicon, hình nền login); Languages (quản lý gói ngôn ngữ – Enable/Disable/Make Default); Interface Management (chọn Power User/Service Provider view, Restricted Mode, Hide bounce controls); Custom Buttons (thêm nút chức năng tuỳ chỉnh với URL + tham số động).

#### Statistics
Trùng nội dung với **Info and Statistics** trong Server Management (Overview, Domain, Traffic usage, Reports).

#### Extensions
Cài đặt, cập nhật, cấu hình các tiện ích mở rộng của Plesk (marketplace extension).

#### WordPress / Laravel (cấp server)
Công cụ quản trị (WP Toolkit, Laravel Toolkit) áp dụng cho toàn bộ website trên server — thao tác Installation/Plugins/Themes/Sets với WordPress, và Install Application/Scan/Deployment/Scheduled Task/Queue với Laravel giống hệt phần trình bày trong tài liệu *Plesk Customer* (mục WordPress, Laravel), chỉ khác là admin thấy toàn bộ site trên server thay vì chỉ site của mình.

#### Monitoring
Giám sát tài nguyên qua Plesk 360 (online) hoặc Grafana tích hợp sẵn (local). Các tab: Overview (CPU/RAM theo core), Services (CPU/RAM theo IIS/Mail/MySQL/Plesk), Disk (dung lượng, throughput), Memory (RAM + swap), CPU (theo core, số tiến trình), Network (băng thông theo interface).

### My Profile
- **Profile and Preferences**: ngôn ngữ giao diện admin, cho phép đa phiên đăng nhập, MFA, thông tin liên hệ (login, tên, email, công ty, địa chỉ, SĐT), tuỳ chọn nhận newsletter từ WebPros.
- **Change Password**: đổi mật khẩu (Old/New/Confirm, Generate, Show).

### Ví dụ: Tạo tài khoản Reseller giới hạn quyền
1. **Tạo Reseller Plan**: Service Plans → Reseller Plans → Add a Plan → đặt tên `Reseller For Test` → giới hạn Customers: 100, Disk: 20GB, Traffic: 20GB, Databases: 5 → giữ mặc định các phân quyền khác → Save.
2. **Tạo Reseller**: Resellers → Add Reseller → nhập username `reseller_annt`, email, thông tin đăng nhập → gán Reseller Plan vừa tạo → Save.
3. **Kiểm tra**: Đăng xuất Admin, đăng nhập bằng `reseller_annt` để xem giao diện Reseller.


---

## Plesk Reseller (Service Provider View)

Reseller là đối tác trung gian giữa Admin và Customer: có thể tạo/quản lý khách hàng, gán gói dịch vụ/tên miền/email/tài nguyên, theo dõi sử dụng, cá nhân hoá giao diện (logo, thông tin liên hệ) — nhưng **không** có quyền cấu hình máy chủ, cài extension, hay thay đổi chính sách bảo mật toàn hệ thống. Các khu vực chính: **Home**, **Hosting Services** (Customers, Domains, Subscriptions, Service Plans, Statistics, Tools & Utilities, WordPress, Laravel), **My Profile**.

### Home
- Thông tin gói dịch vụ của Reseller: Setup date, Service plan, Disk space/Traffic đã dùng, liên kết "View detailed resource usage" và "My resources and permissions overview".
- Quản lý nhanh: số Customer hiện có (+ số vượt giới hạn, Add new); số Subscription hiện có; số Service Plan đã tạo (Add new).
- **Create own subscription**: Reseller có thể tự tạo 1 subscription riêng để lưu trữ website của chính mình.

### Hosting Services

#### Customers
Giống hệt phần Customers ở Admin nhưng **không có** tuỳ chọn "Convert to Reseller" hay "Move to" (Reseller không thể tạo sub-reseller). Chức năng: Add Customer, Change Status, Remove. Danh sách: Customer Name, Subscription, Setup Date, Login as Customer.
- Chi tiết 1 khách hàng: quản lý Domains/Subscriptions (Add Domain, Subdomain, Domain Alias, Set Status, Remove; danh sách domain kèm Disk Usage/Traffic/Rank tracker); quản lý Subscription (Add Subscription, Change Plan, Change Subscriber, Set Status, Remove). Nội dung chi tiết từng subscription (Websites and Domains, Mail, Applications, Files, Databases, Statistics, Users, Account, WordPress, SEO) **giống hệt** tài liệu *Plesk Customer* — xem tại đó.

#### Domains
Danh sách tên miền được cấp phát cho khách hàng của Reseller (không thấy domain thuộc Reseller/Customer khác).

#### Subscriptions
Thao tác Change Plan / Change Subscriber / Set Status / Remove tương tự Admin. Riêng phần **Databases** trong subscription có thêm nút "Move to Subscription" (di chuyển DB sang subscription khác) và Backup Manager cấp subscription (chọn nội dung backup: config/mail/file/DB, loại full/incremental, lịch backup, remote storage FTPS).
- Ví dụ thêm domain mới: Add Domain → chọn kiểu khởi tạo (Blank/HTML-PHP, Website Builder kéo-thả, WordPress, Node.js, domain chỉ dùng email, upload từ máy, Git, Laravel, import từ hosting khác) → nhập tên miền, chọn webspace (subscription mới hoặc dùng chung), có thể gán ngay cho customer mới/có sẵn → Add Domain.

#### Service Plans
Chỉ tạo được **Hosting Plans** (không có Reseller Plans — Reseller không thể tạo sub-reseller). Chức năng: Add a Plan, Clone Plans, Add an add-on, Remove. Các tab cấu hình 1 plan: **Resources** (Overuse policy: không cho phép / cho phép vượt disk-traffic / cho phép vượt tất cả — kèm tài nguyên chi tiết: disk, traffic, domains/subdomains/aliases, mailbox, FTP, database MySQL/PostgreSQL/MSSQL, ngày hết hạn, WordPress instances, Rank Tracker, web users, ODBC); **Permissions** (DNS zone, hosting settings, PHP settings/version, web scripting an toàn, spam filter, antivirus, backup, web statistics, log rotation, WP Toolkit, Laravel Toolkit, Node.js, Sitejet, domains/mail/FTP/database management...); **Hosting Parameters, PHP Settings, Mail, DNS, Performance, Logs & Statistics, Application, Additional Services** — tương tự cấu hình Hosting Plan bên Admin.

#### Statistics
Xem thống kê tài nguyên theo từng khách hàng (tương tự Info and Statistics bên Admin nhưng giới hạn trong phạm vi khách hàng của Reseller).

#### Tools & Utilities (Tools & Settings ở mức Reseller)
Phạm vi hẹp hơn Admin, gồm:
- **Resources**: IIS Application Pool mẫu (General/Performance Settings), Application Vault (danh sách ứng dụng đã cài trên các website), Virtual Host Template (quản lý thư mục mẫu: Upload, Set default).
- **Plesk Management**: Backup Manager (backup toàn bộ tài khoản Reseller — nội dung, lịch, remote storage); Active Plesk Sessions & Active FTP Sessions (theo dõi phiên đăng nhập của khách hàng: Login, IP, Logon/Idle time — có Refresh/Close); Plesk Branding (custom title/logo); Interface Preferences (ngôn ngữ, đa phiên, chọn Power User/Service Provider view); Custom Buttons (thêm nút tuỳ chỉnh với URL + tham số động, giống Admin).

#### WordPress / Laravel
Quản lý ứng dụng WordPress/Laravel do khách hàng của Reseller triển khai — thao tác Installation/Plugins/Themes/Sets (WordPress) và Install Application/Deployment/Scheduled Task/Queue (Laravel) **giống hệt** phần trình bày trong tài liệu *Plesk Customer*.

### Links to Additional Services
- **SEO**: liên kết mở SEO Toolkit (chi tiết xem tài liệu Customer – mục SEO).

### My Profile
- **Profile**: thông tin liên hệ (tên, email, công ty...).
- **Change Password**: đổi mật khẩu.
- **Interface Preferences**: ngôn ngữ giao diện, đa phiên đăng nhập, chọn Power User/Service Provider view.


---

## Plesk Customer (End User)

Customer là người dùng cuối, vận hành website/email/database/ứng dụng trong phạm vi Subscription được cấp. Không có quyền cấu hình hệ thống, không tạo thêm người dùng ngoài phạm vi subscription, không đổi cấu hình gói dịch vụ. Đăng nhập qua `https://<domain>:8443`.

> **Lưu ý**: Nội dung các mục dưới đây (Websites & Domains, Mail, Applications, Files, Databases, Statistics, Users, Account, WordPress, SEO, Laravel) là **tài liệu tham chiếu chung** — khi Admin/Reseller click vào một Subscription cụ thể để quản trị, giao diện và các thao tác hiển thị **giống hệt** những gì trình bày ở đây.

Các khu vực chính: **Websites & Domains**, **Mail**, **Applications**, **Files**, **Databases**, **Statistics**, **Users**, **Account**, **WordPress**, **SEO**, **Laravel**.

### Websites and Domains
- **Thêm mới**: Add Domain (tên miền, webspace/subscription, hosting type: Web Hosting/Forwarding/No Hosting, bật DNS/Mail, document root, preferred domain); Add Subdomain.
- **Danh sách domain**: Domain Name, Status, Disk usage, Traffic; các nút tắt: WordPress Management, Sitejet Builder, File Manager, Mail Account, Databases, Hosting Settings; thao tác Move Domain, Change Domain Name, Remove Website.
- **Chi tiết 1 website** — gồm khung preview + thống kê (disk/traffic, More Statistics: Web Statistics/Data Transfer/Diskspace-traffic) và các tab:
  - **Dashboard**: Connection Info (FTP/DB credentials, phpMyAdmin, Additional FTP accounts); **Files** (quản lý file/thư mục, upload/nén); **Databases** (tạo/quản lý DB, phpMyAdmin, Copy, Export/Import Dump, Check & repair, User Management, Backup Manager); **Scheduled Tasks** (cron); **PHP Composer**; **Git** (Pull code/Push code, deployment mode Automatic/Manual/Disabled); **Failed Request Tracing**; **Applications** (cài CMS nhanh); **WordPress Management, SEO, Website Importing, Sitejet Builder**.
  - **Bảo mật website**: SSL/TLS Certificates (Let's Encrypt miễn phí hoặc upload/manage certificate pool, CSR); Password-Protected Directories; Advisor; Hotlink Protection (chặn site khác nhúng ảnh/video, whitelist "friendly websites").
  - **Hosting & DNS**: Hosting settings (domain, hosting type, preferred domain, document root, SSL, web statistics Webalizer/AWStats, web scripting, system user FTP credentials, Remote Desktop — thường bị khoá theo chính sách gói); IIS Settings (Windows – directory browsing, default documents, MIME types, deny access, expires, headers, authentication, DoS protection); DNS (Records: Add/Disable/Switch to Secondary/Reset/Remove; Settings: primary NS, TTL, SOA Refresh/Retry/Expire/Minimum; Zone Transfers: whitelist IP); Dedicated IIS Application Pool (Windows – General: managed pipeline mode, 32-bit, user profile; Performance: worker processes, idle timeout, CPU limit + action, recycling theo thời gian/requests/memory); Bandwidth Limiting (KB/s, số kết nối đồng thời).
  - **ODBC Data Sources**: kết nối SQL Server/MS Access/MS Excel (driver, server, UID/PWD, tham số nâng cao).
  - **FTP**: thêm tài khoản FTP phụ (home directory, quota, quyền đọc/ghi).
  - **Backup & Restore**: tạo backup (nội dung: config/mail/file/DB; đầy đủ hoặc incremental; ghi chú; loại trừ log/file đặc biệt); Upload backup (giới hạn 2GB/file, kiểm tra chữ ký, mật khẩu bảo vệ); Schedule; remote storage FTPS.
  - **Get Started**: Website Builder (140+ mẫu, no-code, AI content, SEO, ecommerce), Upload Files, Laravel, More Apps, Deploy using Git, WordPress, Node.js, Import an App/Site.

### Mail
- **Mail Accounts** (Create mail address): General (email address, external email, password, mailbox size, description); Forwarding (bật chuyển tiếp, giữ/không giữ bản sao, nhiều địa chỉ); Email alias; Auto-reply (chủ đề, nội dung Plain/HTML, forward, giới hạn số lần, tắt theo ngày); Antivirus. Danh sách account: tên, contact, dung lượng dùng; nút Mail Client Setup, Webmail.
- **Mail Settings** (theo domain): Enabled/Disabled/Not configured; xử lý mail gửi tới user không tồn tại (Forward/Redirect ra ngoài/Reject); Webmail (MailEnable/Horde/None); SSL cho webmail/mail; mailing lists, DKIM, mail autodiscover.
- **Mailing Lists**: tạo danh sách gửi hàng loạt (địa chỉ, bật/tắt, admin email, subscribers, thông báo khi tạo).
- **Limiting Outgoing Messages**: mặc định 50 mail/giờ, có thể tuỳ chỉnh hoặc Unlimited.
- **Mail Importing**: nhập mail qua IMAP (source login/password, đích là mailbox có sẵn hoặc mới, cấu hình nâng cao host/port/encryption/timeout).
- Ở mức Subscription (nhiều domain): các tab trên áp dụng đồng loạt cho tất cả domain qua nút Activate/Deactivate Services, kèm bảng theo dõi Outgoing Mail Control chi tiết theo domain và theo subscription (current limit, số lần vượt giới hạn, thống kê gửi theo giờ).

### Applications
Quản lý ứng dụng cài nhanh (WordPress, Joomla...): tab **Manage My Applications** (Scan, xem đường dẫn cài đặt, Remove), **Featured Applications** (đề xuất phổ biến, chọn version), **All Available Applications**.

### Files
Trình quản lý file/thư mục trên hosting: thêm/sửa/xoá, upload, nén.

### Databases
Tạo/quản lý CSDL MySQL/PostgreSQL (chi tiết thao tác giống mục Databases trong Websites & Domains): danh sách DB (host, user, bảng, kích thước), phpMyAdmin, Connection Info, Copy, Export/Import Dump, Check & repair, Remove, User Management (Add Database User: server, username, password, database, access control local/remote/theo IP).

### Statistics
- **Disk Usage** & **Traffic**: biểu đồ sử dụng / giới hạn được cấp, phân tích theo dịch vụ.
- **Data Transfer Statistics**: báo cáo lưu lượng theo domain (dịch vụ, In/Out, % tổng).
- **Web Statistics**: báo cáo truy cập (lượt xem, trang phổ biến).

### Users
- **User Accounts**: Create New Account (contact name, email nội bộ/external, User role, Access to subscriptions; Plesk username/password, ngôn ngữ, kích hoạt qua email); danh sách (contact name, email, role); Remove.
- **User Roles**: Create User Role (tên role, kích hoạt qua email, quyền truy cập dịch vụ Plesk: quản lý user/role, tạo site, log rotation, FTP ẩn danh, scheduled tasks, spam filter, antivirus, database, backup, statistics, applications, Presence Builder, files, FTP phụ, DNS, mail accounts/lists; quyền truy cập app webmail); danh sách role + số user.

### Account
- **My Profile**: General (contact name, email, external email; Plesk username/password/ngôn ngữ; MFA); Contact Details (company, phone, address đầy đủ).
- Nút **Backup** (mở giao diện backup) và **Additional Service** (nút tuỳ chỉnh do Reseller/Admin cấu hình).
- Thông tin gói: tên gói, ngày thiết lập, trạng thái; các tab xem chi tiết **Resources / Hosting options / Permissions / Additional Services** được cấp theo Service Plan.

### WordPress (WP Toolkit)
4 khu vực: **Installation, Plugins, Themes** (Sets ở cấp server/reseller).

#### Installation
- **Install**: General (installation path, website title, plugin/theme set, ngôn ngữ, version); WordPress Administrator (username/password/email); Database (tên DB, prefix, user, password); Automatic Update Settings (WP core: tắt/chỉ security patch/toàn bộ; plugin & theme: theo cài đặt riêng từng plugin, mặc định bật cho plugin mới, hoặc cập nhật tất cả).
- **Scan**: quét site WP chưa quản lý. **Updates**: Update/Change Settings/Check Updates, danh sách site có bản cập nhật (Core/Plugins/Themes/SmartUpdate).
- **Security**: kiểm tra & áp dụng biện pháp bảo mật.
- **Giao diện quản trị 1 site WP**: Preview, Login (vào wp-admin), Setup, Manage Domain; tab **Dashboard** (File Manager, Copy Data, Clone site sang subdomain/domain khác, Backup/restore, Logs, WP-CLI; Status: version WP/Plugins/Themes/Security/PHP/SSL; Tools: SEO indexing, Debugging WP_DEBUG*, Password Protection, wp-cron, Hotlink protection); **Plugins/Themes** (Install/Remove/Auto-update, danh sách + trạng thái Active/AutoUpdate); **Database** (tên DB, prefix, user, server, mở phpMyAdmin).
- **Khác**: Check WordPress Integrity (Verify Checksums, Reinstall core); Maintenance Mode (template, nội dung hiển thị, countdown timer, social links).

#### Plugins / Themes (WP Toolkit cấp cao hơn — quản lý hàng loạt qua nhiều site)
Install (tìm kiếm, chọn website cài, active sau cài) hoặc Upload (file zip); Activate/Deactivate/Uninstall/Update; danh sách plugin/theme kèm site đã cài.

### SEO (SEO Toolkit)
Phân tích thứ hạng, tiếp cận mạng xã hội, gợi ý tối ưu nội dung.
- **SEO Toolkit Wizard**: chọn domain → điều kiện Site Audit/Log File Analyzer/Rank Tracker → thêm keyword theo dõi (search engine, wildcard subdomain) → thêm đối thủ cạnh tranh (giới hạn bản miễn phí) → hoàn tất.
- **Giao diện quản lý theo domain**: Site Audit (điểm tổng thể, nhóm Content/SEO/Technology, số vấn đề Critical/Warning/Notice); Tasks (tiến độ xử lý, Set Done/Skip); Rank Tracking (thêm/theo dõi từ khoá, filter, xu hướng theo thời gian); Log File Analyzer (phân tích crawler qua access log).

### Laravel
- **Install Application**: chọn domain + source code (vd Laravel skeleton). **Scan**: dò site Laravel đã triển khai.
- **Giao diện quản trị 1 app**: Manage Domain, Logs; tab **Dashboard** (Application Info: URL/Repository/Last Commit; Settings: .env, Scheduled Tasks, Queue, Maintenance mode); **Artisan/Composer/Node.js** (chạy lệnh); **Development** (Deployment Mode Automatic/Manual, Script, Steps, nút Deploy); **Scheduled Task** (cron); **Queue** (Stop Worker When Empty, Timeout, Max Jobs, Max Time, Show failed jobs).
