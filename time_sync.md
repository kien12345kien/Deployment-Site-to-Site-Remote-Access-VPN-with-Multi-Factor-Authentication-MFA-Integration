### Cách kiểm tra nhanh nhất (Nếu bạn không muốn cài đặt phức tạp)

Nếu bạn chỉ muốn máy B khớp giờ ngay lập tức mà không quan tâm đến NTP, hãy dùng lệnh này trên **Máy B** để ép nó lấy giờ theo **Máy A** qua giao thức SSH:

```
date -s "$(ssh root@IP_MAY_A 'date -u')"
```

_(Lệnh này sẽ lấy giờ UTC từ máy A và áp thẳng vào máy B. Bạn sẽ cần nhập password của máy A)._


### Cài đặt thủ công

---

### 1. Kiểm tra trạng thái thời gian hiện tại

Trước khi thay đổi, bạn nên xem cấu hình hiện tại bằng lệnh:


```
timedatectl status
```

Lệnh này sẽ cho bạn biết thời gian hiện tại, múi giờ và liệu chế độ tự động cập nhật (NTP) có đang bật hay không.

### 2. Tắt chế độ cập nhật tự động

Để chỉnh thời gian thủ công, bạn **phải** tắt tính năng tự động đồng bộ hóa thời gian (NTP). Nếu không, hệ thống sẽ tự động ghi đè lại giờ chuẩn từ máy chủ.


```
sudo timedatectl set-ntp false
```

### 3. Điều chỉnh thời gian và ngày tháng

Sử dụng cú pháp: `sudo timedatectl set-time "YYYY-MM-DD HH:MM:SS"`

- **Ví dụ:** Để chỉnh thành 15:30:00 ngày 25 tháng 12 năm 2024:

```bash
sudo timedatectl set-time "2024-12-25 15:30:00"
```
