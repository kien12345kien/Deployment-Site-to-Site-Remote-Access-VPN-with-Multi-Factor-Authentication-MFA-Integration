Tài liệu này hướng dẫn cách cấu hình FreeRADIUS để tích hợp xác thực đa yếu tố (MFA) sử dụng Google Authenticator và PAM trên máy **HQ-RADIUS**.

  

---

  

## Bước 1: Khai báo OpenVPN Gateway là RADIUS Client

Cần cho phép máy OpenVPN (HQ-GW) gửi yêu cầu xác thực tới RADIUS.

  

Mở file cấu hình client: `nano /etc/freeradius/3.0/clients.conf`

  

Thêm đoạn sau vào cuối file:

```conf
client hq-gw-openvpn {
ipaddr = 192.168.10.1
secret = RadiusSecret2026
require_message_authenticator = no
}
```
---

  

## Bước 2: Cấu hình Module PAM cho RADIUS

Thiết lập cách RADIUS giao tiếp với hệ thống để kiểm tra mật khẩu và mã OTP.
Tạo/Mở file cấu hình PAM: `nano /etc/pam.d/radiusd`
Xóa nội dung cũ (nếu có) và nhập chính xác 2 dòng sau:
```bash
auth requisite pam_google_authenticator.so forward_pass secret=/etc/freeradius/3.0/google-auth/${USER} user=freerad
auth required pam_unix.so use_first_pass
```
---

  

## Bước 3: Kích hoạt Module PAM trong FreeRADIUS
1. **Bật module PAM:**

```bash
ln -s /etc/freeradius/3.0/mods-available/pam /etc/freeradius/3.0/mods-enabled/
```

2. **Chỉnh sửa Site mặc định:**

Mở file: `nano /etc/freeradius/3.0/sites-enabled/default`
* Tìm khối `authenticate { ... }`, tìm dòng `#pam` và xóa dấu `#` để thành `pam`.
* Tìm khối `authorize { ... }`, cuộn xuống cuối khối (trước dấu `}` đóng) và thêm đoạn sau:
```conf
update control {
	Auth-Type := pam
}
```

---



## Bước 4: Tạo User và Khởi tạo MFA
1. **Tạo user hệ thống:**
```bash
adduser mfa_user
```
*(Nhập mật khẩu, ví dụ: `matkhau123`, các thông tin khác nhấn Enter).*

2. **Cấu hình Google Authenticator:**
Chuyển sang user vừa tạo:
```bash
su - mfa_user
google-authenticator
```

**Trả lời các câu hỏi như sau:**
* *Time-based tokens (y/n):* `y`
* **Lưu ý:** Quét mã QR hiện trên màn hình bằng ứng dụng Google Authenticator trên điện thoại.
* *Update .google_authenticator file (y/n):* `y`
* *Disallow multiple uses (y/n):* `y`
* *Permit time skew (y/n):* `n` (hoặc `y` tùy nhu cầu)
* *Enable rate-limiting (y/n):* `y`
Gõ `exit` để quay lại quyền root.

---

  

## Bước 5: Phân quyền và Quy hoạch tệp tin Bí mật
FreeRADIUS cần quyền để đọc file secret của Google Authenticator và mật khẩu hệ thống.
1. **Cấp quyền đọc file Shadow:**
```bash
usermod -a -G shadow freerad
```

  

2. **Di chuyển file secret vào thư mục quản lý của FreeRADIUS:**
```bash
mkdir -p /etc/freeradius/3.0/google-auth
cp /home/mfa_user/.google_authenticator /etc/freeradius/3.0/google-auth/mfa_user
# Phân quyền chặt chẽ
chown -R freerad:freerad /etc/freeradius/3.0/google-auth/
chmod 750 /etc/freeradius/3.0/google-auth/
chmod 400 /etc/freeradius/3.0/google-auth/mfa_user
```

  

---

  

## Bước 6: Kiểm tra xác thực (Test)
Sử dụng công cụ `radtest` để kiểm tra trực tiếp trên máy RADIUS.
**Cú pháp:**
```bash
# radtest <username> <mật_khẩu><mã_OTP> localhost 1812 testing123
radtest mfa_user matkhau123123456 localhost 1812 testing123
```
*(Trong đó `123456` là mã OTP hiện tại trên điện thoại của bạn).*
Nếu nhận được phản hồi **Access-Accept**, chúc mừng bạn đã cấu hình MFA thành công!