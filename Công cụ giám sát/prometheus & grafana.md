#  Hệ Thống Giám Sát Prometheus · Grafana · Alertmanager


---

## Mục Lục

1. [Prometheus, Grafana, Alertmanager Dùng Để Làm Gì?](#1-prometheus-grafana-alertmanager-dùng-để-làm-gì)
2. [Cài Đặt node_exporter — trên mọi server cần giám sát](#2-cài-đặt-node_exporter--trên-mọi-server-cần-giám-sát)
3. [Cài Đặt mysqld_exporter — MySQL/MariaDB](#3-cài-đặt-mysqld_exporter--mysqlmariadb)
4. [Cài Đặt redis_exporter — Redis](#4-cài-đặt-redis_exporter--redis)
5. [Cấu Hình HAProxy Expose Metrics](#5-cấu-hình-haproxy-expose-metrics)
6. [Cài Đặt Prometheus](#6-cài-đặt-prometheus)
7. [Alert Rules](#7-alert-rules)
8. [Cài Đặt Alertmanager](#8-cài-đặt-alertmanager)
9. [Cấu Hình Gửi Cảnh Báo qua Gmail](#9-cấu-hình-gửi-cảnh-báo-qua-gmail)
10. [Cấu Hình Gửi Cảnh Báo qua Telegram](#10-cấu-hình-gửi-cảnh-báo-qua-telegram)
11. [Cài Đặt Grafana](#11-cài-đặt-grafana)
12. [Kết Nối Grafana ↔ Prometheus & Import Dashboard](#12-kết-nối-grafana--prometheus--import-dashboard)
13. [PromQL Tham Khảo Nhanh](#13-promql-tham-khảo-nhanh)
14. [Kiểm Tra Toàn Hệ Thống](#14-kiểm-tra-toàn-hệ-thống)

---

## 1. Prometheus, Grafana, Alertmanager Dùng Để Làm Gì?

### 1.1 Prometheus là gì?

Prometheus là một bộ công cụ giám sát và cảnh báo mã nguồn mở, hoạt động theo mô hình HTTP "pull" để thu thập các chỉ số dạng số từ hạ tầng, lưu trữ chúng trong một cơ sở dữ liệu chuỗi thời gian, và đánh giá cảnh báo theo thời gian thực. Nó dùng mô hình dữ liệu đa chiều, lưu metric dưới dạng chuỗi thời gian được định danh bằng tên metric và các cặp key-value gọi là label.

**Nguyên lý hoạt động:**
- **Mô hình pull**: Prometheus server định kỳ truy vấn các HTTP endpoint đã chỉ định để thu thập metric, thay vì để ứng dụng tự đẩy dữ liệu về server.
- **Exporter**: các service nhỏ gọn có nhiệm vụ định dạng và expose dữ liệu cho những ứng dụng/phần cứng vốn không tự sinh ra metric theo chuẩn Prometheus.
- **Service discovery**: Prometheus có thể tích hợp với Kubernetes, các nhà cung cấp cloud và file tĩnh để tự động phát hiện endpoint cần giám sát; với hạ tầng đơn giản có thể khai báo target tĩnh trong `prometheus.yml`.

**Các loại metric cốt lõi**: Counter (chỉ tăng, ví dụ tổng số HTTP request), Gauge (có thể tăng/giảm, ví dụ RAM đang dùng hoặc nhiệt độ CPU), Histogram (lấy mẫu và đếm vào các bucket có thể cấu hình, ví dụ thời gian phản hồi request), và Summary (tương tự histogram nhưng tính toán các quantile có thể cấu hình trên một cửa sổ thời gian trượt).

**Prometheus có thể giám sát gì?** Thông thường là metric của các service chạy liên tục qua HTTP endpoint, metric hệ điều hành/host (CPU, RAM, disk đầy...) thông qua exporter cài trên host, uptime của website, và cả cronjob.

### 1.2 Grafana là gì?

Prometheus tự có giao diện web cơ bản để xem dữ liệu dạng bảng/đồ thị, nhưng để dựng dashboard trực quan, đẹp và linh hoạt hơn thì cần công cụ chuyên biệt. Grafana là công cụ đồng hành tiêu chuẩn của Prometheus: Grafana kết nối tới Prometheus như một nguồn dữ liệu  để dựng các dashboard và biểu đồ trực quan, phong phú. Grafana không tự thu thập metric — nó chỉ truy vấn dữ liệu đã có sẵn trong Prometheus qua PromQL rồi hiển thị.

### 1.3 Alertmanager là gì?

Prometheus tập trung vào việc thu thập và đánh giá metric, còn việc cảnh báo và trực quan hóa được giao cho các công cụ đồng hành chuyên biệt: Alertmanager đảm nhận việc khử trùng lặp, gom nhóm và định tuyến cảnh báo tới các kênh thông báo như Slack, PagerDuty hay email.

Alertmanager là một thành phần quan trọng trong bộ công cụ giám sát Prometheus, giúp quản lý các cảnh báo được Prometheus tạo ra, với 3 chức năng chính:
- **Nhóm cảnh báo**: gom các cảnh báo tương tự lại để giảm số lượng thông báo gửi đi, tránh spam.
- **Xử lý cảnh báo**: nhận alert từ Prometheus và xử lý theo rule đã định (gửi khi FIRING, tắt/RESOLVED khi sự cố đã khắc phục).
- **Gửi thông báo**: hỗ trợ gửi tới nhiều kênh khác nhau như email, Slack, PagerDuty, hoặc qua webhook — trong tài liệu này sẽ dùng **Gmail (SMTP)** và **Telegram**.

### 1.4 Vì sao 3 công cụ này thường đi cùng nhau?

```
[node_exporter]───┐
[mysqld_exporter]──┼──► Prometheus (scrape metrics, đánh giá alert rules)
[redis_exporter]───┤          │
[haproxy /metrics]─┘          ▼
                        Alertmanager ──► Gmail (SMTP)
                                     └──► Telegram Bot
                        Grafana ◄── query Prometheus (dashboard trực quan)
```

Prometheus đóng vai trò trung tâm thu thập + đánh giá điều kiện cảnh báo; Alertmanager phụ trách "cảnh báo ai, khi nào, qua kênh nào"; Grafana phụ trách "nhìn dữ liệu trực quan ra sao". Tách 3 vai trò này giúp mỗi thành phần làm tốt một việc và có thể mở rộng độc lập — có thể gộp chung 1 máy (all-in-one, phù hợp lab/demo) hoặc tách riêng máy Monitoring và các máy target (khuyến nghị cho production).

### Bảng phiên bản tham khảo

| Phần mềm | Phiên bản gợi ý |
|---|---|
| Prometheus | 2.52.x |
| Alertmanager | 0.27.x |
| Grafana | 11.x  |
| node_exporter | 1.7.0 |
| mysqld_exporter | 0.15.1 |
| redis_exporter | 1.62.0 |

> **Chuẩn bị chung**: Ubuntu Server 22.04 LTS đã `apt update && apt upgrade`, có quyền `sudo`, đã xác định IP các server tham gia, và mở firewall các port: `9090` (Prometheus), `9093` (Alertmanager), `3000` (Grafana), `9100` (node_exporter), `9104` (mysqld_exporter), `9121` (redis_exporter), `8404` (HAProxy stats).

---

## 2. Cài Đặt node_exporter — trên mọi server cần giám sát

### 2.1 node_exporter

Để giám sát hệ điều hành — phát hiện lúc nào ổ cứng của server đầy hay server chạy CPU 100% liên tục — cần cài một exporter chuyên biệt trên host để thu thập thông tin hệ điều hành và publish ra một địa chỉ HTTP mà Prometheus có thể truy cập. Đó chính là **node_exporter** — exporter chính thức của dự án Prometheus dùng để expose các metric cấp hệ điều hành Linux (CPU, RAM, disk, network, load...) qua endpoint `/metrics` ở port mặc định `9100`. Đây là exporter nền tảng, gần như bắt buộc phải có trên mọi server muốn giám sát.

### 2.2 Cài đặt

```bash
NODE_VER="1.7.0"
cd /tmp
sudo wget -q https://github.com/prometheus/node_exporter/releases/download/v${NODE_VER}/node_exporter-${NODE_VER}.linux-amd64.tar.gz
sudo tar xzf node_exporter-${NODE_VER}.linux-amd64.tar.gz
sudo cp node_exporter-${NODE_VER}.linux-amd64/node_exporter /usr/local/bin/
sudo useradd -rs /bin/false node_exporter 2>/dev/null || true

sudo tee /etc/systemd/system/node_exporter.service << 'EOF'
[Unit]
Description=Prometheus Node Exporter
After=network.target

[Service]
User=node_exporter
ExecStart=/usr/local/bin/node_exporter --web.listen-address=:9100
Restart=always

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable --now node_exporter

# Kiểm tra
curl -s http://localhost:9100/metrics | head
```

---

## 3. Cài Đặt mysqld_exporter — MySQL/MariaDB

### 3.1 mysqld_exporter

MySQL/MariaDB không tự expose metric theo định dạng Prometheus, nên cần một exporter riêng đứng giữa để đọc các biến trạng thái nội bộ của MySQL (qua `SHOW GLOBAL STATUS`, `SHOW GLOBAL VARIABLES`...) rồi chuyển thành metric mà Prometheus scrape được. **mysqld_exporter** là exporter chính thức cho MySQL/MariaDB, expose ở port `9104`, cho biết số kết nối hiện tại, số query/giây, trạng thái Galera cluster (nếu có), slow query...

### 3.2 Cài đặt

```bash
# Tạo user riêng cho exporter, chỉ cấp quyền đọc trạng thái
sudo mysql -u root -p -e "
CREATE USER 'exporter'@'127.0.0.1' IDENTIFIED BY 'Exporter@2026!' WITH MAX_USER_CONNECTIONS 3;
GRANT PROCESS, REPLICATION CLIENT, SELECT ON *.* TO 'exporter'@'127.0.0.1';
FLUSH PRIVILEGES;"

sudo bash -c 'printf "[client]\nuser=exporter\npassword=Exporter@2026!\nhost=127.0.0.1\n" > /etc/.mysqld_exporter.cnf'
sudo chmod 600 /etc/.mysqld_exporter.cnf

cd /tmp
sudo wget -q https://github.com/prometheus/mysqld_exporter/releases/download/v0.15.1/mysqld_exporter-0.15.1.linux-amd64.tar.gz
sudo tar xzf mysqld_exporter-0.15.1.linux-amd64.tar.gz
sudo cp mysqld_exporter-0.15.1.linux-amd64/mysqld_exporter /usr/local/bin/

sudo tee /etc/systemd/system/mysqld_exporter.service << 'EOF'
[Unit]
Description=MySQL Exporter
After=network.target

[Service]
ExecStart=/usr/local/bin/mysqld_exporter \
  --config.my-cnf=/etc/.mysqld_exporter.cnf \
  --web.listen-address=:9104
Restart=always

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable --now mysqld_exporter

curl -s http://localhost:9104/metrics | head
```

---

## 4. Cài Đặt redis_exporter — Redis

### 4.1 redis_exporter

Tương tự MySQL, Redis không có sẵn endpoint `/metrics` theo chuẩn Prometheus. **redis_exporter** kết nối tới Redis qua giao thức Redis thông thường, có xác thực bằng password rồi expose các metric như memory used, hit rate, số lệnh/giây, trạng thái master/slave... ra port `9121` để Prometheus scrape.

### 4.2 Cài đặt

```bash
cd /tmp
sudo wget -q https://github.com/oliver006/redis_exporter/releases/download/v1.62.0/redis_exporter-v1.62.0.linux-amd64.tar.gz
sudo tar xzf redis_exporter-v1.62.0.linux-amd64.tar.gz
sudo cp redis_exporter-v1.62.0.linux-amd64/redis_exporter /usr/local/bin/

sudo tee /etc/systemd/system/redis_exporter.service << 'EOF'
[Unit]
Description=Redis Exporter
After=network.target

[Service]
ExecStart=/usr/local/bin/redis_exporter \
  --redis.addr=redis://127.0.0.1:6379 \
  --redis.password=YOUR_REDIS_PASSWORD
Restart=always

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable --now redis_exporter

curl -s http://localhost:9121/metrics | head
```

---

## 5. Cấu Hình HAProxy Expose Metrics

### 5.1 HAProxy metrics

Khác với MySQL/Redis, HAProxy từ bản 2.x trở lên đã tích hợp sẵn khả năng expose metric theo chuẩn Prometheus ngay trong module thống kê (`stats`) của nó, nên **không cần cài exporter riêng** — chỉ cần bật một dòng cấu hình để route request tới `/metrics` sang service Prometheus nội bộ của HAProxy. Metric thu được cho biết trạng thái UP/DOWN của từng backend, số request, số session đang xử lý...

### 5.2 Cấu hình

HAProxy (bản 2.x trên Ubuntu 22.04) hỗ trợ endpoint `/metrics` dạng Prometheus ngay trong `listen stats`, không cần exporter riêng:

```haproxy
listen stats
    bind *:8404
    mode http
    stats enable
    stats uri /stats
    stats refresh 10s
    http-request use-service prometheus-exporter if { path /metrics }
```

```bash
sudo systemctl reload haproxy
curl -s http://localhost:8404/metrics | head
```

> Nếu không có HAProxy trong hạ tầng của bạn, bỏ qua bước này.

---

## 6. Cài Đặt Prometheus

### 6.1 cấu trúc cài đặt

Prometheus cần một HTTP endpoint để scrape metric — sau khi các exporter ở mục 2-5 đã sẵn sàng, Prometheus có thể bắt đầu kéo dữ liệu số, ghi nhận thành chuỗi thời gian và lưu vào cơ sở dữ liệu cục bộ phù hợp cho dữ liệu chuỗi thời gian (có thể tích hợp thêm remote storage nếu cần lưu dài hạn). Người dùng truy vấn dữ liệu bằng **PromQL** — ngôn ngữ truy vấn riêng cho phép chọn lọc và tổng hợp dữ liệu chuỗi thời gian theo thời gian thực, đồng thời PromQL cũng là nền tảng để thiết lập điều kiện cảnh báo (alert rules) ở mục 7.

Về vận hành, việc cài đặt Prometheus gồm 3 phần: file thực thi (`prometheus`, `promtool`), file cấu hình `prometheus.yml` (khai báo scrape targets — tức các exporter đã cài), và service systemd để chạy nền liên tục.

### 6.2 Cài đặt

Thực hiện trên máy được chọn làm **Monitoring server**.

```bash
PROM_VER="2.52.0"
cd /tmp
sudo wget -q https://github.com/prometheus/prometheus/releases/download/v${PROM_VER}/prometheus-${PROM_VER}.linux-amd64.tar.gz
sudo tar xzf prometheus-${PROM_VER}.linux-amd64.tar.gz
sudo cp prometheus-${PROM_VER}.linux-amd64/{prometheus,promtool} /usr/local/bin/
sudo mkdir -p /etc/prometheus /var/lib/prometheus
sudo useradd -rs /bin/false prometheus 2>/dev/null || true
sudo cp -r prometheus-${PROM_VER}.linux-amd64/consoles /etc/prometheus/
sudo cp -r prometheus-${PROM_VER}.linux-amd64/console_libraries /etc/prometheus/
sudo chown -R prometheus:prometheus /etc/prometheus /var/lib/prometheus
```

**`/etc/prometheus/prometheus.yml`** — thay IP bằng IP thật của từng target trong hạ tầng bạn:

```yaml
global:
  scrape_interval:     15s
  evaluation_interval: 15s

alerting:
  alertmanagers:
    - static_configs:
        - targets: ['localhost:9093']

rule_files:
  - "/etc/prometheus/alert_rules.yml"

scrape_configs:
  - job_name: 'node_exporter'
    static_configs:
      - targets:
          - '<IP_SERVER_1>:9100'
          - '<IP_SERVER_2>:9100'

  - job_name: 'mysql'
    static_configs:
      - targets:
          - '<IP_DB_SERVER>:9104'

  - job_name: 'redis'
    static_configs:
      - targets:
          - '<IP_REDIS_SERVER>:9121'

  - job_name: 'haproxy'
    metrics_path: /metrics
    static_configs:
      - targets:
          - '<IP_LB_SERVER>:8404'
```

**Systemd service `/etc/systemd/system/prometheus.service`:**

```ini
[Unit]
Description=Prometheus Monitoring
After=network.target

[Service]
User=prometheus
Group=prometheus
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus \
  --web.listen-address=:9090
Restart=always

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now prometheus

# Kiểm tra
curl -s http://localhost:9090/-/healthy
# Truy cập UI: http://<IP_MONITORING>:9090
```

---

## 7. Alert Rules

### 7.1 alert rules

Alert rule là các điều kiện viết bằng PromQL mà Prometheus tự đánh giá định kỳ (theo `evaluation_interval`); khi điều kiện đúng liên tục đủ thời gian `for`, alert chuyển sang trạng thái **FIRING** và được đẩy sang Alertmanager để xử lý gửi thông báo. Việc đặt ngưỡng cảnh báo hợp lý — không quá nhạy gây "báo động giả" liên tục, cũng không quá chậm khiến sự cố ảnh hưởng người dùng trước khi được phát hiện — là yếu tố quan trọng để có một chiến lược cảnh báo hiệu quả, tránh gây mệt mỏi cảnh báo (alert fatigue) cho đội vận hành.

`inhibit_rules` ở cuối file dùng để **ngăn cảnh báo thừa**: ví dụ khi cả server đã DOWN (`NodeDown`), không cần bắn thêm cảnh báo CPU cao/RAM thấp/Disk đầy của chính server đó nữa.

### 7.2 Cấu hình

**`/etc/prometheus/alert_rules.yml`:**

```yaml
groups:
  - name: node_alerts
    rules:
      - alert: NodeDown
        expr: up{job="node_exporter"} == 0
        for: 30s
        labels:
          severity: critical
        annotations:
          summary: 'NODE DOWN: {{ $labels.instance }}'
          description: 'Node {{ $labels.instance }} mat ket noi hon 30 giay.'

      - alert: HighCPU
        expr: 100 - (avg by(instance)(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 85
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: 'CPU CAO: {{ $labels.instance }}'
          description: 'CPU vuot 85%. Hien tai: {{ $value | printf "%.1f" }}%'

      - alert: HighMemory
        expr: (1 - node_memory_MemAvailable_bytes/node_memory_MemTotal_bytes) * 100 > 90
        for: 2m
        labels:
          severity: warning
        annotations:
          summary: 'RAM THAP: {{ $labels.instance }}'
          description: 'RAM con trong duoi 10%.'

      - alert: DiskFull
        expr: |
          (1 - node_filesystem_avail_bytes{
            job="node_exporter",
            fstype!~"tmpfs|overlay|squashfs|devtmpfs",
            mountpoint!~"/boot/efi|/run/.*"
          } / node_filesystem_size_bytes{
            job="node_exporter",
            fstype!~"tmpfs|overlay|squashfs|devtmpfs",
            mountpoint!~"/boot/efi|/run/.*"
          }) * 100 > 80
        for: 2m
        labels:
          severity: warning
        annotations:
          summary: 'DISK DAY: {{ $labels.instance }}'
          description: 'Disk {{ $labels.mountpoint }} su dung {{ $value | printf "%.1f" }}%.'

  - name: haproxy_alerts
    rules:
      - alert: HAProxyBackendDown
        expr: haproxy_backend_up{job="haproxy"} == 0
        for: 10s
        labels:
          severity: critical
        annotations:
          summary: 'HAPROXY BACKEND DOWN: {{ $labels.backend }}'
          description: 'Backend {{ $labels.backend }} bi DOWN tren HAProxy.'

  - name: mysql_alerts
    rules:
      - alert: MySQLDown
        expr: mysql_up == 0
        for: 30s
        labels:
          severity: critical
        annotations:
          summary: 'MYSQL DOWN: {{ $labels.instance }}'
          description: 'MySQL/MariaDB tren {{ $labels.instance }} khong the ket noi.'

      - alert: MySQLTooManyConnections
        expr: mysql_global_status_threads_connected > 100
        for: 2m
        labels:
          severity: warning
        annotations:
          summary: 'MYSQL QUA NHIEU KET NOI: {{ $labels.instance }}'
          description: 'So luong connection hien tai: {{ $value }}'

  - name: redis_alerts
    rules:
      - alert: RedisDown
        expr: redis_up == 0
        for: 30s
        labels:
          severity: critical
        annotations:
          summary: 'REDIS DOWN: {{ $labels.instance }}'
          description: 'Redis tren {{ $labels.instance }} khong the ket noi.'

inhibit_rules:
  - source_match:
      alertname: NodeDown
    target_match_re:
      alertname: 'HighCPU|HighMemory|DiskFull'
    equal: ['instance']
```

```bash
# Kiểm tra cú pháp trước khi reload
promtool check rules /etc/prometheus/alert_rules.yml
sudo systemctl reload prometheus
```

Xem alert đang chạy tại: `http://<IP_MONITORING>:9090/alerts`

---

## 8. Cài Đặt Alertmanager

### 8.1 cơ chế hoạt động

Alertmanager nhận alert (dạng JSON) do Prometheus gửi tới qua cấu hình `alerting.alertmanagers` trong `prometheus.yml`. Bên trong `alertmanager.yml` có 2 khối chính:
- **`route`**: cây định tuyến — quyết định alert nào đi tới `receiver` nào, gom nhóm theo label gì (`group_by`), đợi bao lâu trước khi gửi lần đầu (`group_wait`), khoảng cách giữa các lần gửi nếu có alert mới trong cùng nhóm (`group_interval`), và tần suất gửi lại nếu alert vẫn còn FIRING (`repeat_interval`).
- **`receivers`**: danh sách đích đến thực tế — có thể là email, Telegram, Slack, webhook... mỗi receiver có thể cấu hình 1 hoặc nhiều kênh cùng lúc.

`resolve_timeout` xác định thời gian tối đa Alertmanager chờ trước khi tự coi một cảnh báo là đã hết, nếu không thấy Prometheus gửi lại tín hiệu FIRING.

### 8.2 Cài đặt

```bash
ALERTM_VER="0.27.0"
cd /tmp
sudo wget -q https://github.com/prometheus/alertmanager/releases/download/v${ALERTM_VER}/alertmanager-${ALERTM_VER}.linux-amd64.tar.gz
sudo tar xzf alertmanager-${ALERTM_VER}.linux-amd64.tar.gz
sudo cp alertmanager-${ALERTM_VER}.linux-amd64/{alertmanager,amtool} /usr/local/bin/
sudo mkdir -p /etc/alertmanager /var/lib/alertmanager
sudo useradd -rs /bin/false alertmanager 2>/dev/null || true
sudo chown -R alertmanager:alertmanager /etc/alertmanager /var/lib/alertmanager
```

**Systemd service `/etc/systemd/system/alertmanager.service`:**

```ini
[Unit]
Description=Prometheus Alertmanager
After=network.target

[Service]
User=alertmanager
ExecStart=/usr/local/bin/alertmanager \
  --config.file=/etc/alertmanager/alertmanager.yml \
  --storage.path=/var/lib/alertmanager \
  --web.listen-address=:9093
Restart=always

[Install]
WantedBy=multi-user.target
```

Cấu hình chi tiết `alertmanager.yml` với Gmail và Telegram ở 2 mục kế tiếp — sau khi hoàn tất, chạy:

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now alertmanager
curl -s http://localhost:9093/-/healthy
```

---

## 9. Cấu Hình Gửi Cảnh Báo qua Gmail

### 9.1 Lý thuyết

Alertmanager gửi mail qua giao thức SMTP thông thường — không phải API riêng của Gmail — nên chỉ cần khai báo `smtp_smarthost`, tài khoản gửi và mật khẩu trong khối `global`. Vì Gmail đã tắt xác thực bằng mật khẩu thường cho ứng dụng bên thứ ba, cần dùng **App Password** (mật khẩu ứng dụng 16 ký tự) sinh riêng cho việc này thay vì mật khẩu đăng nhập Gmail thông thường.

> Cần bật **2-Step Verification** cho tài khoản Gmail và tạo **App Password** (16 ký tự) tại: Google Account → Security → App passwords. Không dùng mật khẩu Gmail thông thường.

**`/etc/alertmanager/alertmanager.yml`** (phần Gmail):

```yaml
global:
  smtp_smarthost:     'smtp.gmail.com:587'
  smtp_from:          'your.email@gmail.com'
  smtp_auth_username: 'your.email@gmail.com'
  smtp_auth_password: 'app-password-16-chars'   # Gmail App Password
  smtp_require_tls:   true
  resolve_timeout:    1m

route:
  group_by:        ['alertname', 'severity', 'instance']
  group_wait:      10s
  group_interval:  1m
  repeat_interval: 2h
  receiver: 'gmail'
  routes:
    - match:
        severity: critical
      receiver: 'gmail_critical'
      group_wait:      5s
      repeat_interval: 30m

receivers:
  - name: 'gmail'
    email_configs:
      - to: 'your.email@gmail.com'
        send_resolved: true
        headers:
          Subject: >-
            {{ if eq .Status "firing" }}WARNING FIRING{{ else }}RESOLVED{{ end }}
            [{{ .GroupLabels.alertname }}]

  - name: 'gmail_critical'
    email_configs:
      - to: 'your.email@gmail.com'
        send_resolved: true
        headers:
          Subject: >-
            {{ if eq .Status "firing" }}CRITICAL FIRING{{ else }}CRITICAL RESOLVED{{ end }}
            [{{ .GroupLabels.alertname }}]
```

---

## 10. Cấu Hình Gửi Cảnh Báo qua Telegram

### 10.1 Lý thuyết

Alertmanager có receiver `telegram_configs` hỗ trợ sẵn, gửi tin nhắn thông qua một **Telegram Bot** — một tài khoản bot do chính người dùng tạo qua BotFather, có `bot_token` riêng để xác thực khi gọi Telegram Bot API. Cảnh báo được gửi vào `chat_id` chỉ định (có thể là chat riêng hoặc group), giúp đội vận hành nhận thông báo tức thời trên điện thoại mà không phụ thuộc vào việc kiểm tra email.

**Bước 1 — Tạo Bot Telegram:**
1. Mở Telegram, chat với **@BotFather** → gõ `/newbot` → đặt tên bot → nhận **Bot Token** (dạng `123456789:ABCdefGhIJKlmNoPQRsTUVwxyZ`).
2. Tạo 1 group hoặc dùng chat riêng, thêm bot vào group.
3. Lấy **Chat ID**: gửi 1 tin nhắn bất kỳ vào group, sau đó truy cập:
   ```
   https://api.telegram.org/bot<BOT_TOKEN>/getUpdates
   ```
   Tìm giá trị `"chat":{"id": -100xxxxxxxxxx}` — đó là Chat ID (số âm nếu là group).

**Bước 2 — Thêm receiver Telegram vào `alertmanager.yml`** (kết hợp cùng Gmail ở trên):

```yaml
route:
  group_by:        ['alertname', 'severity', 'instance']
  group_wait:      10s
  group_interval:  1m
  repeat_interval: 2h
  receiver: 'gmail'                # receiver mặc định
  routes:
    - match:
        severity: critical
      receiver: 'all_channels'      # critical -> gửi cả Gmail lẫn Telegram
      group_wait:      5s
      repeat_interval: 30m

receivers:
  - name: 'gmail'
    email_configs:
      - to: 'your.email@gmail.com'
        send_resolved: true

  - name: 'telegram'
    telegram_configs:
      - bot_token: '123456789:ABCdefGhIJKlmNoPQRsTUVwxyZ'
        chat_id: -1001234567890
        parse_mode: 'HTML'
        message: >-
          {{ if eq .Status "firing" }}🚨 <b>FIRING</b>{{ else }}✅ <b>RESOLVED</b>{{ end }}
          [{{ .GroupLabels.alertname }}]
          Instance: {{ .CommonLabels.instance }}

  - name: 'all_channels'
    email_configs:
      - to: 'your.email@gmail.com'
        send_resolved: true
    telegram_configs:
      - bot_token: '123456789:ABCdefGhIJKlmNoPQRsTUVwxyZ'
        chat_id: -1001234567890
        parse_mode: 'HTML'
        message: >-
          {{ if eq .Status "firing" }}🚨 <b>CRITICAL FIRING</b>{{ else }}✅ <b>RESOLVED</b>{{ end }}
          [{{ .GroupLabels.alertname }}]
          Instance: {{ .CommonLabels.instance }}
```

```bash
# Kiểm tra cú pháp
amtool check-config /etc/alertmanager/alertmanager.yml
sudo systemctl restart alertmanager
```

> Lưu ý: `bot_token` và `chat_id` là thông tin nhạy cảm — nên đặt file config quyền `600` và cân nhắc dùng biến môi trường/secret manager trong môi trường production thay vì hardcode.

---

## 11. Cài Đặt Grafana

### 11.1 Lý thuyết

Grafana không lưu metric — nó chỉ đóng vai trò lớp trực quan hóa, kết nối tới Prometheus qua HTTP API và dùng chính PromQL để truy vấn dữ liệu hiển thị lên dashboard. Grafana cũng hỗ trợ cộng đồng chia sẻ dashboard có sẵn (dạng JSON, định danh bằng một ID số) — thay vì tự vẽ từ đầu, có thể import các dashboard phổ biến cho node_exporter, MySQL, Redis, HAProxy... rồi tùy chỉnh thêm.

### 11.2 Cài đặt

```bash
sudo apt install -y apt-transport-https software-properties-common wget
wget -q -O - https://packages.grafana.com/gpg.key \
  | gpg --dearmor | sudo tee /usr/share/keyrings/grafana.gpg > /dev/null
echo "deb [signed-by=/usr/share/keyrings/grafana.gpg] https://packages.grafana.com/oss/deb stable main" \
  | sudo tee /etc/apt/sources.list.d/grafana.list

sudo apt update && sudo apt install -y grafana
sudo systemctl enable --now grafana-server

# Kiểm tra
sudo systemctl status grafana-server
```

Truy cập: `http://<IP_MONITORING>:3000` — đăng nhập mặc định `admin` / `admin`, đổi mật khẩu ngay lần đầu.

---

## 12. Kết Nối Grafana ↔ Prometheus & Import Dashboard

1. Grafana → **Connections → Data Sources → Add data source → Prometheus**
2. URL: `http://localhost:9090` (nếu Grafana và Prometheus cùng máy) hoặc `http://<IP_PROMETHEUS>:9090`
3. **Save & Test**

**Import dashboard có sẵn** (Dashboards → Import → nhập ID):

| Dashboard ID | Tên | Nội dung |
|---|---|---|
| `1860` | Node Exporter Full | CPU · RAM · Disk · Network |
| `367` | HAProxy 2 Full | Request rate · Backend UP/DOWN · Sessions |
| `763` | Redis Dashboard | Memory · Hit rate · Commands/s |
| `7362` | MySQL Overview | Queries · Connections · InnoDB |

---

## 13. PromQL Tham Khảo Nhanh

| Thông số | Câu lệnh PromQL |
|---|---|
| CPU Usage (%) | `100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)` |
| RAM Usage (%) | `(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100` |
| CPU Load (1m) | `node_load1` |
| Server Uptime | `node_time_seconds - node_boot_time_seconds` |
| Disk Usage (%) | `100 - ((node_filesystem_avail_bytes{mountpoint="/"}) * 100 / node_filesystem_size_bytes{mountpoint="/"})` |
| Network In | `rate(node_network_receive_bytes_total[5m])` |
| Network Out | `rate(node_network_transmit_bytes_total[5m])` |
| MySQL Status | `mysql_up` |
| MySQL Connections | `mysql_global_status_threads_connected` |
| MySQL Queries/s | `rate(mysql_global_status_questions[1m])` |
| Redis Status | `redis_up` |
| Redis Memory Used | `redis_memory_used_bytes` |
| HAProxy Backend Status | `haproxy_backend_up` |

---

## 14. Kiểm Tra Toàn Hệ Thống

```bash
echo "===== MONITORING STACK CHECK ====="

echo "[1] Services"
for svc in prometheus alertmanager grafana-server node_exporter; do
  STATUS=$(systemctl is-active $svc 2>/dev/null || echo "not-found")
  printf "  %-20s %s\n" "$svc" "$STATUS"
done

echo "[2] Prometheus targets"
curl -s http://localhost:9090/api/v1/targets | grep -o '"health":"[a-z]*"' | sort | uniq -c

echo "[3] Alertmanager health"
curl -s http://localhost:9093/-/healthy

echo "[4] Grafana health"
curl -s http://localhost:3000/api/health
```

**Test gửi cảnh báo thử:**

```bash
# Trên máy đang có node_exporter — tắt tạm để trigger NodeDown
sudo systemctl stop node_exporter

# Theo dõi: http://<IP_MONITORING>:9090/alerts
# T+30s -> PENDING, T+60s -> FIRING -> Alertmanager gửi Gmail/Telegram

sudo systemctl start node_exporter
# Sau khi phục hồi -> nhận thông báo RESOLVED
```

---

