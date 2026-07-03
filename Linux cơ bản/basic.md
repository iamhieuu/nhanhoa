# 01 – Tổng quan Linux và Ubuntu 22.04 LTS

---

## Mục lục

- [1. Linux là gì](#1-linux-là-gì)
- [2. Kiến trúc hệ thống Linux](#2-kiến-trúc-hệ-thống-linux)
- [3. Các bản phân phối phổ biến](#3-các-bản-phân-phối-phổ-biến)
- [4. Ubuntu 22.04 LTS](#4-ubuntu-2204-lts)

---

## 1. Linux là gì

Linux là một họ hệ điều hành mã nguồn mở dựa trên **Linux Kernel** — được Linus Torvalds phát hành lần đầu ngày 17/9/1991. Viết bằng ngôn ngữ C, Linux có thể chạy trên máy tính cá nhân, server, thiết bị nhúng, điện thoại di động và hệ thống điện toán đám mây.

### Mốc lịch sử quan trọng

| Năm | Sự kiện |
|-----|---------|
| 1991 | Linus Torvalds công bố Linux trên Usenet, phát hành phiên bản đầu tiên |
| 1992 | Linux phát hành dưới giấy phép GNU GPL — cho phép tự do sử dụng, chỉnh sửa, phân phối |
| 1993–1994 | Các distro đầu tiên xuất hiện: Debian, Slackware, Red Hat |
| 1996 | Linux Kernel 2.0 ra mắt với hỗ trợ đa xử lý đối xứng (SMP) |
| 1998–1999 | IBM, Oracle, Dell bắt đầu hỗ trợ Linux |
| 2004 | Ubuntu ra mắt, nhanh chóng trở thành distro phổ biến nhất |
| 2007 | Google phát triển Android dựa trên Linux Kernel |
| 2010s–nay | Linux trở thành nền tảng cho cloud, container (Docker, Kubernetes), IoT |

---

## 2. Kiến trúc hệ thống Linux

Kiến trúc Linux gồm 5 lớp chính, xếp từ thấp đến cao:

```
┌─────────────────────────────────┐
│       System Utilities (App)    │  ← Người dùng tương tác
├─────────────────────────────────┤
│             Shell               │  ← Giao diện dòng lệnh / GUI
├─────────────────────────────────┤
│         System Library          │  ← Cầu nối ứng dụng ↔ Kernel
├─────────────────────────────────┤
│             Kernel              │  ← Lõi hệ điều hành
├─────────────────────────────────┤
│         Hardware Layer          │  ← CPU, RAM, Disk, Network
└─────────────────────────────────┘
```


### 2.1 Kernel

Kernel là lõi của hệ điều hành, chịu trách nhiệm ảo hóa tài nguyên phần cứng và phân phối cho các tiến trình.

**Các loại Kernel:**

| Loại | Đặc điểm | Ví dụ |
|------|----------|-------|
| **Monolithic** | Toàn bộ dịch vụ OS chạy trong kernel, hiệu suất cao | Linux, Unix |
| **Micro** | Chỉ giữ tối thiểu trong kernel, phần còn lại ở user space | Mach, Minix |
| **Hybrid** | Kết hợp Monolithic + Micro | Windows NT, macOS |
| **Exo** | Ít trừu tượng phần cứng nhất, phân bổ tài nguyên trực tiếp | ExOS |
| **Nano** | Trừu tượng hóa phần cứng, không có dịch vụ hệ thống | EROS |

**7 chức năng cốt lõi của Kernel:**

1. **Quản lý tiến trình** – lập lịch, tạo, kết thúc, chuyển đổi ngữ cảnh giữa các tiến trình
2. **Quản lý bộ nhớ** – phân bổ RAM, bộ nhớ ảo, bảo vệ và chia sẻ bộ nhớ
3. **Quản lý thiết bị** – giao tiếp với driver, cung cấp giao diện thống nhất cho phần cứng
4. **Quản lý hệ thống tệp** – đọc/ghi file, mount/unmount filesystem
5. **Quản lý tài nguyên** – phân phối CPU, disk, băng thông mạng giữa các tiến trình
6. **Bảo mật và kiểm soát truy cập** – xác thực, phân quyền, đảm bảo toàn vẹn hệ thống
7. **Giao tiếp giữa tiến trình (IPC)** – message passing, shared memory

### 2.2 System Library

Tập hợp các hàm được định nghĩa sẵn để ứng dụng tương tác với Kernel mà không cần gọi trực tiếp vào kernel space.

| Thư viện | Vai trò |
|----------|---------|
| **GNU C (glibc)** | Thư viện C cơ bản nhất, nền tảng của mọi chương trình C trên Linux |
| **libpthread** | Đa luồng theo chuẩn POSIX |
| **libdl** | Tải và liên kết thư viện động khi runtime |
| **libm** | Các hàm toán học |
| **libcrypt** | Mã hóa và bảo mật |

### 2.3 Shell

Shell là giao diện giữa người dùng và Kernel — nhận lệnh từ người dùng, chuyển thành system call cho Kernel xử lý.

**Phân loại:**
- **Command Line Shell (CLI):** bash, zsh, sh — người dùng nhập lệnh văn bản
- **Graphical Shell (GUI):** GNOME, KDE — giao diện đồ họa cửa sổ


### 2.4 Hardware Layer

Lớp thấp nhất, bao gồm trình điều khiển thiết bị và giao tiếp trực tiếp với CPU, RAM, ổ đĩa, card mạng. Kernel trừu tượng hóa lớp này để phần mềm không cần biết chi tiết phần cứng.

### 2.5 System Utilities

Các công cụ dòng lệnh cho phép người dùng quản lý hệ thống: quản lý file, giám sát hệ thống, cấu hình mạng, quản lý user, v.v.

---

## 3. Các bản phân phối phổ biến

**Distro (Linux Distribution)** = Linux Kernel + GNU tools + thư viện + phần mềm + trình quản lý gói, đóng gói thành một hệ điều hành hoàn chỉnh.

Hiện có khoảng 600 distro, trong đó ~500 đang được phát triển tích cực.

### 3.1 Nhóm Debian

| Distro | Đặc điểm |
|--------|----------|
| **Debian** | Ổn định bậc nhất, chu kỳ phát hành dài, phổ biến trên server |
| **Ubuntu** | Thân thiện, phổ biến nhất với người dùng và sysadmin |
| **Kali Linux** | Chuyên dụng cho bảo mật và kiểm thử xâm nhập |

- Định dạng gói: `.deb`
- Trình quản lý gói: `apt`
- Kiến trúc: x86-64, ARM, v.v.

### 3.2 Nhóm Red Hat

| Distro | Đặc điểm |
|--------|----------|
| **Red Hat Enterprise Linux (RHEL)** | Thương mại, hỗ trợ doanh nghiệp |
| **CentOS / AlmaLinux / Rocky** | Miễn phí, tương thích RHEL, phổ biến trên server |
| **Fedora** | Cộng đồng, tích hợp tính năng mới nhất |

- Định dạng gói: `.rpm`
- Trình quản lý gói: `yum` / `dnf`

### 3.3 Nhóm Slackware

- Ra đời sớm nhất (1993)
- Định dạng gói: `.tgz` / `.txz`
- Ổn định, tùy biến cao, thích hợp cho server và người dùng nâng cao

---

## 4. Ubuntu 22.04 LTS

### 4.1 Tổng quan

Ubuntu 22.04 LTS (Jammy Jellyfish) là bản phát hành **Long-Term Support** của Ubuntu, cung cấp **5 năm** cập nhật bảo mật và bảo trì từ Canonical.

| Thông tin | Giá trị |
|-----------|---------|
| Tên phiên bản | Ubuntu 22.04 LTS (Jammy Jellyfish) |
| Ngày phát hành | 21/4/2022 |
| Hỗ trợ đến | 4/2027 (standard) / 4/2032 (ESM) |
| Desktop | GNOME (tùy chỉnh bởi Canonical) |
| Kernel mặc định | Linux 5.15 LTS |
| Trình quản lý gói | APT + Snap |

### 4.2 Lý do chọn Ubuntu 22.04 LTS

- **Ổn định:** Chu kỳ LTS không thay đổi thường xuyên, phù hợp cho môi trường production và lab học tập.
- **Tài liệu phong phú:** Cộng đồng lớn, tài liệu đầy đủ, dễ tìm giải pháp khi gặp lỗi.
- **Tương thích phần cứng rộng:** Hỗ trợ tốt trên đa số laptop, máy để bàn, máy ảo VMware/VirtualBox.
- **Nền tảng cho server:** Kỹ năng học trên Ubuntu 22.04 Desktop áp dụng trực tiếp vào Ubuntu Server — môi trường phổ biến trong hosting, cloud, DevOps.

### 4.3 Thiết lập sau khi cài xong

Việc đầu tiên sau khi login lần đầu là cập nhật hệ thống:

```bash
sudo apt update        # Cập nhật danh sách gói từ repository
sudo apt upgrade -y    # Cài đặt tất cả bản cập nhật
sudo apt autoremove    # Xóa gói không còn cần thiết
```

**Cài đặt công cụ cơ bản:**

```bash
sudo apt install -y curl wget git vim net-tools tree htop
```

**Kiểm tra thông tin hệ thống:**

```bash
uname -a              # Phiên bản kernel
lsb_release -a        # Thông tin distro
hostnamectl           # Hostname và hệ điều hành
df -h                 # Dung lượng ổ đĩa
free -h               # Dung lượng RAM
```

### 4.4 APT – Trình quản lý gói

`apt` (Advanced Package Tool) là công cụ quản lý gói trên Ubuntu/Debian. Mọi phần mềm cài đặt qua `apt` đều được tải từ **repository** (kho lưu trữ) chính thức, đảm bảo tính toàn vẹn và bảo mật.

| Lệnh | Mô tả |
|------|-------|
| `sudo apt update` | Cập nhật danh sách gói từ repository |
| `sudo apt upgrade` | Nâng cấp các gói đã cài đặt |
| `sudo apt install <gói>` | Cài đặt gói |
| `sudo apt remove <gói>` | Gỡ gói (giữ cấu hình) |
| `sudo apt purge <gói>` | Gỡ gói và xóa luôn cấu hình |
| `apt search <từ khóa>` | Tìm kiếm gói trong repository |
| `apt show <gói>` | Xem thông tin chi tiết của gói |
| `dpkg -l` | Liệt kê tất cả gói đã cài |

> **Lưu ý:** Luôn chạy `sudo apt update` trước `sudo apt install` để đảm bảo danh sách gói mới nhất.

---

## References

- [Introduction to Linux OS – GeeksForGeeks](https://www.geeksforgeeks.org/introduction-to-linux-operating-system/)
- [Kernel in Operating System – GeeksForGeeks](https://www.geeksforgeeks.org/kernel-in-operating-system/)
- [Getting Started With Ubuntu 22.04 LTS – ITU Online](https://www.ituonline.com/blogs/getting-started-with-ubuntu-22-04-lts-features-installation-and-tips/)
- [Ubuntu Release Cycle – Canonical](https://ubuntu.com/about/release-cycle)
