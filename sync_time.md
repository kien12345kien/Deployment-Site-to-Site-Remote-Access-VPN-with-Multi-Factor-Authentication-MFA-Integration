### Cài đặt

```bash
apt update
apt install systemd-timesyncd
```

### Thiết lập múi giờ (nếu cần)

```bash
timedatectl set-timezone Asia/Ho_Chi_Minh
```
### Kiểm tra trạng thái

```bash
timedatectl status
```

Nếu thấy `NTP service: active` và `System clock synchronized: yes` là OK.

### Bật đồng bộ nếu chưa bật

```bash
timedatectl set-ntp true
```

### Kiểm tra lại

```bash
timedatectl status
```