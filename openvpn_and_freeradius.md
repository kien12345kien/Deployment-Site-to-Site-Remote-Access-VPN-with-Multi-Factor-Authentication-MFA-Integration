Giai đoạn này thực hiện kết nối giữa OpenVPN Server và FreeRADIUS để hoàn tất hệ thống xác thực đa yếu tố.

  

---

  

## Giai đoạn 1: Cấu hình trên HQ-GW (OpenVPN Gateway)

  

### Bước 1: Cài đặt Module kết nối RADIUS

Cài đặt module PAM để OpenVPN có thể gửi yêu cầu xác thực tới máy chủ RADIUS.

```bash
apt update
apt install libpam-radius-auth -y
```

### Bước 2: Khai báo máy chủ HQ-RADIUS

Cấu hình địa chỉ IP và mật khẩu dùng chung (shared secret) để HQ-GW "nói chuyện" được với HQ-RADIUS.

Mở file: `nano /etc/pam_radius_auth.conf`

Tìm dòng `127.0.0.1` (thường ở dòng 18), xóa đi hoặc comment lại và thêm dòng sau:

```conf
192.168.10.10 RadiusSecret2026 3
```

*(Trong đó `3` là thời gian timeout tính bằng giây).*

### Bước 3: Tạo "Luật PAM" cho OpenVPN

Tạo file cấu hình để định nghĩa cách dịch vụ OpenVPN sử dụng module RADIUS:

```bash
nano /etc/pam.d/openvpn
```

Dán nội dung sau vào:

```bash
auth sufficient pam_radius_auth.so
account required pam_permit.so
```

### Bước 4: Kích hoạt Plugin trong OpenVPN Server

Mở file cấu hình server: `nano /etc/openvpn/server/server.conf`
  
Thêm các dòng sau vào cuối file để bật tính năng xác thực người dùng qua plugin PAM:

```conf
verify-client-cert none
username-as-common-name
# Lưu ý: Đường dẫn plugin có thể thay đổi tùy kiến trúc CPU (x86_64 hoặc aarch64)
plugin /usr/lib/aarch64-linux-gnu/openvpn/plugins/openvpn-plugin-auth-pam.so openvpn
```

**Khởi động lại dịch vụ:**

```bash
systemctl daemon-reload
systemctl restart openvpn-server@server
```

  

---

  

## Giai đoạn 2: Điều chỉnh cuối cùng trên HQ-RADIUS

Trên máy **HQ-RADIUS**, cần tinh chỉnh cấu hình PAM để đảm bảo quá trình chuyển tiếp mật khẩu và OTP mượt mà hơn.

Mở file: `nano /etc/pam.d/radiusd`

Sửa lại dòng thứ 2, thay `use_first_pass` bằng `try_first_pass`:

```bash
auth requisite pam_google_authenticator.so forward_pass secret=/etc/freeradius/3.0/google-auth/${USER} user=freerad
auth required pam_unix.so try_first_pass
```

  

---

  

## Giai đoạn 3: Kiểm tra trên Remote-Client

Bây giờ bạn cần cập nhật file cấu hình trên máy khách để hệ thống hỏi thông tin đăng nhập.

1. **Sửa file cấu hình Client:**

Mở file `intern-client.ovpn` trên máy Remote-Client, thêm dòng sau vào cuối file:

```conf
auth-user-pass
```

2. **Khởi tạo kết nối:**

```bash
openvpn --config intern-client.ovpn
```

3. **Xác thực:**
* **Username:** Nhập `mfa_user`
* **Password:** Nhập theo cú pháp `<Mật_khẩu_hệ_thống><Mã_OTP>`
* *Ví dụ:* Nếu mật khẩu là `matkhau123` và mã trên điện thoại là `456789`, bạn nhập: `matkhau123456789`

Nếu Terminal hiện `Initialization Sequence Completed`, bạn đã thiết lập thành công hệ thống VPN bảo mật với xác thực 2 lớp (MFA)!