# 2 – Quản lý User, Group, Quyền và Chủ sở hữu

---

## Mục lục

- [1. User](#1-user)
- [2. Group](#2-group)
- [3. Chủ sở hữu (Owner)](#3-chủ-sở-hữu-owner)
- [4. Quyền (Permissions)](#4-quyền-permissions)

---

## 1. User

### 1.1 Khái niệm

- **User** là tài khoản có thể đăng nhập và sử dụng hệ thống. Mỗi user có **username** và **password**.
- **Root (UID=0):** Super user, toàn quyền hệ thống.
- **Regular user (UID ≥ 1000 trên Ubuntu):** Người dùng thông thường, bị giới hạn quyền.
- **System user (UID 1–999):** Tài khoản dịch vụ (www-data, mysql...), không dùng để đăng nhập.

### 1.2 File lưu thông tin user

**`/etc/passwd`** – Lưu thông tin cơ bản của từng user:

```
username:password:UID:GID:comment:home_dir:shell
```

Ví dụ:
```
hieu:x:1001:1001:Nguyen Thanh Hieu:/home/hieu:/bin/bash
```

| Trường | Ý nghĩa |
|--------|---------|
| `hieu` | Tên đăng nhập |
| `x` | Mật khẩu đã giấu (lưu thực tế ở `/etc/shadow`) |
| `1001` | UID |
| `1001` | GID  |
| `Nguyen Thanh Hieu` | Mô tả  |
| `/home/hieu` | Thư mục home |
| `/bin/bash` | Shell mặc định |

<img width="427" height="388" alt="image" src="https://github.com/user-attachments/assets/d87b5399-0da0-4da8-9d9a-7da31bce62ec" />

**`/etc/shadow`** – Lưu mật khẩu đã hash (chỉ root đọc được):

```
username:hashed_password:last_change:min:max:warning:inactive:expired
```

<img width="686" height="95" alt="image" src="https://github.com/user-attachments/assets/855ab0c2-1b7b-4cc9-b4d5-74118e13c3d3" />

### 1.3 Thêm user

```bash
sudo useradd [options] <username>
```

| Option | Mô tả | Ví dụ |
|--------|-------|-------|
| `-m` | Tạo thư mục home tại `/home/<username>` | `useradd -m hieu` |
| `-s` | Chỉ định shell | `-s /bin/bash` |
| `-u` | Chỉ định UID | `-u 1500` |
| `-c` | Thêm mô tả (họ tên đầy đủ) | `-c "Nguyen Thanh Hieu"` |
| `-G` | Thêm vào group bổ sung | `-G sudo,www-data` |

**Ví dụ thực tế:**
```bash
# Tạo user với home dir, bash shell và mô tả
sudo useradd -m -s /bin/bash -c "Nguyen Thanh Hieu" hieu

# Đặt mật khẩu
sudo passwd hieu

# Tạo user không được phép đăng nhập shell (dùng cho service)
sudo useradd -r -s /sbin/nologin www-data
```


### 1.4 Sửa user

```bash
sudo usermod [options] <username>
```

| Option | Mô tả |
|--------|-------|
| `-c` | Sửa comment |
| `-d` | Sửa thư mục home |
| `-l` | Đổi tên đăng nhập |
| `-L` | Khóa tài khoản |
| `-U` | Mở khóa tài khoản |
| `-e YYYY-MM-DD` | Đặt ngày hết hạn |
| `-G <group>` | Gán vào group mới, rời khỏi tất cả group cũ |
| `-aG <group>` | Thêm vào group mới, giữ nguyên group cũ |

**Ví dụ:**
```bash
# Thêm user hieu vào group sudo (giữ nguyên group hiện tại)
sudo usermod -aG sudo hieu

# Khóa tài khoản
sudo usermod -L hieu

# Mở khóa tài khoản
sudo usermod -U hieu

# Đổi tên đăng nhập
sudo usermod -l hieu_new hieu
```

### 1.5 Xóa user

```bash
sudo userdel [options] <username>
```

| Option | Mô tả |
|--------|-------|
| `-r` | Xóa user và thư mục home (user phải đã logout) |
| `-f` | Buộc xóa kể cả user đang đăng nhập |

```bash
# Xóa user và toàn bộ home directory
sudo userdel -r hieu
```


### 1.6 Chuyển đổi user

```bash
# Chuyển sang user khác (cần nhập mật khẩu của user đích)
su <username>

# Chuyển sang root
sudo su

# Chạy lệnh với quyền user khác
sudo -u <username> <command>
```

> Để dùng `sudo`, user phải thuộc group `sudo` (Ubuntu) hoặc `wheel` (CentOS/AlmaLinux).

---

## 2. Group

### 2.1 Khái niệm

- **Group** là tập hợp các user. Dùng để gán quyền cho nhiều user cùng lúc.
- Mỗi user luôn thuộc ít nhất một group — **primary group** (group mặc định, tạo cùng lúc với user).
- User có thể thuộc nhiều **secondary group** bổ sung.
- Mỗi group có **GID** duy nhất.

### 2.2 File lưu thông tin group

**`/etc/group`** – Định dạng:

```
groupname:password:GID:member1,member2,...
```

<img width="199" height="87" alt="image" src="https://github.com/user-attachments/assets/04db9301-7f1b-4aab-95e1-97d6fbbfc890" />


### 2.3 Tạo group

```bash
sudo groupadd [options] <group_name>
```

```bash
# Tạo group
sudo groupadd developers

# Tạo group với GID cụ thể
sudo groupadd -g 1500 developers

# Đặt mật khẩu cho group
sudo gpasswd developers
```


### 2.4 Sửa group

```bash
sudo groupmod [options] <group_name>
```

| Option | Mô tả |
|--------|-------|
| `-g <gid>` | Sửa GID |
| `-n <tên mới>` | Đổi tên group |

```bash
# Đổi tên group
sudo groupmod -n devteam developers

# Sửa GID
sudo groupmod -g 1600 devteam
```


### 2.5 Xóa group

```bash
sudo groupdel <group_name>
```

> Không thể xóa group đang là primary group của bất kỳ user nào. Phải xóa hoặc chuyển user trước.

### 2.6 Kiểm tra thông tin user và group

```bash
# Xem các group của user hiện tại
groups

# Xem các group của user cụ thể
groups hieu

# Xem thông tin chi tiết user
id hieu

# Liệt kê user đang đăng nhập
who
w
```

---

## 3. Chủ sở hữu (Owner)

### 3.1 Khái niệm

Mỗi file và thư mục trên Linux đều có:
- **Owner (chủ sở hữu):** User sở hữu file
- **Group owner:** Group sở hữu file
- **Others:** Tất cả người dùng còn lại

### 3.2 Lệnh chown – Thay đổi chủ sở hữu

```bash
chown [options] <owner>[:<group>] <file/dir>
```

| Cú pháp | Ý nghĩa |
|---------|---------|
| `chown hieu file.txt` | Đổi owner thành `hieu` |
| `chown hieu:developers file.txt` | Đổi owner thành `hieu`, group thành `developers` |
| `chown :developers file.txt` | Chỉ đổi group, giữ nguyên owner |
| `chown -R hieu:developers /var/www/` | Đổi đệ quy cho cả thư mục |

| Option | Mô tả |
|--------|-------|
| `-R` | Áp dụng đệ quy cho toàn bộ thư mục con |
| `-H` | Với `-R`: theo symlink ở command line, xử lý nội dung bên trong |
| `-L` | Với `-R`: theo tất cả symlink trong quá trình duyệt |
| `-P` | Với `-R`: không theo symlink, chỉ đổi quyền sở hữu của symlink |

**Ví dụ thực tế:**

```bash
# Đổi owner file
sudo chown hieu report.txt

# Đổi cả owner lẫn group
sudo chown hieu:developers /var/www/html

# Áp dụng đệ quy
sudo chown -R www-data:www-data /var/www/

# Sao chép owner từ file này sang file khác
sudo chown --reference=source.txt target.txt
```


---

## 4. Quyền (Permissions)

### 4.1 Ba loại quyền

| Ký hiệu | Tên | Giá trị số | Với file | Với thư mục |
|---------|-----|------------|----------|-------------|
| `r` | Read | 4 | Đọc nội dung file | Liệt kê nội dung (`ls`) |
| `w` | Write | 2 | Chỉnh sửa nội dung | Tạo/xóa file bên trong |
| `x` | Execute | 1 | Chạy file thực thi | Vào thư mục (`cd`) |

### 4.2 Ba đối tượng áp dụng

```
-rwxr-xr--  1  hieu  developers  4096  Jul 3  file.sh
 ^^^         ← owner (hieu): rwx = 7
    ^^^      ← group (developers): r-x = 5
       ^^^   ← others: r-- = 4
```

Ký tự đầu tiên chỉ loại:
- `-` : File thường
- `d` : Directory
- `l` : Symbolic link

### 4.3 Xem quyền

```bash
# Xem quyền dạng dài
ls -l

# Xem quyền chi tiết một file
stat file.txt

# Xem quyền thư mục hiện tại
ls -la
```


### 4.4 Lệnh chmod – Thay đổi quyền

#### Chế độ số (Absolute Mode)

Kết hợp các giá trị: `r=4`, `w=2`, `x=1`

| Giá trị | Quyền | Ký hiệu |
|---------|-------|---------|
| 0 | Không có quyền | `---` |
| 1 | Chỉ execute | `--x` |
| 2 | Chỉ write | `-w-` |
| 3 | Write + Execute | `-wx` |
| 4 | Chỉ read | `r--` |
| 5 | Read + Execute | `r-x` |
| 6 | Read + Write | `rw-` |
| 7 | Đầy đủ | `rwx` |

```bash
# Owner: rwx(7), Group: r-x(5), Others: r--(4)
chmod 754 file.sh

# Quyền phổ biến với file web
chmod 644 index.html      # rw-r--r--
chmod 755 /var/www/html   # rwxr-xr-x

# Áp dụng đệ quy
chmod -R 755 /var/www/
```

#### Chế độ ký hiệu 

```
chmod [who][operator][permission] file
```

| Who | Ý nghĩa |
|-----|---------|
| `u` | User (owner) |
| `g` | Group |
| `o` | Others |
| `a` | All (tất cả) |

| Operator | Ý nghĩa |
|----------|---------|
| `+` | Thêm quyền |
| `-` | Bỏ quyền |
| `=` | Đặt quyền cụ thể (ghi đè) |

```bash
# Thêm quyền execute cho owner
chmod u+x script.sh

# Bỏ quyền write của others
chmod o-w file.txt

# Đặt group chỉ có read
chmod g=r file.txt

# Thêm execute cho tất cả
chmod a+x script.sh
```


### 4.5 Quyền đặc biệt

| Quyền | Giá trị | Áp dụng | Tác dụng |
|-------|---------|---------|---------|
| **SUID** | 4000 | File | Chạy file với quyền của owner, không phải người chạy |
| **SGID** | 2000 | File/Dir | File: chạy với quyền group owner. Dir: file tạo mới kế thừa group của thư mục |
| **Sticky Bit** | 1000 | Directory | Chỉ owner file mới được xóa file của mình trong thư mục (dùng cho `/tmp`) |

```bash
# Đặt SUID
chmod u+s /usr/bin/passwd
chmod 4755 /usr/bin/passwd

# Đặt SGID trên thư mục
chmod g+s /shared/
chmod 2775 /shared/

# Đặt Sticky Bit
chmod +t /tmp/
chmod 1777 /tmp/
```

### 4.6 umask – Quyền mặc định khi tạo file/thư mục

`umask` xác định quyền bị **loại bỏ** khi tạo file/thư mục mới.

- Quyền tối đa của file: **666**
- Quyền tối đa của thư mục: **777**

Nếu `umask = 022`:
- File mới: `666 - 022 = 644` (rw-r--r--)
- Thư mục mới: `777 - 022 = 755` (rwxr-xr-x)

```bash
# Xem umask hiện tại
umask

# Thay đổi umask trong phiên hiện tại
umask 027

# Đặt umask vĩnh viễn (thêm vào ~/.bashrc hoặc /etc/profile)
echo "umask 027" >> ~/.bashrc
```

---

## Bảng tóm tắt lệnh

| Lệnh | Mô tả |
|------|-------|
| `useradd -m -s /bin/bash <user>` | Tạo user với home dir |
| `passwd <user>` | Đặt/đổi mật khẩu |
| `usermod -aG <group> <user>` | Thêm user vào group |
| `userdel -r <user>` | Xóa user và home dir |
| `groupadd <group>` | Tạo group |
| `groupdel <group>` | Xóa group |
| `id <user>` | Xem UID, GID, các group của user |
| `groups <user>` | Xem danh sách group của user |
| `chown user:group file` | Đổi owner và group |
| `chmod 755 file` | Đặt quyền theo số |
| `chmod u+x file` | Thêm quyền theo ký hiệu |
| `ls -l` | Xem quyền file |
| `stat file` | Xem thông tin chi tiết file |
| `umask` | Xem quyền mặc định |

---

## References

- [User & Group Linux – SunCloud](https://suncloud.vn/quan-ly-nguoi-dung-user-va-nhom-group-trong-linux)
- [Basic terminal commands for Ubuntu 22.04 – Medium](https://medium.com/@photon17/basic-terminal-command-lines-for-ubuntu-22-04-beginners-51d83f8a1132)
