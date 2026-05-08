Tài liệu này hướng dẫn chi tiết cách thiết lập OpenVPN Server trên Debian 13 và cấu hình Client kết nối cơ bản sử dụng Chứng chỉ số (Certificate).
## Giai đoạn 1: Thiết lập OpenVPN Server

  
### Bước 1: Cài đặt OpenVPN và Easy-RSA

Easy-RSA là công cụ quản lý hạ tầng khóa công khai (PKI) giúp bạn tự tạo Trạm cấp phát chứng chỉ (CA) ngay trên server.

```bash
apt update
apt install openvpn easy-rsa -y
```

  

### Bước 2: Xây dựng hệ thống Chứng chỉ số (PKI)

Tạo thư mục làm việc để quản lý chứng chỉ:

```bash
make-cadir ~/openvpn-ca
cd ~/openvpn-ca
```

  

#### 1. Khởi tạo và tạo CA (Certificate Authority)

Khi hệ thống hỏi *Common Name*, nhấn **Enter** để lấy mặc định hoặc nhập `HQ-VPN-CA`.

```bash
./easyrsa init-pki
./easyrsa build-ca nopass
```

  

#### 2. Tạo Certificate và Private Key cho Server
Gõ `yes` khi hệ thống yêu cầu xác nhận.

```bash
./easyrsa gen-req hq-server nopass
./easyrsa sign-req server hq-server
```

  

#### 3. Tạo tham số Diffie-Hellman và TLS Auth Key
Các thành phần này giúp tăng cường bảo mật cho quá trình trao đổi khóa và chống tấn công DoS.

```bash
./easyrsa gen-dh
openvpn --genkey secret ta.key
```

  

#### 4. Cấp phát "chìa khóa" vào thư mục cấu hình OpenVPN
Chuyển các file cần thiết vào thư mục hệ thống:

```bash
cp pki/ca.crt pki/issued/hq-server.crt pki/private/hq-server.key pki/dh.pem ta.key /etc/openvpn/server/
```

  

---

  

### Bước 3: Cấu hình OpenVPN Server
Tạo file cấu hình chính: `nano /etc/openvpn/server/server.conf`

Dán nội dung cấu hình sau (đã được tối ưu):

```conf
# Thông số mạng cơ bản
port 1194
proto udp
dev tun

# Đường dẫn chứng chỉ
ca ca.crt
cert hq-server.crt
key hq-server.key
dh dh.pem
tls-auth ta.key 0

# Dải IP cấp cho Remote Client
server 10.8.0.0 255.255.255.0

# Đẩy route mạng LAN HQ xuống cho Client
push "route 192.168.10.0 255.255.255.0"
push "dhcp-option DNS 8.8.8.8"

# Cấu hình bảo mật
cipher AES-256-GCM
auth SHA256
keepalive 10 120
persist-key
persist-tun
user nobody
group nogroup

# Log & Status
status /var/log/openvpn/openvpn-status.log
log-append /var/log/openvpn/openvpn.log
verb 3
```

  

**Tạo thư mục log và cấp quyền:**

```bash
mkdir -p /var/log/openvpn
chown nobody:nogroup /var/log/openvpn
```

  

---

  

### Bước 4: Cấu hình NAT/Firewall

Để Client (10.8.0.0/24) truy cập được vào mạng LAN (192.168.10.0/24), server phải thực hiện NAT (Masquerade).

> **Lưu ý:** Thay `eth1` bằng tên card mạng thực tế của bạn (ví dụ: `ens34`).

```bash
sudo iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth1 -j MASQUERADE
```

  

---

  

### Bước 5: Khởi động OpenVPN Server

```bash
systemctl enable --now openvpn-server@server
systemctl status openvpn-server@server
```

Nếu thấy trạng thái `Active: active (running)`, server đã sẵn sàng.

  

---

  

## Giai đoạn 2: Cấu hình và Kiểm tra kết nối Client

Việc kiểm tra kết nối thuần túy qua Certificate trước khi cài đặt MFA giúp khoanh vùng lỗi dễ dàng hơn.

### Bước 1: Tạo Chứng chỉ (Certificate) cho Client

Quay lại thư mục Easy-RSA trên HQ-GW:
```bash
cd ~/openvpn-ca
```
1. **Khởi tạo Request:**
```bash
./easyrsa gen-req intern-client nopass
```

2. **Ký chứng chỉ:** Gõ `yes` để xác nhận.
```bash
./easyrsa sign-req client intern-client
```

  

### Bước 2: Đóng gói file `.ovpn` "All-in-One"

Sử dụng script sau để nhúng tất cả chứng chỉ vào một file duy nhất giúp Client sử dụng tiện lợi hơn. Chạy lệnh này tại thư mục `~/openvpn-ca`:

```bash
cat <<EOF > intern-client.ovpn
client
dev tun
proto udp
remote 10.10.10.1 1194
resolv-retry infinite
nobind
persist-key
persist-tun
remote-cert-tls server
cipher AES-256-GCM
auth SHA256
key-direction 1
verb 3
<ca>
$(cat pki/ca.crt)
</ca>
<cert>
$(awk '/BEGIN/,/END/' pki/issued/intern-client.crt)
</cert>
<key>
$(cat pki/private/intern-client.key)
</key>
<tls-auth>
$(cat /etc/openvpn/server/ta.key)
</tls-auth>
EOF
```

  

---

  

### Bước 3: Kiểm tra kết nối trên Remote-Client

1. **Chuyển file sang Remote-Client:**

Từ máy Remote-Client (IP: 10.10.10.100), thực hiện lệnh lấy file:

```bash
scp root@10.10.10.1:/root/openvpn-ca/intern-client.ovpn .
```

2. **Cài đặt và Chạy OpenVPN:**

```bash
apt update && apt install openvpn -y
openvpn --config intern-client.ovpn
```

3. **Nghiệm thu:**

Nếu thấy dòng `Initialization Sequence Completed`, kết nối đã thành công.

Thử Ping vào mạng nội bộ từ máy Client:

```bash
ping 192.168.10.100
```

Nếu thành công, hạ tầng mạng và VPN của bạn đã ổn định. Bạn đã sẵn sàng để nâng cấp lên xác thực MFA với FreeRADIUS.