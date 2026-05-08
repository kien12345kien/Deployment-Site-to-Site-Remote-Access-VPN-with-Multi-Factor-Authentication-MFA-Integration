
### Bước 1: Cấu hình HQ-GW (Trụ sở chính)

Tạo file cấu hình cho `swanctl`. Bạn có thể viết thẳng vào `/etc/swanctl/swanctl.conf` hoặc tạo file riêng trong `/etc/swanctl/conf.d/`.
**File:** `/etc/swanctl/conf.d/hq-to-branch.conf`

```bash
connections {
    hq-to-branch {
        local_addrs = 10.10.10.1
        remote_addrs = 10.10.20.1  # Trỏ thẳng đến IP máy C

        local {
            auth = psk
            id = hq-gateway
        }
        remote {
            auth = psk
            id = br-gateway
        }

        children {
            net-to-net {
                local_ts = 192.168.10.0/24, 10.8.0.0/24
                remote_ts = 192.168.20.0/24
                esp_proposals = aes256gcm16-sha256-modp2048
                start_action = trap # Tự động kích hoạt khi có traffic
            }
        }
        version = 2
        proposals = aes256gcm16-sha256-modp2048
    }
}

secrets {
    ike-hq-br {
        id-hq = hq-gateway
        id-br = br-gateway
        secret = "Key_Bi_Mat_Sieu_Manh_2026"
    }
}
```

  

### Bước 2: Cấu hình BR-GW (Chi nhánh)

Tương tự, trên máy BR-GW, cấu hình đảo ngược lại các thông số IP và ID.

  

**File:** `/etc/swanctl/conf.d/branch-to-hq.conf`

```bash
connections {
    branch-to-hq {
        local_addrs = 10.10.20.1
        remote_addrs = 10.10.10.1

        local {
            auth = psk
            id = br-gateway
        }
        remote {
            auth = psk
            id = hq-gateway
        }

        children {
            net-to-net {
                local_ts = 192.168.20.0/24
                remote_ts = 192.168.10.0/24, 10.8.0.0/24
                esp_proposals = aes256gcm16-sha256-modp2048
                start_action = trap
            }
        }
        version = 2
        proposals = aes256gcm16-sha256-modp2048
    }
}

secrets {
    ike-hq-br {
        id-hq = hq-gateway
        id-br = br-gateway
        secret = "Key_Bi_Mat_Sieu_Manh_2026"
    }
}
```

  

### Bước 3: Nạp cấu hình và Khởi tạo đường hầm

Sử dụng `swanctl` để nạp cấu hình vào bộ nhớ mà không cần khởi động lại dịch vụ.

  

1. **Nạp cấu hình** (Thực hiện trên CẢ 2 GW):

```bash
swanctl --load-all
```

  

2. **Kích hoạt kết nối**:

Mặc dù tunnel có thể tự bật khi có traffic, bạn có thể ép hầm bật lên để kiểm tra bằng lệnh:

```bash

# Trên máy HQ hoặc BR
swanctl --initiate --child net-to-net
```

  

### Bước 4: Kiểm tra (Verify)

1. **Xem trạng thái IPsec SAs**:

```bash
swanctl --list-sas
```

*Kết quả thành công:* Bạn sẽ thấy dòng chữ `ESTABLISHED`, thuật toán mã hóa (VD: `AES_GCM_16`), và cặp mạng `192.168.10.0/24 === 192.168.20.0/24`.

  

2. **Ping test (Kiểm tra thực tế)**:

Đứng từ máy **HQ-Client** (`192.168.10.100`), ping sang IP của **BR-GW** hoặc Client phía BR:

```bash
ping 192.168.20.1
```

Nếu có phản hồi (`reply`), gói tin đã đi qua đường hầm IPsec thành công.

### Ngắt kết nối
#### 1. Tắt tạm thời các kết nối (Gỡ bỏ Tunnel)

Nếu bạn chỉ muốn ngắt đường hầm (Tunnel) giữa máy A và máy C mà không muốn gỡ cài đặt phần mềm, hãy sử dụng lệnh sau:

- **Ngắt một kết nối cụ thể:**

```bash
swanctl --terminate --ike hq-to-branch
```

_(Thay `hq-to-branch` bằng tên kết nối trong file `.conf` của bạn)._

- **Ngắt tất cả các kết nối đang hoạt động:**
```bash
sudo swanctl --terminate --all
```

#### 2. Dừng hoàn toàn dịch vụ IPSec

Để đảm bảo IPSec không tự động chạy ngầm và giải phóng các cổng (UDP 500, 4500):

- **Dừng dịch vụ ngay lập tức:**
```bash
systemctl stop strongswan
# Hoặc tùy hệ thống:
systemctl stop charon
```

- **Ngăn IPSec tự khởi động cùng hệ thống:**

```bash
systemctl disable strongswan
```