# NFS Server & LVM

---
## Mục lục
 
- [Phần 1: NFS Server](#phần-1-nfs-server)
  - [1.1. Tổng quan](#11-tổng-quan)
  - [1.2. Cài đặt NFS Server](#12-cài-đặt-nfs-server)
    - [Bước 1: Cài đặt dịch vụ NFS](#bước-1-cài-đặt-dịch-vụ-nfs)
    - [Bước 2: Tạo thư mục muốn share và phân quyền](#bước-2-tạo-thư-mục-muốn-share-và-phân-quyền)
    - [Bước 3: Cấu hình file /etc/exports](#bước-3-cấu-hình-file-etcexports)
    - [Bước 4: Áp dụng cấu hình, khởi động dịch vụ và mở tường lửa](#bước-4-áp-dụng-cấu-hình-khởi-động-dịch-vụ-và-mở-tường-lửa)
  - [1.3. Kết nối NFS Server với Client](#13-kết-nối-nfs-server-với-client)
    - [1.3.1. Client là Windows](#131-client-là-windows)
    - [1.3.2. Client là Linux](#132-client-là-linux)
  - [1.4. Permanent Mount cho NFS Client trên Ubuntu 22.04](#14-permanent-mount-cho-nfs-client-trên-ubuntu-2204)
    - [Bước 1: Mở file fstab](#bước-1-mở-file-fstab)
    - [Bước 2: Thêm dòng cấu hình](#bước-2-thêm-dòng-cấu-hình)
    - [Bước 3: Kiểm tra cấu hình trước khi reboot](#bước-3-kiểm-tra-cấu-hình-trước-khi-reboot)
- [Phần 2: LVM (Logical Volume Manager)](#phần-2-lvm-logical-volume-manager)
  - [2.1. Tổng quan](#21-tổng-quan)
  - [2.2. Quá trình khởi tạo LVM](#22-quá-trình-khởi-tạo-lvm)
    - [Bước 1: Tạo Physical Volume](#bước-1-tạo-physical-volume)
    - [Bước 2: Tạo Volume Group](#bước-2-tạo-volume-group)
    - [Bước 3: Tạo Logical Volume](#bước-3-tạo-logical-volume)
    - [Bước 4: Tạo Filesystem trên LV](#bước-4-tạo-filesystem-trên-lv)
    - [Bước 5: Permanent Mount trên Ubuntu 22.04 (chỉnh sửa fstab)](#bước-5-permanent-mount-trên-ubuntu-2204-chỉnh-sửa-fstab)
  - [2.3. Mở rộng LV (Extend)](#23-mở-rộng-lv-extend)
  - [2.4. Thêm ổ cứng mới để mở rộng LV](#24-thêm-ổ-cứng-mới-để-mở-rộng-lv)
    - [Bước 1: Tạo Physical Volume](#bước-1-tạo-physical-volume-1)
    - [Bước 2: Mở rộng Volume Group](#bước-2-mở-rộng-volume-group)
    - [Bước 3: Mở rộng Logical Volume](#bước-3-mở-rộng-logical-volume)
  - [2.5. Out / thay thế 1 ổ cứng khỏi hệ thống LVM](#25-out--thay-thế-1-ổ-cứng-khỏi-hệ-thống-lvm)
    - [Bước 1: Chuyển dữ liệu (pvmove)](#bước-1-chuyển-dữ-liệu-pvmove)
    - [Bước 2: Gỡ ổ cứng ra khỏi VG](#bước-2-gỡ-ổ-cứng-ra-khỏi-vg)
    - [Bước 3: Xóa định dạng LVM, tháo ổ](#bước-3-xóa-định-dạng-lvm-tháo-ổ)
  - [2.6. Xóa Logical Volume, Volume Group, Physical Volume](#26-xóa-logical-volume-volume-group-physical-volume)
  - [2.7. Bổ sung: các lệnh xem thông tin LVM thường dùng](#27-bổ-sung-các-lệnh-xem-thông-tin-lvm-thường-dùng)
  - [2.8. Bổ sung: Đổi tên LV/VG](#28-bổ-sung-đổi-tên-lvvg)
  - [2.9. Bổ sung: LVM Snapshot (chụp nhanh dữ liệu)](#29-bổ-sung-lvm-snapshot-chụp-nhanh-dữ-liệu)
---

## Phần 1: NFS Server

### 1.1. Tổng quan

**NFS (Network File System)** là một giao thức hệ thống tệp phân tán, cho phép máy client truy cập vào các thư mục và tệp tin nằm trên một máy server khác thông qua mạng, sử dụng giống như đang thao tác trên ổ đĩa cục bộ của chính mình.

**Mục đích:** chia sẻ thư mục/dữ liệu dùng chung giữa nhiều máy trong cùng hệ thống mạng (ví dụ: chia sẻ thư mục web, thư mục backup, home directory dùng chung...).

**Mô hình hoạt động:**

```
[NFS Server]  --- chia sẻ thư mục qua mạng --->  [NFS Client]
   /data                                            /mnt/connect_nfs
(exportfs, nfs-server)                          (mount -t nfs)
```

---

### 1.2. Cài đặt NFS Server

#### Bước 1: Cài đặt dịch vụ NFS

```bash
sudo apt update
sudo apt install nfs-kernel-server -y
```

**Giải thích:**
- `nfs-kernel-server`: gói chứa daemon NFS server chạy trực tiếp trong kernel (hiệu năng cao hơn so với NFS chạy ở user-space).

<img width="466" height="88" alt="image" src="https://github.com/user-attachments/assets/ce5edb83-7333-4b28-83b8-97803f557513" />

---

#### Bước 2: Tạo thư mục muốn share và phân quyền

```bash
sudo mkdir -p /data/nfs_share
sudo chmod 777 /data/nfs_share
sudo chown nobody:nogroup /data/nfs_share
```

Ở đây làm bài lab test nên em để phân quyền tạm thời là `777` và chủ sở hữu chuyển sang mặc định.

**Giải thích:**
- `chmod 777`: cho phép mọi user (owner/group/other) đọc, ghi, thực thi — chỉ dùng trong môi trường lab, **không dùng cho production**.
- `chown nobody:nogroup`: đổi chủ sở hữu thư mục sang user/group ẩn danh — phù hợp khi client kết nối với quyền không xác định (anonymous).

<img width="428" height="62" alt="image" src="https://github.com/user-attachments/assets/148ac6fb-c4e0-4677-90bc-a779ce7f9df4" />

---

#### Bước 3: Cấu hình file /etc/exports

File này là nơi khai báo cho hệ thống biết thư mục nào, chia sẻ cho ai, với quyền hạn gì.

```bash
sudo nano /etc/exports
```

Nội dung khai báo:

```
/data/nfs_share    192.168.254.50(rw,sync,no_subtree_check,no_root_squash)
```

**Giải thích cú pháp:** `<thư_mục_chia_sẻ>   <IP_client>(<các_option>)`

| Option | Ý nghĩa |
|---|---|
| `rw` | Cho phép client đọc và **ghi** (read-write). Dùng `ro` nếu chỉ muốn cho đọc (read-only). |
| `sync` | Ghi dữ liệu xuống đĩa ngay khi nhận yêu cầu, đảm bảo an toàn dữ liệu (chậm hơn `async` nhưng ít rủi ro mất dữ liệu khi mất điện/crash). |
| `no_subtree_check` | Tắt kiểm tra subtree — tăng hiệu năng, khuyến nghị dùng khi export cả một filesystem/thư mục lớn. |
| `no_root_squash` | Cho phép user `root` trên client có quyền `root` luôn trên server đối với thư mục share (mặc định là `root_squash`, ánh xạ root client → user `nobody` trên server để tăng bảo mật). |

> IP `192.168.254.50` là client được phép truy cập. Có thể thay bằng dải mạng, ví dụ `192.168.254.0/24` để cho phép cả subnet.

<img width="472" height="251" alt="image" src="https://github.com/user-attachments/assets/2f90948d-37e8-410b-bfa6-29d0bf7e5702" />

---

#### Bước 4: Áp dụng cấu hình, khởi động dịch vụ và mở tường lửa

```bash
sudo exportfs -ra
sudo systemctl enable --now nfs-kernel-server
sudo ufw allow from 192.168.254.50 to any port nfs
```

**Giải thích:**
- `exportfs -ra`: đọc lại toàn bộ (`-a`) nội dung file `/etc/exports` và áp dụng lại (`-r` = re-export) mà không cần restart service.
- `systemctl enable --now`: vừa bật service ngay lập tức (`--now`), vừa cấu hình để service tự khởi động cùng hệ thống (`enable`).
- `ufw allow ... port nfs`: mở cổng NFS (thường là 2049) cho đúng IP client, tránh mở toàn bộ ra ngoài.

Kiểm tra thư mục đã export thành công:

```bash
sudo exportfs -v
showmount -e localhost
```

<img width="448" height="125" alt="image" src="https://github.com/user-attachments/assets/d5ced77c-d9df-4767-895e-097e908bec8d" />

---

### 1.3. Kết nối NFS Server với Client

#### 1.3.1. Client là Windows

**Bật tính năng NFS Client trên Windows:**
Vào *Turn Windows features on or off* → tick chọn **Services for NFS** → *Client for NFS*.

<img width="203" height="185" alt="image" src="https://github.com/user-attachments/assets/a5e41865-94a2-4036-92d1-077ad95748e1" />

**Mount ổ NFS vào Windows:**

```powershell
mount -o anon \\192.168.254.10\data\nfs_share Z:
```

**Giải thích:**
- `-o anon`: mount với quyền ẩn danh (khớp với `no_root_squash`/`nobody` phía server).
- `Z:`: ổ đĩa sẽ hiển thị trong File Explorer.

<img width="838" height="301" alt="image" src="https://github.com/user-attachments/assets/cd68b51a-fd04-46bb-b0a3-32ad0ffcdee1" />

---

#### 1.3.2. Client là Linux

**Cài đặt gói hỗ trợ NFS:**

```bash
sudo apt update
sudo apt install nfs-common -y
```

<img width="385" height="92" alt="image" src="https://github.com/user-attachments/assets/0c1e7801-a9f7-4dd6-a343-3a4b9da230e2" />

**Tạo điểm gắn mount:**

```bash
sudo mkdir -p /mnt/connect_nfs
```

**Gắn mount (tạm thời — mất khi reboot):**

```bash
sudo mount -t nfs 192.168.254.10:/data/nfs_share /mnt/connect_nfs
```

**Giải thích:**
- `-t nfs`: chỉ định loại filesystem cần mount là NFS.
- `192.168.254.10:/data/nfs_share`: `IP_server:đường_dẫn_export`.

<img width="487" height="179" alt="image" src="https://github.com/user-attachments/assets/307a6e5a-723f-4057-bda0-bbc0c0966c68" />

**Kết quả đạt được:**

```bash
df -h | grep nfs
```

<img width="482" height="126" alt="image" src="https://github.com/user-attachments/assets/1293450a-77ad-4abe-929d-2b96c665ae4e" />

---

### 1.4. Permanent Mount cho NFS Client trên Ubuntu 22.04

Mount thủ công (`mount -t nfs...`) sẽ **mất khi reboot**. Để tự động mount mỗi khi khởi động máy, cần khai báo trong `/etc/fstab`.

#### Bước 1: Mở file fstab

```bash
sudo nano /etc/fstab
```

#### Bước 2: Thêm dòng cấu hình

```
192.168.254.10:/data/nfs_share   /mnt/connect_nfs   nfs   defaults,_netdev,x-systemd.automount   0   0
```

**Giải thích từng cột (chuẩn 6 cột của fstab):**

| Cột | Giá trị ví dụ | Ý nghĩa |
|---|---|---|
| 1. Filesystem | `192.168.254.10:/data/nfs_share` | Nguồn share: `IP_server:đường_dẫn` |
| 2. Mount point | `/mnt/connect_nfs` | Thư mục local để gắn vào |
| 3. Type | `nfs` | Loại filesystem |
| 4. Options | `defaults,_netdev,x-systemd.automount` | Xem chi tiết bên dưới |
| 5. Dump | `0` | Không backup bằng lệnh `dump` |
| 6. Pass (fsck order) | `0` | Không kiểm tra fsck khi boot (NFS không cần fsck local) |

**Giải thích các option quan trọng:**
- `defaults`: dùng các option mặc định (`rw`, `suid`, `dev`, `exec`, `auto`, `nouser`, `async`).
- `_netdev`: **bắt buộc với mount qua mạng** — báo cho hệ thống biết đây là thiết bị mạng, phải đợi mạng sẵn sàng rồi mới mount, tránh lỗi boot treo khi chưa có mạng.
- `x-systemd.automount`: chỉ mount thực sự khi có truy cập đầu tiên vào thư mục (lazy mount) — giúp boot nhanh hơn, tránh treo nếu server NFS tạm thời không phản hồi.

#### Bước 3: Kiểm tra cấu hình trước khi reboot

```bash
sudo mount -a
df -h | grep nfs
```

**Giải thích:**
- `mount -a`: mount thử tất cả các entry trong `/etc/fstab` — nếu không báo lỗi tức là cấu hình đúng, không cần chờ reboot mới biết.

> **Lưu ý:** nếu server NFS bị tắt/mất mạng mà client cứ cố mount lúc boot (không có `_netdev`/`x-systemd.automount`), máy Ubuntu có thể bị treo lâu ở màn hình boot. Hai option này giúp tránh rủi ro đó.

---

## Phần 2: LVM (Logical Volume Manager)

### 2.1. Tổng quan

**LVM** là một phương pháp quản lý ổ đĩa trên Linux, cho phép quản lý không gian lưu trữ linh hoạt hơn nhiều so với cách chia phân vùng (partition) truyền thống — có thể mở rộng/thu nhỏ dung lượng mà không cần format lại hay mất dữ liệu.

**Ví dụ thực tế:** có 2 ổ cứng → tạo PV từ mỗi ổ → gộp vào 1 VG dung lượng 4GB → cắt ra LV theo nhu cầu → format → mount vĩnh viễn.

**Kiến trúc phân lớp:**

```
Disk (ổ cứng vật lý)
   ↓
PV (Physical Volume)     — lớp "nguyên liệu thô"
   ↓
VG (Volume Group)        — "kho chung" gộp nhiều PV lại
   ↓
LV (Logical Volume)      — "phân đoạn" cắt ra từ kho, giống 1 partition ảo
   ↓
Filesystem (ext4, xfs...)
```

| Thành phần | Định nghĩa |
|---|---|
| **PV** (Physical Volume) | Ổ đĩa hoặc phân vùng vật lý được "gắn nhãn" LVM để có thể đưa vào VG. |
| **VG** (Volume Group) | Một "hồ chứa" dung lượng, gộp từ một hoặc nhiều PV. |
| **LV** (Logical Volume) | Một phần dung lượng được cắt ra từ VG, dùng để tạo filesystem và mount như ổ đĩa bình thường. |

---

### 2.2. Quá trình khởi tạo LVM

Tạo 1 ổ cứng vật lý 3GB, 1 ổ cứng vật lý 10GB (thêm ổ đĩa ảo trong VM để làm lab).

<img width="556" height="335" alt="image" src="https://github.com/user-attachments/assets/7a5f5878-d171-4083-a83b-95cbcbdaac7c" />

#### Bước 1: Tạo Physical Volume

```bash
sudo pvcreate /dev/sdb
```

**Giải thích:**
- `pvcreate`: khởi tạo (ghi metadata LVM) lên một ổ đĩa/phân vùng để nó có thể được LVM nhận diện và sử dụng.

<img width="313" height="114" alt="image" src="https://github.com/user-attachments/assets/1f8a50e1-4e12-4e41-85cf-90870a97dad2" />

#### Bước 2: Tạo Volume Group

```bash
sudo vgcreate vg_data /dev/sdb
```

**Giải thích:**
- `vgcreate <tên_VG> <PV1> [PV2...]`: tạo Volume Group từ một hoặc nhiều PV. Có thể gộp nhiều PV vào cùng 1 VG ngay từ đầu, ví dụ: `vgcreate vg_data /dev/sdb /dev/sdc`.

<img width="314" height="95" alt="image" src="https://github.com/user-attachments/assets/9f50b9ef-4db7-40d0-9f6b-6f66db6ebc96" />

#### Bước 3: Tạo Logical Volume

```bash
sudo lvcreate -n data_lv -L 2G vg_data
```

**Giải thích các flag:**

| Flag | Ý nghĩa |
|---|---|
| `-n` | Tên (name) của LV cần tạo. |
| `-L` | Chỉ định dung lượng cụ thể theo đơn vị (VD: `-L 2G` = 2GB). |
| `-l` | Chỉ định dung lượng theo **số extent** hoặc **phần trăm** VG, VD: `-l 100%FREE` = dùng toàn bộ dung lượng còn trống của VG. |

<img width="485" height="129" alt="image" src="https://github.com/user-attachments/assets/0d619f65-9fa6-41fb-9389-b3579a9d8a22" />

#### Bước 4: Tạo Filesystem trên LV

Định dạng chuẩn của Linux cho filesystem là `ext4`.

```bash
sudo mkfs.ext4 /dev/vg_data/data_lv
```

**Giải thích:**
- `mkfs.ext4`: format thiết bị theo định dạng ext4. Có thể thay bằng `mkfs.xfs` nếu muốn dùng XFS (thường dùng cho file lớn, hiệu năng cao, dễ mở rộng nhưng **không thể thu nhỏ** như ext4).

<img width="469" height="160" alt="image" src="https://github.com/user-attachments/assets/0eadd115-2645-45f3-88ed-7db55989e53d" />

#### Bước 5: Permanent Mount trên Ubuntu 22.04 

```bash
sudo mkdir -p /mnt/lvm_storage
```

**Lấy UUID của LV** (khuyến nghị dùng UUID thay vì đường dẫn `/dev/...` vì đường dẫn có thể đổi, còn UUID thì cố định):

```bash
sudo blkid /dev/vg_data/data_lv
```

Mở file fstab để cấu hình mount:

```bash
sudo nano /etc/fstab
```

Thêm dòng (dùng UUID lấy được ở trên):

```
UUID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx   /mnt/lvm_storage   ext4   defaults   0   2
```

Hoặc dùng trực tiếp đường dẫn logic (cũng hoạt động vì LVM tạo symlink cố định dạng `/dev/vg_name/lv_name`):

```
/dev/vg_data/data_lv   /mnt/lvm_storage   ext4   defaults   0   2
```

**Giải thích cột `Pass` = `2`:** khác với NFS (dùng `0`), ổ đĩa local ext4 nên đặt `1` (cho phân vùng root) hoặc `2` (cho các phân vùng phụ) để hệ thống chạy `fsck` kiểm tra lỗi filesystem khi boot.

Kiểm tra cấu hình trước khi reboot:

```bash
sudo mount -a
df -h | grep lvm_storage
```

Kết quả đạt được:

<img width="718" height="200" alt="image" src="https://github.com/user-attachments/assets/6e896c21-79ea-4d7a-a7ff-f56ce7968e05" />

<img width="488" height="167" alt="image" src="https://github.com/user-attachments/assets/c4c90f6e-25a0-4203-878f-d46944e5a985" />

---

### 2.3. Mở rộng LV (Extend)

```bash
sudo lvextend -L +1G /dev/vg_data/data_lv
sudo resize2fs /dev/vg_data/data_lv
```

**Giải thích:**
- `lvextend -L +1G`: `+/-L` = cộng/trừ thêm dung lượng vào mức hiện tại (khác với `-L 3G` là **đặt cứng** tổng dung lượng = 3GB).
- `resize2fs`: sau khi mở rộng LV, filesystem (ext4) bên trong **chưa tự động lớn theo** — cần lệnh này để filesystem nhận đúng dung lượng mới. Nếu dùng XFS thì thay bằng `sudo xfs_growfs /mnt/lvm_storage`.
- Có thể gộp cả 2 bước thành 1 lệnh: `sudo lvextend -L +1G -r /dev/vg_data/data_lv` (flag `-r` = tự động resize filesystem luôn, không cần chạy `resize2fs` riêng).

Kết quả đạt được:

<img width="611" height="100" alt="image" src="https://github.com/user-attachments/assets/a73a8ece-3de0-4693-b566-2b3bc2dfd47d" />

<img width="637" height="231" alt="image" src="https://github.com/user-attachments/assets/0de53268-e04f-4253-9061-fe4d27b1b336" />

---

### 2.4. Thêm ổ cứng mới để mở rộng LV

Sử dụng ổ cứng 10GB mới thêm vào VM.

#### Bước 1: Tạo Physical Volume

```bash
sudo pvcreate /dev/sdc
```

<img width="310" height="32" alt="image" src="https://github.com/user-attachments/assets/cebf9c61-9214-4f50-8c04-403465098cff" />

#### Bước 2: Mở rộng Volume Group

```bash
sudo vgextend vg_data /dev/sdc
```

**Giải thích:**
- `vgextend <VG> <PV_mới>`: thêm một PV mới vào VG đã tồn tại, giúp tăng tổng dung lượng khả dụng của VG.

<img width="313" height="98" alt="image" src="https://github.com/user-attachments/assets/bc36bdbb-df08-4974-bad3-7cd0519ca185" />

#### Bước 3: Mở rộng Logical Volume

```bash
sudo lvextend -L +5G /dev/vg_data/data_lv
sudo resize2fs /dev/vg_data/data_lv
```

<img width="634" height="100" alt="image" src="https://github.com/user-attachments/assets/bc91e76d-200c-48a5-9a9d-1943b9a6e0c8" />

---

### 2.5. Out / thay thế 1 ổ cứng khỏi hệ thống LVM

Việc này giúp di chuyển dữ liệu sang ổ mới nếu ổ cứng cũ sắp hỏng, thực hiện được **ngay cả khi hệ thống đang hoạt động** (online, không downtime).

#### Bước 1: Chuyển dữ liệu (pvmove)

```bash
sudo pvmove /dev/sdb
```

**Giải thích:**
- `pvmove <PV_nguồn>`: di chuyển toàn bộ dữ liệu (extent) đang nằm trên PV này sang các PV còn trống khác trong cùng VG, thực hiện online mà không gián đoạn dịch vụ.
- Có thể chỉ định đích cụ thể: `sudo pvmove /dev/sdb /dev/sdc`.

<img width="272" height="80" alt="image" src="https://github.com/user-attachments/assets/85adec1b-b816-4dfd-830a-a79cc3c50a26" />

#### Bước 2: Gỡ ổ cứng ra khỏi VG

```bash
sudo vgreduce vg_data /dev/sdb
```

**Giải thích:**
- `vgreduce <VG> <PV>`: loại bỏ một PV (đã rỗng dữ liệu, sau khi `pvmove`) ra khỏi VG.

<img width="302" height="28" alt="image" src="https://github.com/user-attachments/assets/f29042e2-a0ef-461f-9be8-90f4c086385a" />

#### Bước 3: Xóa định dạng LVM, tháo ổ

```bash
sudo pvremove /dev/sdb
```

**Giải thích:**
- `pvremove`: xóa metadata LVM khỏi ổ đĩa, trả ổ đĩa về trạng thái "sạch" — sau bước này có thể tháo ổ khỏi hệ thống an toàn.

<img width="565" height="85" alt="image" src="https://github.com/user-attachments/assets/4c89229f-47b7-4dc6-97d7-6cf32931f707" />

---

### 2.6. Xóa Logical Volume, Volume Group, Physical Volume

Xóa sẽ làm **ngược lại thứ tự tạo** (LV → VG → PV):

```bash
# 1. Ngắt mount
sudo umount /mnt/lvm_storage

# 2. Xóa logical volume
sudo lvremove /dev/vg_data/data_lv

# 3. Xóa volume group
sudo vgremove vg_data

# 4. Xóa physical volume
sudo pvremove /dev/sdb
```

**Lưu ý:** nếu đã cấu hình permanent mount ở `/etc/fstab`, phải **xóa/comment dòng tương ứng trong fstab** trước khi reboot — nếu không hệ thống sẽ báo lỗi hoặc treo khi boot vì cố mount một thiết bị không còn tồn tại.

<img width="562" height="206" alt="image" src="https://github.com/user-attachments/assets/4246cc59-f4a8-4185-be47-cfe278414e49" />

---

### 2.7. Bổ sung: các lệnh xem thông tin LVM thường dùng

> Phần này được bổ sung thêm dựa trên kiến thức chuẩn LVM — do file `3. Tìm hiểu, cấu hình LVM.md` bạn upload bị lỗi tải (GitHub trả về `429 Too Many Requests` khi bot cố lấy nội dung ảnh/link), nên nội dung gốc trong file đó **chưa đọc được**. Nếu file đó có thêm bước/ảnh khác với file chính này, bạn tải lại (re-upload) hoặc copy nội dung thô để mình bổ sung tiếp cho chính xác.

Các lệnh kiểm tra thông tin, rất hữu ích khi thi thực hành hoặc debug:

```bash
sudo pvs          # xem danh sách PV (dạng rút gọn)
sudo pvdisplay    # xem chi tiết từng PV
sudo vgs          # xem danh sách VG (dạng rút gọn)
sudo vgdisplay    # xem chi tiết từng VG
sudo lvs          # xem danh sách LV (dạng rút gọn)
sudo lvdisplay    # xem chi tiết từng LV
lsblk             # xem cây thiết bị khối (disk/partition/LV) trực quan
```

**Bảng phân biệt nhanh:**

| Lệnh | Cấp | Kiểu hiển thị |
|---|---|---|
| `pvs` / `vgs` / `lvs` | PV / VG / LV | Rút gọn, dạng bảng, xem nhanh |
| `pvdisplay` / `vgdisplay` / `lvdisplay` | PV / VG / LV | Chi tiết đầy đủ từng thuộc tính |

### 2.8. Bổ sung: Đổi tên LV/VG

```bash
sudo lvrename vg_data data_lv data_lv_new
sudo vgrename vg_data vg_data_new
```

### 2.9. Bổ sung: LVM Snapshot (chụp nhanh dữ liệu)

Snapshot dùng để tạo bản sao trạng thái của LV tại một thời điểm — hữu ích để backup an toàn trước khi thao tác rủi ro (VD: update, migrate dữ liệu).

```bash
sudo lvcreate -L 1G -s -n data_lv_snap /dev/vg_data/data_lv
```

**Giải thích:**
- `-s`: tạo snapshot thay vì LV thường.
- `-L 1G`: dung lượng dành riêng cho vùng lưu các thay đổi (delta) so với LV gốc — không phải dung lượng full copy dữ liệu.

Khi cần khôi phục lại trạng thái cũ:

```bash
sudo lvconvert --merge /dev/vg_data/data_lv_snap
```
