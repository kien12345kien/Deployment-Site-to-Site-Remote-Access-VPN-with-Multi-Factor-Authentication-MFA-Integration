### BƯỚC 1: Cài đặt "Điệp viên" Node Exporter (Trên máy chịu tải HQ-GW)

Máy **HQ-GW** là nơi chúng ta cần "soi" kỹ nhất. Rất may trên Debian, cài đặt cục bộ cực kỳ dễ:

1. **Cài đặt qua apt:**

```bash
apt update
apt install prometheus-node-exporter -y
```

2. **Kiểm tra dịch vụ:**

```bash
systemctl status prometheus-node-exporter
```

(Dịch vụ này sẽ tự động chạy ngầm và mở port `9100`).

---

### BƯỚC 2: Cài đặt "Kho dữ liệu" Prometheus (Trên máy HQ-Client)

Để tránh làm nặng máy Gateway, em nên cài Prometheus và Grafana lên máy **HQ-Client (coi như nó gánh thêm vai trò Monitoring Server).

1. **Cài đặt Prometheus:**

```bash
apt install prometheus -y
```

2. **Chỉ đường cho Prometheus tới lấy data từ HQ-GW:** Em mở file cấu hình của Prometheus:

```bash
nano /etc/prometheus/prometheus.yml
```

Cuộn xuống dưới cùng, thêm một `job` mới để nó biết đường sang HQ-GW (Giả sử IP LAN của HQ-GW là `192.168.10.1`):

```yaml
    scrape_configs:
      - job_name: 'prometheus'
        static_configs:
          - targets: ['localhost:9090']
    
      # Thêm khối này vào dưới cùng:
      - job_name: 'hq-gateway'
        static_configs:
          - targets: ['192.168.10.1:9100']
```

3. **Khởi động lại Prometheus để nhận cấu hình mới:**

```bash
systemctl restart prometheus
```

---

### BƯỚC 3: Cài đặt "Bảng điều khiển" Grafana (Trên máy HQ-Client)

Grafana không có sẵn trong repo mặc định của Debian, nên ta cần tải từ kho chính hãng:

1. **Thêm key và repo của Grafana:**

```bash
apt-get install -y apt-transport-https wget gnupg
mkdir -p /etc/apt/keyrings/
wget -q -O - https://apt.grafana.com/gpg.key | gpg --dearmor | tee /etc/apt/keyrings/grafana.gpg > /dev/null
echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main" | tee /etc/apt/sources.list.d/grafana.list
```

2. **Cài đặt và Khởi động:**

```bash
apt-get update
apt-get install grafana -y
systemctl enable --now grafana-server
```


---

### BƯỚC 4: Kết nối và "Hóa phép" ra Biểu đồ (Thực hiện trên Trình duyệt web)

Bây giờ, em hãy mở trình duyệt web trên máy tính thật của em và truy cập vào Grafana: 👉 **URL:** `[http://192.168.10.10:3000](http://192.168.10.10:3000)` 👉 **Tài khoản mặc định:** `admin` / Mật khẩu: `admin` (Đăng nhập xong nó sẽ bắt đổi mật khẩu).

**Tiến hành kết nối:**

1. Vào menu **Connections** -> **Data Sources** -> Chọn **Add data source**.

2. Chọn **Prometheus**.

3. Ở ô _Prometheus server URL_, nhập: `http://localhost:9090` (Vì nó cài cùng máy với Grafana).

4. Cuộn xuống dưới cùng, bấm **Save & Test**. Hiện chữ màu xanh là thành công!


**Tuyệt chiêu "Ăn sẵn" biểu đồ chuyên nghiệp:** Thay vì phải ngồi vẽ từng cái chart mệt mỏi, cộng đồng đã làm sẵn những Dashboard cực kỳ đỉnh cao.

1. Vào menu **Dashboards** -> Bấm nút **New** (ở góc phải) -> Chọn **Import**.

2. Nhập mã ID "thần thánh": **`1860`** (Đây là Node Exporter Full - Dashboard nổi tiếng nhất thế giới).

3. Bấm **Load**, chọn Data Source là _Prometheus_ vừa tạo, rồi bấm **Import**.


BÙM! Ngay lập tức, một bảng điều khiển đen tuyền, ngập tràn các biểu đồ: CPU Load, RAM, Network Traffic, Disk I/O của máy HQ-GW sẽ hiện ra sống động theo thời gian thực!

---

### 💡 CÁCH DÙNG GRAFANA ĐỂ CHỨNG MINH TRONG ĐỒ ÁN

Để lấy điểm tuyệt đối cho phần Đánh giá Hiệu năng, em hãy làm thao tác sau:

1. Mở cái Dashboard Grafana vừa tạo lên, để nguyên đó.

2. Mở Terminal, dùng máy Client bắn lệnh `iperf3 -P 9` (Test 9 luồng).

3. Quay lại nhìn Grafana:

    - Em sẽ thấy biểu đồ **Network Traffic** vọt lên đỉnh (chạm mốc 289 Mbps).
        
    - Biểu đồ **CPU Basic** sẽ đổi màu, cột `%system` (màu vàng) và `%user` (màu xanh) dâng cao.
        
    - _(Nâng cao)_: Em có thể tạo một panel mới, dùng câu lệnh PromQL: `irate(node_network_receive_drop_total[1m])` để vẽ biểu đồ Tỉ lệ rớt gói tin!

## TROUBLE SHOTING
### BƯỚC 1: Kiểm tra "Điệp viên" Node Exporter (Trên máy HQ-GW)

Có thể do nãy giờ ép máy chạy quá tải (test 9 luồng liên tục), dịch vụ báo cáo số liệu Node Exporter đã bị crash (sập).

- **Thao tác:** Mở Terminal của máy **HQ-GW** và gõ lệnh:

```bash
systemctl status prometheus-node-exporter
```

- **Xử lý:** Nếu thấy nó báo chữ đỏ hoặc `inactive`, em hãy gõ lệnh đánh thức nó dậy:

```bash
systemctl restart prometheus-node-exporter
```

### BƯỚC 2: Kiểm tra lại bức tường `nftables` (Trên máy HQ-GW)

Nếu Node Exporter vẫn `active (running)`, thì 100% là do tường lửa. Có thể em chưa lưu luật tường lửa vào file cứng, nên một thao tác reload nào đó đã làm mất luật mở cổng 9100.

- **Thao tác:** Đứng trên máy **HQ-GW**, em kiểm tra lại file cấu hình:

````bash
cat /etc/nftables.conf | grep 9100
*   **Xử lý:** Nếu lệnh trên KHÔNG in ra dòng chữ nào, em phải mở file ra thêm lại luật vào `chain input`:   

nftables
	ip saddr 192.168.10.10 tcp dport 9100 accept
    Sau đó, bắt buộc phải nạp lại cấu hình:
    sudo nft -f /etc/nftables.conf
```

### BƯỚC 3: Test nhanh từ máy HQ-Client
Để biết chắc chắn đường truyền đã thông chưa mà không cần đợi Prometheus tự quét, em đứng trên máy **HQ-Client** và gõ:

curl -m 5 http://192.168.10.1:9100/metrics
````

Nếu màn hình nổ ra một đống chữ số, tức là thông! Lúc này quay lại trang Targets của Prometheus (bức ảnh em vừa chụp) f5 lại, nó sẽ xanh lét chữ `UP`. Khi nó đã `UP` ổn định, em sang Grafana làm mới lại biến (Refresh) là lấy được biểu đồ.

## Uninstall
Để gỡ cài đặt sạch sẽ toàn bộ hệ thống giám sát này, bạn cần thao tác trên cả 2 máy (HQ-GW và HQ-Client) để dừng dịch vụ, xóa phần mềm và dọn dẹp các tệp cấu hình/dữ liệu còn sót lại.

Dưới đây là các bước chi tiết:

### BƯỚC 1: Trên máy HQ-GW (Gỡ bỏ Node Exporter)

Bạn mở Terminal trên máy HQ-GW và chạy lần lượt các lệnh sau:

1. **Dừng dịch vụ:**

```bash
systemctl stop prometheus-node-exporter
```

2. **Gỡ cài đặt phần mềm và xóa cấu hình:**

```bash
apt remove --purge prometheus-node-exporter -y
```

3. **Dọn dẹp các gói phụ thuộc không còn dùng đến:**

```bash
apt autoremove -y
```

---

### BƯỚC 2: Trên máy HQ-Client (Gỡ bỏ Prometheus và Grafana)

Bạn mở Terminal trên máy HQ-Client và chạy các lệnh sau:

#### 1. Gỡ bỏ Prometheus

- **Dừng dịch vụ và gỡ phần mềm:**

```bash
systemctl stop prometheus
apt remove --purge prometheus -y
```

- **Xóa toàn bộ dữ liệu và file cấu hình của Prometheus:**

```bash
rm -rf /etc/prometheus
rm -rf /var/lib/prometheus
```

#### 2. Gỡ bỏ Grafana

- **Dừng dịch vụ và gỡ phần mềm:**

```bash
systemctl stop grafana-server
apt remove --purge grafana -y
```

- **Xóa toàn bộ dữ liệu, biểu đồ (Dashboards) và log của Grafana:**
```bash
rm -rf /etc/grafana
rm -rf /var/lib/grafana
rm -rf /var/log/grafana
```

- _(Tùy chọn)_ **Xóa kho lưu trữ (Repository) và Key của Grafana** để hệ thống apt gọn gàng hơn:

```bash
rm /etc/apt/sources.list.d/grafana.list
rm /etc/apt/keyrings/grafana.gpg
```

#### 3. Dọn dẹp hệ thống lần cuối

Chạy lệnh này để xóa các gói phụ thuộc thừa và cập nhật lại danh sách dịch vụ:

Bash

```bash
apt autoremove -y
apt update
systemctl daemon-reload
```