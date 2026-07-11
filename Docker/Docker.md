# Docker – Tổng quan và Triển khai thực tế

> **Môi trường thực hành:** Ubuntu 22.04 LTS
> **Phiên bản:** Docker Engine 26.x · Docker Compose v2.x
> **Mục tiêu:** Nắm vững lý thuyết, cài đặt, vận hành Docker và triển khai ứng dụng thực tế bằng Dockerfile + Docker Compose

---

## Mục lục

- [1. Tổng quan Docker](#1-tổng-quan-docker)
  - [1.1 Vì sao cần Docker](#11-vì-sao-cần-docker)
  - [1.2 Lịch sử hình thành](#12-lịch-sử-hình-thành)
  - [1.3 Docker là gì](#13-docker-là-gì)
  - [1.4 So sánh Container và Virtual Machine](#14-so-sánh-container-và-virtual-machine)
- [2. Kiến trúc Docker](#2-kiến-trúc-docker)
  - [2.1 Docker Engine](#21-docker-engine)
  - [2.2 Docker Image](#22-docker-image)
  - [2.3 Docker Container](#23-docker-container)
  - [2.4 Docker Registry](#24-docker-registry)
  - [2.5 Docker Volume](#25-docker-volume)
  - [2.6 Docker Network](#26-docker-network)
- [3. Cài đặt Docker trên Ubuntu 22.04](#3-cài-đặt-docker-trên-ubuntu-2204)
  - [3.1 Gỡ phiên bản cũ (nếu có)](#31-gỡ-phiên-bản-cũ-nếu-có)
  - [3.2 Cài đặt Docker Engine](#32-cài-đặt-docker-engine)
  - [3.3 Cài đặt Docker Compose](#33-cài-đặt-docker-compose)
  - [3.4 Cấu hình sau cài đặt](#34-cấu-hình-sau-cài-đặt)
- [4. Làm việc với Docker Image](#4-làm-việc-với-docker-image)
  - [4.1 Các lệnh Image cơ bản](#41-các-lệnh-image-cơ-bản)
  - [4.2 Docker Hub](#42-docker-hub)
- [5. Làm việc với Docker Container](#5-làm-việc-với-docker-container)
  - [5.1 Vòng đời Container](#51-vòng-đời-container)
  - [5.2 Các lệnh Container quan trọng](#52-các-lệnh-container-quan-trọng)
  - [5.3 Port Mapping và Environment Variables](#53-port-mapping-và-environment-variables)
- [6. Docker Volume – Quản lý dữ liệu bền vững](#6-docker-volume--quản-lý-dữ-liệu-bền-vững)
  - [6.1 Các loại Volume](#61-các-loại-volume)
  - [6.2 Lệnh quản lý Volume](#62-lệnh-quản-lý-volume)
- [7. Docker Network](#7-docker-network)
  - [7.1 Các loại Network](#71-các-loại-network)
  - [7.2 Lệnh quản lý Network](#72-lệnh-quản-lý-network)
- [8. Dockerfile – Đóng gói ứng dụng thành Image](#8-dockerfile--đóng-gói-ứng-dụng-thành-image)
  - [8.1 Dockerfile là gì](#81-dockerfile-là-gì)
  - [8.2 Các chỉ thị quan trọng trong Dockerfile](#82-các-chỉ-thị-quan-trọng-trong-dockerfile)
  - [8.3 Ví dụ thực tế – Dockerfile cho ứng dụng Node.js](#83-ví-dụ-thực-tế--dockerfile-cho-ứng-dụng-nodejs)
  - [8.4 Ví dụ thực tế – Dockerfile cho ứng dụng PHP](#84-ví-dụ-thực-tế--dockerfile-cho-ứng-dụng-php)
  - [8.5 Multi-stage Build](#85-multi-stage-build)
- [9. Docker Compose – Quản lý ứng dụng đa container](#9-docker-compose--quản-lý-ứng-dụng-đa-container)
  - [9.1 Docker Compose là gì](#91-docker-compose-là-gì)
  - [9.2 Cú pháp file docker-compose.yml](#92-cú-pháp-file-docker-composeyml)
  - [9.3 Các lệnh Docker Compose](#93-các-lệnh-docker-compose)
- [10. Thực chiến – Deploy WordPress với Docker Compose](#10-thực-chiến--deploy-wordpress-với-docker-compose)
- [11. Thực chiến – Deploy ứng dụng Node.js + MongoDB](#11-thực-chiến--deploy-ứng-dụng-nodejs--mongodb)
- [12. Bảo mật Docker cơ bản](#12-bảo-mật-docker-cơ-bản)
- [13. Lỗi thường gặp và cách xử lý](#13-lỗi-thường-gặp-và-cách-xử-lý)
- [14. Bảng tóm tắt lệnh quan trọng](#14-bảng-tóm-tắt-lệnh-quan-trọng)
- [References](#references)

---

## 1. Tổng quan Docker

### 1.1 Vì sao cần Docker

Hãy tưởng tượng bạn vừa được phân công vào một dự án mới. Mở `README.md` ra thấy yêu cầu cài: Node.js 18, React, Redis 7, PostgreSQL 14, Elasticsearch 8... mỗi thứ một version cụ thể.

Cài thủ công từng cái:
- Mất nửa ngày setup.
- Dễ xung đột với phần mềm đã cài trước đó.
- Máy đồng nghiệp khác version, code chạy khác nhau.
- Lên production lại phải setup lại từ đầu.

**Docker giải quyết tất cả chỉ bằng vài dòng lệnh:** đóng gói toàn bộ ứng dụng + dependencies + cấu hình vào một "hộp" (container), chạy đồng nhất trên mọi môi trường — dev, staging, production.

> **"Works on my machine" → với Docker: "Works on every machine"**

### 1.2 Lịch sử hình thành

**Trước khi có Docker — 3 thế hệ hạ tầng:**

**Thế hệ 1 — Physical Server (Máy chủ vật lý):**
```
[Physical Server]
 └── OS
      └── App A + App B + App C
```
- Toàn bộ ứng dụng chạy trên cùng 1 máy vật lý.
- Vấn đề: App A và App B xung đột tài nguyên, không thể cô lập.
- Không tận dụng hết tài nguyên — lãng phí phần cứng.

**Thế hệ 2 — Virtual Machine (Máy ảo):**
```
[Physical Server]
 └── Hypervisor (VMware, VirtualBox...)
      ├── VM1: OS + App A
      ├── VM2: OS + App B
      └── VM3: OS + App C
```
- Mỗi app chạy trong VM riêng, cô lập hoàn toàn.
- Vấn đề: Mỗi VM cần cả một OS riêng → nặng, khởi động chậm (vài phút), chiếm nhiều RAM/disk.

**Thế hệ 3 — Container (Hiện tại):**
```
[Physical Server]
 └── OS (Linux Kernel)
      └── Docker Engine
           ├── Container A: App A + libs
           ├── Container B: App B + libs
           └── Container C: App C + libs
```
- Containers **chia sẻ chung kernel** của host OS — không cần OS riêng cho mỗi app.
- Khởi động trong vài **giây** (so với vài phút của VM).
- Nhẹ hơn VM **10–100 lần**.

**Mốc lịch sử Docker:**

| Năm | Sự kiện |
|-----|---------|
| 2008 | LXC (Linux Containers) ra đời — nền tảng kỹ thuật cho container |
| 2013 | Solomon Hykes giới thiệu Docker tại PyCon — mã nguồn mở |
| 2014 | Docker 1.0 ra mắt chính thức |
| 2015 | Docker Compose, Docker Swarm ra mắt |
| 2017 | Kubernetes trở thành chuẩn orchestration; Docker tích hợp K8s |
| 2019 | Docker Enterprise bán cho Mirantis; Docker Hub ra giới hạn pull |
| 2023–nay | Docker Desktop cải thiện mạnh; Docker Compose v2 mặc định |

### 1.3 Docker là gì

**Docker** là nền tảng phần mềm mã nguồn mở giúp tự động hóa việc triển khai ứng dụng trong các container — các gói phần mềm nhẹ, di động, tự cung cấp mọi thứ cần thiết để ứng dụng chạy được.

Docker gồm 2 thành phần chính:
- **Docker Engine:** Công cụ tạo và chạy container (chạy trên Linux/Mac/Windows).
- **Docker Hub:** Registry (kho lưu trữ) image public — hơn 100.000 image sẵn có.

### 1.4 So sánh Container và Virtual Machine

| Tiêu chí | Container (Docker) | Virtual Machine |
|----------|-------------------|----------------|
| **Kích thước** | MB (chỉ chứa app + libs) | GB (cả OS đầy đủ) |
| **Khởi động** | Vài giây | Vài phút |
| **Cô lập** | Process-level (chia sẻ kernel) | Hoàn toàn (kernel riêng) |
| **Hiệu năng** | Gần native | Có overhead do hypervisor |
| **Di động** | Rất cao — chạy mọi nơi có Docker | Thấp hơn — phụ thuộc hypervisor |
| **Bảo mật** | Chia sẻ kernel → rủi ro hơn | Cô lập hoàn toàn → an toàn hơn |
| **Phù hợp** | Microservices, CI/CD, dev environment | Chạy OS khác nhau, bảo mật cao |

---

## 2. Kiến trúc Docker

```
┌────────────────────────────────────────────────────┐
│                   Docker Client                    │
│            (docker CLI / Docker Desktop)           │
└───────────────────────┬────────────────────────────┘
                        │ REST API
┌───────────────────────▼────────────────────────────┐
│                   Docker Daemon                    │
│                   (dockerd)                        │
│  ┌──────────┐  ┌───────────┐  ┌────────────────┐  │
│  │  Images  │  │Containers │  │    Networks     │  │
│  └──────────┘  └───────────┘  └────────────────┘  │
│  ┌──────────┐  ┌───────────────────────────────┐   │
│  │  Volumes │  │         Registry              │   │
│  └──────────┘  │  (Docker Hub / Private)       │   │
│                └───────────────────────────────┘   │
└────────────────────────────────────────────────────┘
```

### 2.1 Docker Engine

Docker Engine là core của Docker, gồm 3 phần:

| Thành phần | Mô tả |
|-----------|-------|
| **Docker Daemon (dockerd)** | Tiến trình chạy nền, quản lý image, container, network, volume |
| **Docker Client (docker CLI)** | Giao diện dòng lệnh người dùng gõ lệnh `docker ...` |
| **REST API** | Kênh giao tiếp giữa Client và Daemon |

### 2.2 Docker Image

**Image** là template bất biến (read-only) chứa đầy đủ mọi thứ cần thiết để chạy ứng dụng: code, runtime, thư viện, biến môi trường, cấu hình.

- Image được xây dựng từ **Dockerfile**.
- Image có tính **phân lớp (layered)**: mỗi lệnh trong Dockerfile tạo ra một layer; các layer được cache và tái sử dụng.
- Image **không thay đổi** sau khi build — mọi thay đổi khi chạy được ghi vào container layer riêng.

```
Image: ubuntu:22.04
 Layer 1: Base Ubuntu filesystem
 Layer 2: apt install nginx
 Layer 3: COPY config files
 Layer 4: EXPOSE 80
```

### 2.3 Docker Container

**Container** là một instance đang chạy của Image. Nếu Image là "bản thiết kế" thì Container là "ngôi nhà" được xây từ bản thiết kế đó.

- Từ 1 Image có thể tạo ra **nhiều Container** chạy song song.
- Container có **writable layer** riêng — dữ liệu ghi vào container layer sẽ mất khi container bị xóa (dùng Volume để persist data).
- Container **cô lập** với host và các container khác (network, process, filesystem).

### 2.4 Docker Registry

**Registry** là nơi lưu trữ và phân phối Docker Image.

| Registry | Mô tả |
|---------|-------|
| **Docker Hub** | Registry public mặc định — `hub.docker.com` |
| **GitHub Container Registry** | Tích hợp với GitHub Actions |
| **Amazon ECR** | Registry của AWS |
| **Google Container Registry** | Registry của GCP |
| **Private Registry** | Tự host với `registry:2` image |

### 2.5 Docker Volume

**Volume** là cơ chế lưu trữ dữ liệu bền vững cho container. Dữ liệu trong Volume **không bị mất** khi container bị xóa hay restart.

3 loại mount:
- **Named Volume:** Docker quản lý, lưu tại `/var/lib/docker/volumes/` — khuyến nghị.
- **Bind Mount:** Map thư mục host vào container — dùng trong development.
- **tmpfs Mount:** Lưu trong RAM, mất khi container dừng — dùng cho sensitive data tạm.

### 2.6 Docker Network

**Network** cho phép các container giao tiếp với nhau và với host.

| Loại | Mô tả | Dùng khi |
|------|-------|---------|
| **bridge** | Mặc định — container trong cùng bridge network giao tiếp được với nhau | Container đơn lẻ hoặc Compose |
| **host** | Container dùng network stack của host — không có NAT | Cần hiệu năng network cao nhất |
| **none** | Hoàn toàn cô lập mạng | Container cần bảo mật tuyệt đối |
| **overlay** | Kết nối container trên nhiều host | Docker Swarm, multi-host |
| **macvlan** | Gán MAC address riêng cho container | Tích hợp legacy network |

---

## 3. Cài đặt Docker trên Ubuntu 22.04

### 3.1 Gỡ phiên bản cũ (nếu có)

```bash
sudo apt remove docker docker-engine docker.io containerd runc
```

### 3.2 Cài đặt Docker Engine

**Bước 1:** Cập nhật và cài dependencies:

```bash
sudo apt update
sudo apt install -y ca-certificates curl gnupg lsb-release
```

**Bước 2:** Thêm GPG key chính thức của Docker:

```bash
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
  | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

**Bước 3:** Thêm repository Docker:

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" \
  | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

**Bước 4:** Cài đặt Docker Engine:

```bash
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

**Bước 5:** Kiểm tra:

```bash
sudo systemctl status docker
sudo docker version
sudo docker run hello-world
```

Kết quả đúng:
```
Hello from Docker!
This message shows that your installation appears to be working correctly.
```

### 3.3 Cài đặt Docker Compose

Docker Compose v2 đã được cài tích hợp qua `docker-compose-plugin` ở bước trên. Kiểm tra:

```bash
docker compose version
# Docker Compose version v2.x.x
```

>  Docker Compose v2 dùng lệnh `docker compose` (có dấu cách), thay vì `docker-compose` (có dấu gạch) của v1 cũ.

### 3.4 Cấu hình sau cài đặt

**Chạy Docker không cần sudo:**

```bash
# Thêm user hiện tại vào group docker
sudo usermod -aG docker $USER

# Áp dụng ngay (không cần logout)
newgrp docker

# Kiểm tra
docker run hello-world
```

**Bật auto-start:**

```bash
sudo systemctl enable docker
sudo systemctl enable containerd
```

**Kiểm tra thông tin hệ thống Docker:**

```bash
docker info
```

---

## 4. Làm việc với Docker Image

### 4.1 Các lệnh Image cơ bản

```bash
# Tải image từ Docker Hub
docker pull nginx                        # Tải tag mới nhất
docker pull nginx:1.25                   # Tải tag cụ thể
docker pull ubuntu:22.04

# Liệt kê image đang có trên máy
docker images
docker image ls

# Xem thông tin chi tiết image
docker inspect nginx

# Xem lịch sử các layer của image
docker history nginx

# Xóa image
docker rmi nginx
docker image rm nginx:1.25

# Xóa toàn bộ image không dùng
docker image prune
docker image prune -a                    # Xóa cả image không có container nào dùng

# Tìm kiếm image trên Docker Hub
docker search nginx

# Tag image
docker tag nginx:latest myrepo/nginx:v1.0

# Push image lên registry
docker push myrepo/nginx:v1.0
```

### 4.2 Docker Hub

Truy cập [hub.docker.com](https://hub.docker.com) để:
- Tìm kiếm image official (nginx, mysql, redis, node, python...).
- Xem các tag có sẵn của từng image.
- Tạo private repository cho team.

**Đăng nhập Docker Hub từ CLI:**

```bash
docker login
# Nhập username và password/token
```

---

## 5. Làm việc với Docker Container

### 5.1 Vòng đời Container

```
docker pull image
      │
      ▼
   [Image]
      │ docker create / docker run
      ▼
  [Created]
      │ docker start
      ▼
  [Running] ◄──── docker restart
      │               │
      │ docker pause  │
      ▼               │
  [Paused]            │
      │ docker unpause │
      └───────────────►│
      │
      │ docker stop (SIGTERM → SIGKILL sau 10s)
      ▼
  [Stopped/Exited]
      │ docker rm
      ▼
   [Deleted]
```

### 5.2 Các lệnh Container quan trọng

```bash
# Tạo và chạy container
docker run nginx                         # Chạy foreground, Ctrl+C để dừng
docker run -d nginx                      # Chạy background (detached)
docker run -d --name my-nginx nginx      # Đặt tên container
docker run -it ubuntu:22.04 bash        # Chạy interactive với terminal

# Liệt kê container
docker ps                               # Đang chạy
docker ps -a                            # Tất cả (kể cả đã dừng)
docker ps -a --format "table {{.ID}}\t{{.Names}}\t{{.Status}}"

# Xem log
docker logs my-nginx                    # Xem toàn bộ log
docker logs -f my-nginx                 # Theo dõi log realtime
docker logs --tail 50 my-nginx          # Xem 50 dòng cuối

# Vào trong container đang chạy
docker exec -it my-nginx bash           # Mở terminal bash
docker exec -it my-nginx sh             # Mở terminal sh (nếu không có bash)
docker exec my-nginx nginx -t           # Chạy lệnh cụ thể

# Dừng, khởi động lại, xóa
docker stop my-nginx                    # Dừng nhẹ nhàng (SIGTERM)
docker kill my-nginx                    # Dừng ngay lập tức (SIGKILL)
docker start my-nginx                   # Khởi động lại
docker restart my-nginx                 # Stop + Start

docker rm my-nginx                      # Xóa container (phải stop trước)
docker rm -f my-nginx                   # Xóa ngay kể cả đang chạy
docker container prune                  # Xóa tất cả container đã dừng

# Xem tài nguyên container đang dùng
docker stats                            # Realtime CPU/RAM/Network
docker stats --no-stream               # Xem một lần

# Copy file giữa host và container
docker cp index.html my-nginx:/usr/share/nginx/html/
docker cp my-nginx:/etc/nginx/nginx.conf ./nginx.conf

# Xem thông tin chi tiết
docker inspect my-nginx
```

### 5.3 Port Mapping và Environment Variables

```bash
# Map port: -p <host_port>:<container_port>
docker run -d -p 8080:80 nginx          # http://localhost:8080 → nginx port 80
docker run -d -p 3306:3306 mysql:8.0

# Truyền biến môi trường
docker run -d \
  -e MYSQL_ROOT_PASSWORD=secret \
  -e MYSQL_DATABASE=mydb \
  -e MYSQL_USER=user1 \
  -e MYSQL_PASSWORD=pass123 \
  mysql:8.0

# Đọc từ file .env
docker run --env-file .env mysql:8.0

# Giới hạn tài nguyên
docker run -d \
  --memory="512m" \
  --cpus="1.5" \
  nginx
```

---

## 6. Docker Volume – Quản lý dữ liệu bền vững

### 6.1 Các loại Volume

**Named Volume (khuyến nghị cho production):**
```bash
# Docker tự quản lý, lưu tại /var/lib/docker/volumes/
docker run -d -v mydata:/var/lib/mysql mysql:8.0
```

**Bind Mount (dùng trong development):**
```bash
# Map thư mục host vào container — thay đổi ngay lập tức
docker run -d -v /home/hieu/app:/var/www/html nginx
docker run -d -v $(pwd):/app node:18    # Dùng thư mục hiện tại
```

**Read-only Mount:**
```bash
docker run -d -v /home/hieu/config:/etc/nginx/conf.d:ro nginx
```

### 6.2 Lệnh quản lý Volume

```bash
# Tạo volume
docker volume create mydata

# Liệt kê volume
docker volume ls

# Xem thông tin chi tiết
docker volume inspect mydata

# Xóa volume
docker volume rm mydata

# Xóa toàn bộ volume không dùng
docker volume prune
```

---

## 7. Docker Network

### 7.1 Các loại Network

```bash
# Xem network hiện có
docker network ls

# Kết quả mặc định:
# NETWORK ID   NAME      DRIVER    SCOPE
# xxx          bridge    bridge    local     ← Mặc định cho container đơn lẻ
# xxx          host      host      local     ← Dùng network của host
# xxx          none      null      local     ← Cô lập hoàn toàn
```

### 7.2 Lệnh quản lý Network

```bash
# Tạo network
docker network create mynetwork
docker network create --driver bridge --subnet 172.20.0.0/16 mynetwork

# Chạy container trong network cụ thể
docker run -d --network mynetwork --name app nginx
docker run -d --network mynetwork --name db mysql:8.0

# Container trong cùng network có thể giao tiếp qua tên:
# app container có thể ping db bằng hostname "db"

# Gán container vào network
docker network connect mynetwork my-container

# Tách container khỏi network
docker network disconnect mynetwork my-container

# Xem container nào trong network
docker network inspect mynetwork

# Xóa network
docker network rm mynetwork
docker network prune                    # Xóa network không dùng
```

---

## 8. Dockerfile – Đóng gói ứng dụng thành Image

### 8.1 Dockerfile là gì

**Dockerfile** là file văn bản chứa các chỉ thị (instructions) để Docker build một Image. Mỗi chỉ thị tạo ra một layer trong Image — các layer được cache nên build lần sau nhanh hơn nếu không có thay đổi.

### 8.2 Các chỉ thị quan trọng trong Dockerfile

| Chỉ thị | Cú pháp | Mô tả |
|---------|---------|-------|
| `FROM` | `FROM ubuntu:22.04` | Base image — bắt buộc phải có, luôn đặt đầu tiên |
| `WORKDIR` | `WORKDIR /app` | Đặt thư mục làm việc trong container |
| `COPY` | `COPY . /app` | Copy file từ host vào image |
| `ADD` | `ADD file.tar.gz /app` | Như COPY nhưng tự giải nén .tar và fetch URL |
| `RUN` | `RUN apt install -y nginx` | Chạy lệnh khi BUILD image — cài thư viện, cấu hình |
| `CMD` | `CMD ["nginx", "-g", "daemon off;"]` | Lệnh mặc định khi container KHỞI ĐỘNG — override được |
| `ENTRYPOINT` | `ENTRYPOINT ["node"]` | Điểm vào container — khó override hơn CMD |
| `ENV` | `ENV NODE_ENV=production` | Đặt biến môi trường trong image |
| `ARG` | `ARG VERSION=1.0` | Biến truyền vào lúc BUILD (không có trong runtime) |
| `EXPOSE` | `EXPOSE 80` | Khai báo port container lắng nghe (chỉ là metadata, không mở port thật) |
| `VOLUME` | `VOLUME ["/data"]` | Tạo mount point — data ở đây sẽ persist |
| `USER` | `USER node` | Chạy các lệnh tiếp theo với user cụ thể (bảo mật) |
| `LABEL` | `LABEL version="1.0"` | Thêm metadata cho image |
| `.dockerignore` | *(file riêng)* | Loại trừ file khi COPY — giống .gitignore |

>  **Phân biệt RUN vs CMD vs ENTRYPOINT:**
> - `RUN`: Chạy **khi build** image → cài phần mềm, tạo file cấu hình.
> - `CMD`: Lệnh mặc định **khi start** container → dễ override bằng `docker run image <lệnh_khác>`.
> - `ENTRYPOINT`: Luôn chạy **khi start** container → khó override, dùng khi muốn container hoạt động như một executable.

### 8.3 Ví dụ thực tế – Dockerfile cho ứng dụng Node.js

**Cấu trúc project:**
```
my-app/
├── Dockerfile
├── .dockerignore
├── package.json
├── package-lock.json
└── src/
    └── index.js
```

**File `.dockerignore`:**
```
node_modules
npm-debug.log
.git
.env
```

**File `Dockerfile`:**
```dockerfile
# Bước 1: Chọn base image
FROM node:18-alpine

# Bước 2: Đặt thư mục làm việc
WORKDIR /app

# Bước 3: Copy package.json TRƯỚC (tận dụng layer cache)
# Nếu package.json không đổi, Docker dùng cache → không cần npm install lại
COPY package*.json ./

# Bước 4: Cài dependencies
RUN npm ci --only=production

# Bước 5: Copy toàn bộ source code
COPY . .

# Bước 6: Khai báo port
EXPOSE 3000

# Bước 7: Chạy bằng user không phải root (bảo mật)
USER node

# Bước 8: Lệnh khởi động
CMD ["node", "src/index.js"]
```

**Build và chạy:**
```bash
# Build image với tag
docker build -t my-node-app:1.0 .

# Xem image vừa build
docker images | grep my-node-app

# Chạy container
docker run -d -p 3000:3000 --name my-app my-node-app:1.0

# Kiểm tra
curl http://localhost:3000
```

### 8.4 Ví dụ thực tế – Dockerfile cho ứng dụng PHP

```dockerfile
FROM php:8.2-apache

# Cài extension PHP cần thiết
RUN docker-php-ext-install pdo pdo_mysql mysqli

# Bật mod_rewrite cho Apache (cần cho WordPress/Laravel)
RUN a2enmod rewrite

# Copy source code
COPY . /var/www/html/

# Phân quyền
RUN chown -R www-data:www-data /var/www/html

EXPOSE 80

CMD ["apache2-foreground"]
```

### 8.5 Multi-stage Build

Multi-stage build tạo image nhỏ hơn bằng cách tách biệt môi trường build và runtime:

```dockerfile
# Stage 1: Build
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build                       # Build ra thư mục /app/dist

# Stage 2: Production (chỉ copy output, bỏ toàn bộ dev tools)
FROM node:18-alpine AS production
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/package*.json ./
RUN npm ci --only=production
EXPOSE 3000
USER node
CMD ["node", "dist/index.js"]
```

>  Image production từ multi-stage build thường nhỏ hơn **10–20 lần** so với image build thông thường vì không chứa dev dependencies, source code gốc, build tools.

---

## 9. Docker Compose – Quản lý ứng dụng đa container

### 9.1 Docker Compose là gì

**Docker Compose** là công cụ định nghĩa và chạy ứng dụng gồm **nhiều container** thông qua một file `docker-compose.yml`. Thay vì gõ nhiều lệnh `docker run` phức tạp, chỉ cần `docker compose up`.

**Ví dụ so sánh:**

Không có Compose (phải chạy 3 lệnh riêng):
```bash
docker network create mynet
docker run -d --network mynet -e MYSQL_ROOT_PASSWORD=secret --name db mysql:8.0
docker run -d --network mynet -p 8080:80 --name app --link db:db wordpress
```

Có Compose (1 file, 1 lệnh):
```bash
docker compose up -d
```

### 9.2 Cú pháp file docker-compose.yml

```yaml
# Phiên bản schema (tùy chọn trong Compose v2)
version: "3.9"

services:
  # Định nghĩa từng container
  web:
    image: nginx:1.25                   # Dùng image có sẵn
    # hoặc build từ Dockerfile:
    # build:
    #   context: .
    #   dockerfile: Dockerfile
    container_name: my-web             # Tên container (tùy chọn)
    ports:
      - "8080:80"                       # host:container
    volumes:
      - ./html:/usr/share/nginx/html    # Bind mount
      - nginx-logs:/var/log/nginx       # Named volume
    environment:
      - NGINX_HOST=hieucute.id.vn
    env_file:
      - .env                            # Đọc từ file .env
    depends_on:
      - app                             # Khởi động sau service "app"
    networks:
      - frontend
    restart: unless-stopped             # Tự restart khi crash

  app:
    build: .                            # Build từ Dockerfile ở thư mục hiện tại
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DB_HOST=db                      # Tên service = hostname trong network
    depends_on:
      - db
    networks:
      - frontend
      - backend

  db:
    image: mysql:8.0
    volumes:
      - db-data:/var/lib/mysql          # Persist database
    environment:
      MYSQL_ROOT_PASSWORD: ${DB_ROOT_PASSWORD}    # Từ .env
      MYSQL_DATABASE: myapp
      MYSQL_USER: appuser
      MYSQL_PASSWORD: ${DB_PASSWORD}
    networks:
      - backend
    restart: unless-stopped

# Khai báo volumes
volumes:
  db-data:
  nginx-logs:

# Khai báo networks
networks:
  frontend:
  backend:
```

**Restart policy:**

| Policy | Hành vi |
|--------|---------|
| `no` | Không tự restart (mặc định) |
| `always` | Luôn restart kể cả khi dừng thủ công |
| `on-failure` | Chỉ restart khi exit code khác 0 |
| `unless-stopped` | Restart trừ khi dừng thủ công — **khuyến nghị production** |

### 9.3 Các lệnh Docker Compose

```bash
# Khởi động tất cả service (detached)
docker compose up -d

# Khởi động và build lại image
docker compose up -d --build

# Chỉ khởi động một số service
docker compose up -d web db

# Dừng tất cả (giữ container)
docker compose stop

# Dừng và xóa container (giữ volume và image)
docker compose down

# Dừng và xóa cả volume
docker compose down -v

# Xem trạng thái
docker compose ps

# Xem log
docker compose logs
docker compose logs -f web              # Theo dõi log service web
docker compose logs --tail 100

# Vào trong container của service
docker compose exec web bash
docker compose exec db mysql -u root -p

# Scale service (chạy nhiều instance)
docker compose up -d --scale app=3

# Restart một service
docker compose restart web

# Build lại image
docker compose build
docker compose build --no-cache app     # Build không dùng cache

# Xem biến môi trường đã được resolve
docker compose config
```

---

## 10. Thực chiến – Deploy WordPress với Docker Compose

**Cấu trúc project:**
```
wordpress/
├── docker-compose.yml
└── .env
```

**File `.env`:**
```env
MYSQL_ROOT_PASSWORD=SuperSecret@2024
MYSQL_DATABASE=wordpress
MYSQL_USER=wpuser
MYSQL_PASSWORD=WpPass@2024
```

**File `docker-compose.yml`:**
```yaml
version: "3.9"

services:
  db:
    image: mysql:8.0
    container_name: wp-mysql
    volumes:
      - db_data:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
    networks:
      - wp-network
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 5

  wordpress:
    image: wordpress:6.5-php8.2-apache
    container_name: wp-app
    ports:
      - "8080:80"
    volumes:
      - wp_data:/var/www/html
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: ${MYSQL_USER}
      WORDPRESS_DB_PASSWORD: ${MYSQL_PASSWORD}
      WORDPRESS_DB_NAME: ${MYSQL_DATABASE}
    depends_on:
      db:
        condition: service_healthy     # Chờ MySQL sẵn sàng mới khởi động
    networks:
      - wp-network
    restart: unless-stopped

volumes:
  db_data:
  wp_data:

networks:
  wp-network:
```

**Triển khai:**
```bash
# Tạo thư mục và các file
mkdir wordpress && cd wordpress
# (tạo .env và docker-compose.yml như trên)

# Khởi động
docker compose up -d

# Kiểm tra
docker compose ps
docker compose logs -f

# Truy cập WordPress
# http://localhost:8080
```

**Dừng và dọn dẹp:**
```bash
# Dừng giữ data
docker compose down

# Dừng và xóa toàn bộ data (cẩn thận!)
docker compose down -v
```

---

## 11. Thực chiến – Deploy ứng dụng Node.js + MongoDB

**Cấu trúc project:**
```
node-app/
├── src/
│   └── index.js
├── Dockerfile
├── docker-compose.yml
├── package.json
└── .env
```

**File `src/index.js` (Express API đơn giản):**
```javascript
const express = require('express');
const mongoose = require('mongoose');

const app = express();
const PORT = process.env.PORT || 3000;
const MONGO_URI = process.env.MONGO_URI || 'mongodb://localhost:27017/myapp';

mongoose.connect(MONGO_URI)
  .then(() => console.log('MongoDB connected'))
  .catch(err => console.error('MongoDB error:', err));

app.get('/', (req, res) => {
  res.json({ status: 'OK', message: 'hieucute.id.vn API' });
});

app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
```

**File `Dockerfile`:**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY src ./src
EXPOSE 3000
USER node
CMD ["node", "src/index.js"]
```

**File `.env`:**
```env
MONGO_ROOT_PASSWORD=MongoSecret@2024
MONGO_DB=myapp
```

**File `docker-compose.yml`:**
```yaml
version: "3.9"

services:
  app:
    build: .
    container_name: node-api
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - PORT=3000
      - MONGO_URI=mongodb://admin:${MONGO_ROOT_PASSWORD}@mongo:27017/${MONGO_DB}?authSource=admin
    depends_on:
      mongo:
        condition: service_healthy
    networks:
      - app-network
    restart: unless-stopped

  mongo:
    image: mongo:7.0
    container_name: mongo-db
    volumes:
      - mongo_data:/data/db
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: ${MONGO_ROOT_PASSWORD}
      MONGO_INITDB_DATABASE: ${MONGO_DB}
    networks:
      - app-network
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "mongosh", "--eval", "db.adminCommand('ping')"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  mongo_data:

networks:
  app-network:
```

**Triển khai:**
```bash
# Build và khởi động
docker compose up -d --build

# Kiểm tra
docker compose ps
curl http://localhost:3000

# Xem log realtime
docker compose logs -f app
```

---

## 12. Bảo mật Docker cơ bản

### 12.1 Không chạy container với quyền root

```dockerfile
# Trong Dockerfile
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser
```

### 12.2 Dùng image nhỏ và đáng tin cậy

```dockerfile
# Ưu tiên Alpine (nhỏ ~5MB) thay vì Ubuntu (>200MB)
FROM node:18-alpine        # 
FROM node:18               # Nặng hơn không cần thiết
```

**Chỉ dùng image từ nguồn đáng tin:**
- Official images trên Docker Hub (có badge "Official Image").
- Verified publishers.
- Tránh image lạ, ít star, không có Dockerfile nguồn.

### 12.3 Không nhúng secret vào image

```dockerfile
#  Sai — secret bị lưu trong image layer
ENV DB_PASSWORD=secret123

#  Đúng — truyền qua biến môi trường khi chạy
docker run -e DB_PASSWORD=secret123 myapp
# Hoặc dùng .env file với docker compose
```

### 12.4 Giới hạn tài nguyên container

```bash
docker run -d \
  --memory="256m" \
  --memory-swap="256m" \
  --cpus="0.5" \
  --pids-limit 100 \
  nginx
```

### 12.5 Scan vulnerability

```bash
# Dùng Docker Scout (tích hợp sẵn)
docker scout cves nginx:latest
docker scout quickview nginx:latest
```

### 12.6 Cập nhật Docker thường xuyên

```bash
sudo apt update && sudo apt upgrade docker-ce docker-ce-cli containerd.io
```

---

## 13. Lỗi thường gặp và cách xử lý

| Lỗi | Nguyên nhân | Cách xử lý |
|-----|------------|------------|
| `permission denied while trying to connect to the Docker daemon` | User chưa trong group docker | `sudo usermod -aG docker $USER && newgrp docker` |
| `port is already allocated` | Port trên host đã bị dùng | Đổi host port: `-p 8081:80` hoặc dừng process đang chiếm port |
| `No space left on device` | Disk đầy do image/container cũ | `docker system prune -a` |
| `Error response from daemon: OCI runtime` | Lỗi cấu hình container | Xem log: `docker logs container_name` |
| Container khởi động rồi dừng ngay | CMD bị lỗi hoặc app crash | `docker logs container_name`, kiểm tra CMD trong Dockerfile |
| `name is already in use` | Container cùng tên đã tồn tại | `docker rm old_container_name` rồi tạo lại |
| Build image chậm | Không tận dụng layer cache | Sắp xếp lại Dockerfile: COPY file ít thay đổi trước |
| `exec format error` | Build image trên chip khác (M1 vs AMD64) | Thêm `--platform linux/amd64` khi build |
| Compose service không kết nối được nhau | Service dùng tên sai hoặc khác network | Dùng tên service làm hostname, đảm bảo cùng network |

**Dọn dẹp toàn bộ (cẩn thận!):**
```bash
# Xóa container, network, image không dùng
docker system prune

# Xóa luôn cả volume (MẤT DATA)
docker system prune -a --volumes

# Xem Docker đang dùng bao nhiêu disk
docker system df
```

---

## 14. Bảng tóm tắt lệnh quan trọng

### Image

| Lệnh | Mô tả |
|------|-------|
| `docker pull <image>` | Tải image |
| `docker images` | Liệt kê image |
| `docker build -t <name>:<tag> .` | Build image từ Dockerfile |
| `docker rmi <image>` | Xóa image |
| `docker image prune -a` | Xóa image không dùng |
| `docker push <image>` | Push image lên registry |

### Container

| Lệnh | Mô tả |
|------|-------|
| `docker run -d -p 8080:80 --name c1 nginx` | Tạo và chạy container |
| `docker ps` / `docker ps -a` | Liệt kê container |
| `docker stop c1` | Dừng container |
| `docker start c1` | Khởi động container |
| `docker rm c1` | Xóa container |
| `docker exec -it c1 bash` | Vào terminal container |
| `docker logs -f c1` | Xem log realtime |
| `docker stats` | Giám sát tài nguyên |
| `docker inspect c1` | Xem thông tin chi tiết |

### Volume & Network

| Lệnh | Mô tả |
|------|-------|
| `docker volume create myvol` | Tạo volume |
| `docker volume ls` | Liệt kê volume |
| `docker volume prune` | Xóa volume không dùng |
| `docker network create mynet` | Tạo network |
| `docker network ls` | Liệt kê network |
| `docker network inspect mynet` | Xem thông tin network |

### Docker Compose

| Lệnh | Mô tả |
|------|-------|
| `docker compose up -d` | Khởi động tất cả service |
| `docker compose up -d --build` | Khởi động và build lại |
| `docker compose down` | Dừng và xóa container |
| `docker compose down -v` | Dừng và xóa cả volume |
| `docker compose ps` | Xem trạng thái service |
| `docker compose logs -f` | Xem log realtime |
| `docker compose exec web bash` | Vào terminal service |
| `docker compose restart web` | Restart một service |

### Dọn dẹp

| Lệnh | Mô tả |
|------|-------|
| `docker system df` | Xem disk usage |
| `docker system prune` | Dọn container/network/image không dùng |
| `docker system prune -a --volumes` | Dọn tất cả (kể cả volume) |

---

## References

1. [Cài đặt và sử dụng Docker trên Ubuntu — CyStack](https://cystack.net/tutorial/cai-dat-va-su-dung-docker-tren-ubuntu)
2. [Docker: Chưa biết gì đến biết dùng (Phần 1) — Viblo](https://viblo.asia/p/docker-chua-biet-gi-den-biet-dung-phan-1-lich-su-ByEZkWrEZQ0)
3. [Docker là gì? Tổng quan về Docker — Máy Chủ Vina](https://www.maychuvina.com/docker-la-gi/)
4. [Docker Official Documentation — docs.docker.com](https://docs.docker.com/)
5. [Docker Hub — hub.docker.com](https://hub.docker.com/)
6. [Docker Compose File Reference — docs.docker.com](https://docs.docker.com/compose/compose-file/)
7. [Dockerfile Best Practices — docs.docker.com](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
