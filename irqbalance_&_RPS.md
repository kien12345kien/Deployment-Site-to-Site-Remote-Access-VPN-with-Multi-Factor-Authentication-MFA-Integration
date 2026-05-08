Trong môi trường máy ảo (VM) và đặc biệt là với giao thức IPsec (như StrongSwan), việc chỉ sử dụng cấu hình mặc định thường dẫn đến tình trạng nghẽn cổ chai tại một nhân CPU đơn lẻ.

  

Hướng dẫn này sẽ giúp bạn triển khai bộ đôi "vũ khí kép": **irqbalance** (phân tải phần cứng) và **RPS** (phân tải phần mềm) để tối ưu hóa việc xử lý gói tin và giải mã AES-256-GCM trên tất cả các nhân CPU.

  

---

  

### Bước 1: Cài đặt và kích hoạt irqbalance (Hardware Interrupts)

  

`irqbalance` là một daemon chạy ngầm, tự động theo dõi và phân phối các luồng ngắt phần cứng (Hardware Interrupts) từ card mạng đều ra các nhân CPU vật lý.

  

1. **Cài đặt công cụ:**

```bash

apt update

apt install irqbalance -y

```

  

2. **Khởi động và cho phép chạy cùng hệ thống:**

```bash

systemctl enable --now irqbalance

```

  

3. **Kiểm tra trạng thái:**

```bash

systemctl status irqbalance

```

*(Đảm bảo trạng thái báo màu xanh **active (running)**).*

  

---

  

### Bước 2: Kích hoạt RPS (Receive Packet Steering)

**Tại sao cần RPS?**

Trong các môi trường ảo hóa (VMware, VirtualBox, KVM), card mạng ảo thường chỉ có **1 hàng đợi duy nhất (Single Queue)**. `irqbalance` chỉ có tác dụng khi card mạng hỗ trợ Multi-Queue. Nếu chỉ có 1 hàng đợi, toàn bộ luồng ngắt vẫn sẽ dồn vào CPU 0.


**RPS (Receive Packet Steering)** là tính năng của nhân Linux giúp "chia bài" ngay sau khi nhận gói tin, đẩy bớt công việc xử lý giao thức (SoftIRQ) và giải mã IPsec sang các CPU khác.

#### Cấu hình RPS cho hệ thống 2 CPU:

Để bật cho cả CPU 0 và CPU 1, chúng ta sử dụng chuỗi nhị phân `11`, tương đương với giá trị Hex là `3`.

  

1. **Xác định Interface:** Giả sử card mạng WAN là `enp18s0` và LAN là `enp10s0`.

2. **Áp dụng cấu hình:**

```bash

# Cho card WAN

echo 3 | tee /sys/class/net/enp18s0/queues/rx-0/rps_cpus

  

# Cho card LAN (nếu cần)

echo 3 | tee /sys/class/net/enp10s0/queues/rx-0/rps_cpus

```

  

---

  

### Bước 3: Nghiệm thu và Kiểm tra hiệu năng

  

Sau khi cấu hình, bạn có thể kiểm tra xem tải trọng đã được phân bổ đều hay chưa.

  

1. **Theo dõi ngắt hệ thống (Real-time):**

```bash

watch -n 1 "cat /proc/interrupts"

```

*Kết quả mong đợi:* Các chỉ số ngắt sẽ tăng đều ở cả CPU 0 và CPU 1 thay vì chỉ tập trung ở một nhân.

  

2. **Kiểm tra băng thông (Throughput):**

Thực hiện test tải (ví dụ sử dụng `iperf3` với nhiều luồng).

*Kết quả mong đợi:*

- Ngắt phần mềm (SoftIRQ) được chia đều.

- Chỉ số `%system` (kiểm tra bằng lệnh `sar` hoặc `top`) tăng đều ở các core.

- Tổng băng thông hệ thống sẽ cải thiện rõ rệt (có thể tăng từ 30-50% tùy cấu hình) do tận dụng được sức mạnh đa nhân để giải mã dữ liệu.