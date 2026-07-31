# Hướng dẫn đồng bộ dữ liệu Windows lên Wasabi Cloud Storage

Khi đăng ký gói Wasabi trial, cần chuẩn bị các thông tin sau:

- URL Console: https://console.wasabisys.com
- Email: 
- Password: 

Thông tin này dùng để đăng nhập, tạo bucket, quản lý file được đồng bộ lên cloud.

- ACCESS-KEY: 
- SECRET-KEY: 

Thông tin ACCESS-KEY, SECRET-KEY dùng để kết nối từ máy Windows lên bucket được tạo trên trang quản lý storage.

---

### Bước 1: Đăng ký tài khoản Wasabi trial và tạo Access Key / Secret Key

1. Truy cập: https://wasabi.com/trial-signup
2. Điền thông tin: Họ tên, Email, Số điện thoại, Tên công ty
3. Kiểm tra email và bấm link xác nhận để kích hoạt tài khoản
4. Đăng nhập vào Wasabi Console: https://console.wasabisys.com

Tạo Access Key / Secret Key:
1. Trong Console, vào mục Access Keys
2. Bấm Create New Access Key
3. Lưu lại `Access Key` và `Secret Key` (Secret Key chỉ hiện 1 lần, không xem lại được)

<img width="1611" height="772" alt="image" src="https://github.com/user-attachments/assets/03a8e2f8-e831-4525-b44e-a2430a99ee9e" />

---

## Bước 1: Tạo bucket trên trang quản lý storage

Đăng nhập vào https://console.wasabisys.com, sau đó bấm Create Bucket.

<img width="1626" height="630" alt="image" src="https://github.com/user-attachments/assets/db51f9f6-0a00-4dc5-a833-a06fe81213cc" />

Tại Bucket Name: Đặt tên cần tạo, không khoảng cách, không viết hoa, không có dấu.  
  
Chọn region cần dùng để tí kết nối trên Windows.  
Bucket Versioning: Tích chọn nếu muốn lưu nhiều phiên bản của cùng một object trong bucket.  
<img width="1399" height="823" alt="image" src="https://github.com/user-attachments/assets/c39ff6ad-6e4a-47c6-8f8f-64a8042ef7f5" />

Đây là tính năng sẽ ghi lại tất cả các lần truy cập. Khi bật Bucket Logging, Wasabi sẽ tạo các file log chứa thông tin chi tiết như: địa chỉ IP, thời gian truy cập, người dùng thực hiện thao tác, loại thao tác thực hiện trên object,…  

Bucket Tags – Gắn nhãn quản lý: Tính năng cho phép gắn các cặp khóa/giá trị (key/value) để phân loại, quản lý hoặc theo dõi chi phí của bucket  

<img width="1313" height="779" alt="image" src="https://github.com/user-attachments/assets/008ec071-3871-40ce-b629-f6a60a080eb3" />

<img width="1327" height="761" alt="image" src="https://github.com/user-attachments/assets/69a37a4d-57a7-48ee-a7e3-37dd4f63d3e1" />

Create hoàn tất  
<img width="1396" height="820" alt="image" src="https://github.com/user-attachments/assets/a5e14563-af31-421a-a73c-81080829d8e5" />

---

## Bước 2: Cài đặt Rclone

Tải Rclone cho Windows tại link: https://rclone.org/downloads/

Sau khi download về, tiến hành giải nén và đổi tên thư mục thành rclone. Copy vào ổ C

<img width="1127" height="634" alt="image" src="https://github.com/user-attachments/assets/fd9d1a25-0f6c-4f9d-af6d-d9f096658fb3" />


---

## Bước 3: Kết nối remote rclone vào bucket vừa tạo

Vào đường dẫn C:\rclone vừa copy ở trên, nhập vào thanh địa chỉ: cmd rồi Enter để mở Command Prompt ngay tại thư mục đó.

<img width="1120" height="632" alt="image" src="https://github.com/user-attachments/assets/45474a44-5464-40fc-bc00-8ec7071ce1ee" />

Gõ vào cmd: rclone config
<img width="977" height="515" alt="image" src="https://github.com/user-attachments/assets/cc8742a1-f51d-43b8-9cee-13400bab0a7f" />


Enter name for new remote.
`name> tenremote`  ← chỗ này đặt tên remote tuỳ ý (ví dụ: `wasabi`)

Option Storage.
Type of storage to configure.
Chọn số tương ứng với:
`4 / Amazon S3 Compliant Storage Providers including ... Wasabi ...`
`Storage> 4`

Option provider.
Choose your S3 provider.
Chọn số tương ứng với:
`.. / Wasabi Object Storage`
`provider> ..`  ← số thứ tự có thể thay đổi tuỳ phiên bản rclone, chỉ cần chọn đúng dòng có chữ Wasabi

Option env_auth.
`env_auth> 1`  ← chọn "Enter AWS credentials in the next step"

Option access_key_id.
`access_key_id> < ACCESS KEY >`  ← điền Access Key đã lưu ở Bước 0

Option secret_access_key.
`secret_access_key> < SECRET KEY >`  ← điền Secret Key đã lưu ở Bước 0

Option region.
`region> 1`  ← để mặc định (Use this if unsure)

Option endpoint.
Endpoint for S3 API.
Chọn endpoint tương ứng với region đã chọn lúc tạo bucket, ví dụ:
- `Wasabi US East 1 (N. Virginia)` → `s3.wasabisys.com`
- `Wasabi US East 2` → `s3.us-east-2.wasabisys.com`
- `Wasabi AP Southeast 1 (Singapore)` → `s3.ap-southeast-1.wasabisys.com`
`endpoint> ..`  ← chọn đúng region lúc tạo bucket

Option location_constraint.
`location_constraint> enter` (bỏ trống)

Option acl.
`acl> 1`  ← private (Owner gets FULL_CONTROL, No one else has access)

Edit advanced config?
`y/n> y`
 
Option bucket_acl.
Canned ACL used when creating buckets.
`bucket_acl> 1`  ← private (Owner gets FULL_CONTROL, No one else has access)
 
Các option còn lại (`upload_cutoff`, `chunk_size`, `max_upload_parts`, `copy_cutoff`, `disable_checksum`, `shared_credentials_file`, `profile`, `session_token`, `upload_concurrency`, `force_path_style`, `v2_auth`, `use_dual_stack`, `use_arn_region`, `list_chunk`, `list_version`, `list_url_encode`, `no_check_bucket`, `no_head`, `no_head_object`, `encoding`, `disable_http2`, `download_url`, `directory_markers`, `use_multipart_etag`, `use_unsigned_payload`, `use_presigned_request`, `versions`, `version_at`, `version_deleted`, `decompress`, `might_gzip`, `use_accept_encoding_gzip`, `use_already_exists`, `use_multipart_uploads`, `use_x_id`, `sign_accept_encoding`, `sdk_log_mode`, `description`...): cứ bấm Enter để bỏ qua, giữ mặc định.
 
Edit advanced config? (hỏi lại lần 2 sau khi đi hết danh sách)
`y/n> n`
 
Configuration complete.
```
Options:
- type: s3
- provider: Wasabi
- access_key_id: < ACCESS KEY >
- secret_access_key:
- endpoint: s3.wasabisys.com
- acl: private
Keep this "tenremote" remote?
y) Yes this is OK (default)
e) Edit this remote
d) Delete this remote
y/e/d> y
```
 
Current remotes:
```
Name       Type
====       ====
tenremote  s3
```
 
`e/n/d/r/c/s/q> q`  ← thoát cấu hình
 
 <img width="323" height="326" alt="image" src="https://github.com/user-attachments/assets/3b16f821-d714-4dd6-90dd-4104e33188a7" />

---

### Bước 4: Kiểm tra kết nối với bucket

Kiểm tra xem máy Windows đã kết nối được với bucket chưa:

```
rclone lsd tenremote:
```

<img width="749" height="80" alt="image" src="https://github.com/user-attachments/assets/7b8f48e0-4cf6-48b9-ba86-1336718293a3" />

có thể bắt đầu chuyển dữ liệu lên: rclone copy C:\data hieuwasabi:hieuwasabi -v –log-file=rclone.log

Trong đó:
- `C:\data` : là thư mục cần đồng bộ dữ liệu lên storage
- `tenremote` : là tên remote đặt lúc config
- `tenbucket` : là tên bucket đặt trên trang quản lý storage
- `rclone.log` : ghi lại log quá trình upload dữ liệu

<img width="544" height="103" alt="image" src="https://github.com/user-attachments/assets/fcea3f80-8ded-4eaa-9969-784070506e8d" />

Sau khi chạy đồng bộ xong, có thể kiểm tra dữ liệu trên trang Wasabi Console, hoặc dùng lệnh:
  
```
rclone ls tenremote:tenbucket
```
<img width="494" height="82" alt="image" src="https://github.com/user-attachments/assets/50216883-4b0d-4248-be55-118cecc8b493" />

Upload file thành công  
<img width="1881" height="933" alt="image" src="https://github.com/user-attachments/assets/13b39a39-e3f2-47f8-9cbc-07e22ae73eb6" />

---

## Bước 5: Đặt lịch tự đồng bộ dữ liệu daily

1. Bấm `Windows + R`, gõ: `taskschd.msc`
2. Vào Task Scheduler Library > Create Task
 <img width="783" height="540" alt="image" src="https://github.com/user-attachments/assets/06ccac56-bb87-487f-a08d-d7e453beb320" />
3. Tab General: đặt tên và mô tả cho task, ví dụ: `Syncdata`
4. Tab Triggers: bấm New → chọn Daily → đặt thời gian bắt đầu
<img width="1175" height="509" alt="image" src="https://github.com/user-attachments/assets/da523627-ecea-4044-9728-476cc2576d33" />

5. Tab Actions: bấm New → chọn Start a program
   - Program/script: `C:\rclone\rclone.exe`
   - Add arguments:
     ```
     copy C:\data tenremote:tenbucket -v --log-file=C:\rclone\rclone.log
     ```

     <img width="663" height="493" alt="image" src="https://github.com/user-attachments/assets/5aaed7bb-ea36-4e59-9ee0-b49902eabeb1" />

Mục setting, tích thêm cột fail  
<img width="625" height="471" alt="image" src="https://github.com/user-attachments/assets/81a07384-8624-4554-b846-30c7d29146ba" />

7. Bấm OK để lưu
8. Sau khi tạo xong, kiểm tra cột Status của task đã chuyển sang Ready

<img width="1920" height="407" alt="image" src="https://github.com/user-attachments/assets/5ac66903-65fa-417a-9d7e-57fcd23fd4fc" />

---

*Tài liệu tham khảo cấu trúc từ: [Nhân Hòa Knowledgebase - Hướng dẫn đồng bộ dữ liệu Windows lên S3](https://wiki.nhanhoa.com/kb/huong-dan-dong-bo-du-lieu-windows-len-s3-nhan-hoa-cloud-storage/), điều chỉnh lại cho dịch vụ Wasabi Cloud Storage.*
