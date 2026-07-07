# Triển Khai Web Trên IIS Windows Server 2022

> **Phạm vi áp dụng:** Windows Server 2022 – IIS 10.0
> **Đối tượng:** KTV/Sysadmin triển khai và vận hành website trên nền tảng Windows Server
> **Ngôn ngữ lệnh:** PowerShell (chạy với quyền Administrator)

---

## Mục Lục

- [Triển Khai Web Trên IIS Windows Server 2022](#triển-khai-web-trên-iis-windows-server-2022)
  - [1. Tổng Quan](#1-tổng-quan)
  - [2. Cài Đặt IIS](#2-cài-đặt-iis)
    - [2.1 Cài đặt Web Server (IIS) qua PowerShell](#21-cài-đặt-web-server-iis-qua-powershell)
    - [2.2 Kiểm tra dịch vụ sau cài đặt](#22-kiểm-tra-dịch-vụ-sau-cài-đặt)
  - [3. Cấu Hình Application Pool](#3-cấu-hình-application-pool)
    - [3.1 Khái niệm Application Pool](#31-khái-niệm-application-pool)
    - [3.2 Tạo và cấu hình Application Pool](#32-tạo-và-cấu-hình-application-pool)
  - [4. Bật Tính Năng Hỗ Trợ ASP / .NET 3.5 - 4.x](#4-bật-tính-năng-hỗ-trợ-asp--net-35---4x)
  - [5. Bật CGI và Cấu Hình PHP Trên IIS](#5-bật-cgi-và-cấu-hình-php-trên-iis)
    - [5.1 Bật tính năng CGI](#51-bật-tính-năng-cgi)
    - [5.2 Tải và cài đặt PHP](#52-tải-và-cài-đặt-php)
    - [5.3 Khai báo Handler Mapping cho PHP](#53-khai-báo-handler-mapping-cho-php)
  - [6. Cấu Hình SSL/HTTPS Cho Website](#6-cấu-hình-sslhttps-cho-website)
    - [6.1 Tạo self-signed certificate (môi trường test)](#61-tạo-self-signed-certificate-môi-trường-test)
    - [6.2 Import certificate đã mua (production)](#62-import-certificate-đã-mua-production)
    - [6.3 Gán HTTPS Binding cho site](#63-gán-https-binding-cho-site)
    - [6.4 Redirect HTTP sang HTTPS](#64-redirect-http-sang-https)
  - [7. Demo Triển Khai Website](#7-demo-triển-khai-website)
    - [7.1 Demo 1: Website Basic HTML](#71-demo-1-website-basic-html)
    - [7.2 Demo 2: Website Classic ASP](#72-demo-2-website-classic-asp)
    - [7.3 Demo 3: Website .NET 3.5/4.x (ASP.NET)](#73-demo-3-website-net-3540x-aspnet)
    - [7.4 Demo 4: Website PHP](#74-demo-4-website-php)
  - [8. Troubleshooting Thường Gặp](#8-troubleshooting-thường-gặp)
  - [9. Checklist Bàn Giao](#9-checklist-bàn-giao)
  - [References](#references)

---

## 1. Tổng Quan

IIS (Internet Information Services) là web server tích hợp sẵn trên Windows Server, hỗ trợ đa dạng công nghệ backend: HTML tĩnh, Classic ASP, ASP.NET (3.5/4.x/Core), PHP thông qua FastCGI. Tài liệu này hướng dẫn triển khai đầy đủ từ cài đặt, cấu hình Application Pool, bật các tính năng hỗ trợ ngôn ngữ, cấu hình SSL, cho đến demo thực tế với 4 loại website phổ biến.

| Thành phần | Vai trò |
|---|---|
| IIS (World Wide Web Publishing Service - W3SVC) | Dịch vụ web server chính |
| Application Pool | Tiến trình cách ly (w3wp.exe) chạy website, quản lý vòng đời và tài nguyên |
| Handler Mapping | Ánh xạ loại file (.php, .aspx...) tới engine xử lý tương ứng |
| CGI/FastCGI | Giao thức để IIS gọi các interpreter ngoài (PHP, Python...) |

---

## 2. Cài Đặt IIS

### 2.1 Cài đặt Web Server (IIS) qua PowerShell

```powershell
Install-WindowsFeature -Name Web-Server -IncludeManagementTools
```

Cài kèm các module thường dùng cho vận hành thực tế:

```powershell
Install-WindowsFeature -Name Web-Common-Http, Web-Default-Doc, Web-Dir-Browsing, `
Web-Http-Errors, Web-Static-Content, Web-Http-Logging, Web-Stat-Compression, `
Web-Filtering, Web-Basic-Auth, Web-Windows-Auth, Web-Mgmt-Console
```

### 2.2 Kiểm tra dịch vụ sau cài đặt

```powershell
Get-Service W3SVC, WAS
Get-WindowsFeature -Name Web-Server
```

Trạng thái `Running` ở cả hai service xác nhận IIS đã hoạt động. Truy cập `http://localhost` để thấy trang chào mặc định của IIS.

---

## 3. Cấu Hình Application Pool

### 3.1 Khái niệm Application Pool

Mỗi Application Pool là một tiến trình `w3wp.exe` độc lập, giúp cô lập website này khỏi lỗi của website khác trên cùng server. Cấu hình quan trọng cần nắm:

| Thuộc tính | Ý nghĩa |
|---|---|
| .NET CLR Version | Phiên bản .NET Framework website sử dụng (`v4.0`, `v2.0`, hoặc `No Managed Code` cho PHP/HTML tĩnh) |
| Managed Pipeline Mode | `Integrated` (mặc định, khuyến nghị) hoặc `Classic` |
| Identity | Tài khoản chạy pool (`ApplicationPoolIdentity` mặc định) |
| Idle Time-out | Thời gian pool tự tắt khi không có request |

### 3.2 Tạo và cấu hình Application Pool

```powershell
Import-Module WebAdministration

New-WebAppPool -Name "AppPool-DemoSite"

Set-ItemProperty IIS:\AppPools\AppPool-DemoSite -Name managedRuntimeVersion -Value "v4.0"
Set-ItemProperty IIS:\AppPools\AppPool-DemoSite -Name managedPipelineMode -Value "Integrated"
Set-ItemProperty IIS:\AppPools\AppPool-DemoSite -Name processModel.idleTimeout -Value "00:20:00"
```

Đối với site HTML tĩnh hoặc PHP (không cần .NET), đặt `managedRuntimeVersion` về rỗng:

```powershell
Set-ItemProperty IIS:\AppPools\AppPool-DemoSite -Name managedRuntimeVersion -Value ""
```

---

## 4. Bật Tính Năng Hỗ Trợ ASP / .NET 3.5 - 4.x

```powershell
Install-WindowsFeature -Name Web-ASP, Web-Net-Ext45, Web-Asp-Net45, NET-Framework-45-ASPNET
```

Nếu cần hỗ trợ ứng dụng legacy .NET 3.5:

```powershell
Install-WindowsFeature -Name NET-Framework-Core
```

> Lưu ý: .NET Framework 3.5 trên Windows Server 2022 yêu cầu nguồn cài đặt từ file ISO (`sxs`), do Windows Update mặc định không có sẵn gói offline. Sử dụng tham số `-Source`:
> ```powershell
> Install-WindowsFeature -Name NET-Framework-Core -Source D:\sources\sxs
> ```

Kiểm tra lại:

```powershell
Get-WindowsFeature -Name Web-Asp-Net45, NET-Framework-Core
```

---

## 5. Bật CGI và Cấu Hình PHP Trên IIS

### 5.1 Bật tính năng CGI

```powershell
Install-WindowsFeature -Name Web-CGI
```

### 5.2 Tải và cài đặt PHP

1. Tải bản PHP **Non Thread Safe (NTS)** phù hợp kiến trúc x64 tại trang chủ: [windows.php.net/download](https://windows.php.net/download/)
2. Giải nén vào `C:\php`
3. Copy file cấu hình mẫu và đổi tên:

```powershell
Copy-Item C:\php\php.ini-production C:\php\php.ini
```

4. Mở `C:\php\php.ini`, chỉnh các dòng sau (bỏ dấu `;` đầu dòng nếu bị comment):

```ini
extension_dir = "ext"
extension=curl
extension=fileinfo
extension=mbstring
extension=mysqli
extension=openssl
cgi.fix_pathinfo=1
```

### 5.3 Khai báo Handler Mapping cho PHP

```powershell
Import-Module WebAdministration

New-WebHandler -Name "PHP_via_FastCGI" `
  -Path "*.php" `
  -Verb "*" `
  -Modules "FastCgiModule" `
  -ScriptProcessor "C:\php\php-cgi.exe" `
  -ResourceType File
```

Đăng ký PHP làm FastCGI application:

```powershell
& "$env:windir\system32\inetsrv\appcmd.exe" set config /section:system.webServer/fastCgi `
  /+"[fullPath='C:\php\php-cgi.exe']"
```

Kiểm tra nhanh bằng file `phpinfo.php` (xem [Demo 4](#74-demo-4-website-php)).

---

## 6. Cấu Hình SSL/HTTPS Cho Website

### 6.1 Tạo self-signed certificate (môi trường test)

```powershell
New-SelfSignedCertificate -DnsName "demo.local" -CertStoreLocation "cert:\LocalMachine\My"
```

### 6.2 Import certificate đã mua (production)

```powershell
Import-PfxCertificate -FilePath "C:\certs\demo_com.pfx" `
  -CertStoreLocation Cert:\LocalMachine\My `
  -Password (ConvertTo-SecureString -String "MatKhauPFX" -AsPlainText -Force)
```

### 6.3 Gán HTTPS Binding cho site

```powershell
$cert = Get-ChildItem Cert:\LocalMachine\My | Where-Object { $_.Subject -match "demo.local" }

New-WebBinding -Name "DemoSite" -Protocol "https" -Port 443 -IPAddress "*"

(Get-WebBinding -Name "DemoSite" -Protocol "https").AddSslCertificate($cert.Thumbprint, "My")
```

### 6.4 Redirect HTTP sang HTTPS

Cài module URL Rewrite (nếu chưa có), sau đó thêm rule vào `web.config` của site:

```xml
<system.webServer>
  <rewrite>
    <rules>
      <rule name="HTTP to HTTPS redirect" stopProcessing="true">
        <match url="(.*)" />
        <conditions>
          <add input="{HTTPS}" pattern="off" ignoreCase="true" />
        </conditions>
        <action type="Redirect" url="https://{HTTP_HOST}/{R:1}" redirectType="Permanent" />
      </rule>
    </rules>
  </rewrite>
</system.webServer>
```

---

## 7. Demo Triển Khai Website

### 7.1 Demo 1: Website Basic HTML

**Bước 1 – Tạo thư mục và nội dung:**

```powershell
New-Item -Path "C:\inetpub\wwwroot\demo-html" -ItemType Directory -Force
Set-Content -Path "C:\inetpub\wwwroot\demo-html\index.html" -Value "<html><body><h1>Demo HTML tren IIS</h1></body></html>"
```

**Bước 2 – Tạo site trong IIS:**

```powershell
New-WebAppPool -Name "Pool-HTML"
Set-ItemProperty IIS:\AppPools\Pool-HTML -Name managedRuntimeVersion -Value ""

New-Website -Name "DemoHTML" `
  -PhysicalPath "C:\inetpub\wwwroot\demo-html" `
  -ApplicationPool "Pool-HTML" `
  -Port 8081
```

**Bước 3 – Kiểm tra kết quả:**

```powershell
Invoke-WebRequest http://localhost:8081
```

Kết quả mong đợi: trả về HTTP 200 và nội dung `<h1>Demo HTML tren IIS</h1>`.

---

### 7.2 Demo 2: Website Classic ASP

**Bước 1 – Bật tính năng ASP:**

```powershell
Install-WindowsFeature -Name Web-ASP
```

**Bước 2 – Tạo thư mục và file `default.asp`:**

```powershell
New-Item -Path "C:\inetpub\wwwroot\demo-asp" -ItemType Directory -Force

Set-Content -Path "C:\inetpub\wwwroot\demo-asp\default.asp" -Value @"
<%
Response.Write("Demo Classic ASP - " & Now())
%>
"@
```

**Bước 3 – Tạo site:**

```powershell
New-WebAppPool -Name "Pool-ASP"
Set-ItemProperty IIS:\AppPools\Pool-ASP -Name managedRuntimeVersion -Value ""

New-Website -Name "DemoASP" `
  -PhysicalPath "C:\inetpub\wwwroot\demo-asp" `
  -ApplicationPool "Pool-ASP" `
  -Port 8082
```

**Bước 4 – Kiểm tra kết quả:**

```powershell
Invoke-WebRequest http://localhost:8082/default.asp
```

> Nếu gặp lỗi 500, kiểm tra mục **Feature Delegation** hoặc bật hiển thị lỗi chi tiết tại `Error Pages > Edit Feature Settings > Detailed errors`.

---

### 7.3 Demo 3: Website .NET 3.5/4.x (ASP.NET)

**Bước 1 – Tạo thư mục project:**

```powershell
New-Item -Path "C:\inetpub\wwwroot\demo-aspnet" -ItemType Directory -Force
```

**Bước 2 – Tạo `default.aspx`:**

```powershell
Set-Content -Path "C:\inetpub\wwwroot\demo-aspnet\default.aspx" -Value @"
<%@ Page Language="C#" AutoEventWireup="true" CodeFile="default.aspx.cs" Inherits="_Default" %>
<html>
<body>
    <form id="form1" runat="server">
        <h1>Demo ASP.NET tren IIS</h1>
    </form>
</body>
</html>
"@
```

**Bước 3 – Tạo `default.aspx.cs`:**

```powershell
Set-Content -Path "C:\inetpub\wwwroot\demo-aspnet\default.aspx.cs" -Value @"
using System;

public partial class _Default : System.Web.UI.Page
{
    protected void Page_Load(object sender, EventArgs e)
    {
    }
}
"@
```

**Bước 4 – Tạo `web.config`:**

```powershell
Set-Content -Path "C:\inetpub\wwwroot\demo-aspnet\web.config" -Value @"
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <system.web>
    <compilation debug="true" targetFramework="4.8" />
  </system.web>
</configuration>
"@
```

**Bước 5 – Tạo site với Application Pool dùng .NET v4.0:**

```powershell
New-WebAppPool -Name "Pool-AspNet"
Set-ItemProperty IIS:\AppPools\Pool-AspNet -Name managedRuntimeVersion -Value "v4.0"

New-Website -Name "DemoAspNet" `
  -PhysicalPath "C:\inetpub\wwwroot\demo-aspnet" `
  -ApplicationPool "Pool-AspNet" `
  -Port 8083
```

**Bước 6 – Kiểm tra kết quả:**

```powershell
Invoke-WebRequest http://localhost:8083/default.aspx
```

---

### 7.4 Demo 4: Website PHP

**Bước 1 – Tạo thư mục và file `phpinfo.php`:**

```powershell
New-Item -Path "C:\inetpub\wwwroot\demo-php" -ItemType Directory -Force

Set-Content -Path "C:\inetpub\wwwroot\demo-php\phpinfo.php" -Value "<?php phpinfo(); ?>"
```

**Bước 2 – Tạo site (dùng chung Handler Mapping PHP đã cấu hình ở [mục 5](#5-bật-cgi-và-cấu-hình-php-trên-iis)):**

```powershell
New-WebAppPool -Name "Pool-PHP"
Set-ItemProperty IIS:\AppPools\Pool-PHP -Name managedRuntimeVersion -Value ""

New-Website -Name "DemoPHP" `
  -PhysicalPath "C:\inetpub\wwwroot\demo-php" `
  -ApplicationPool "Pool-PHP" `
  -Port 8084
```

**Bước 3 – Kiểm tra kết quả:**

```powershell
Invoke-WebRequest http://localhost:8084/phpinfo.php
```

Kết quả mong đợi: trang `phpinfo()` hiển thị đầy đủ thông tin phiên bản PHP, module đã load.

---


## References

1. [Create a New Website in Windows IIS 10](https://www.ssl.com/how-to/create-new-website-iis-10/)
2. [How to Host PHP on Windows With IIS](https://stackify.com/how-to-host-php-on-windows_with-iis/)
3. [Microsoft Learn – IIS Documentation](https://learn.microsoft.com/en-us/iis/)
4. [PHP for Windows – Official Downloads](https://windows.php.net/download/)
