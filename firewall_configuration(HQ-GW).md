
Máy **HQ-GW** là trái tim của hệ thống. Nó vừa phải hứng **IPsec**, hứng **OpenVPN**, vừa phải làm trạm trung chuyển (**Hub**) cho Remote Client ping sang Branch.

---

## Hướng dẫn

Mở file /etc/nftables.conf và dán toàn bộ nội dung sau vào:

---

## Cấu hình nftables

```bash
#!/usr/sbin/nft -f

flush ruleset

# --- BẢNG FILTER ---
table inet filter {
    # 1. Luồng đi trực tiếp vào Gateway (Input)
    chain input {
        type filter hook input priority 0; policy drop;

        # Chấp nhận loopback (nội bộ máy)
        iif "lo" accept

        # Chấp nhận các kết nối đã thiết lập (phản hồi từ internet/mạng khác)
        ct state established,related accept

        # --- BỔ SUNG: Cho phép Ping (ICMP) ---
        # Giúp bạn kiểm tra kết nối từ Máy B và Máy C tới IP WAN của A
        ip protocol icmp accept

        # Cho phép SSH để quản trị
        tcp dport 22 accept

        # Mở cửa cho IPsec Site-to-Site (IKEv2 & ESP Payload)
        udp dport { 500, 4500 } accept
        ip protocol esp accept

        # Mở cửa cho OpenVPN Remote Access
        udp dport 1194 accept
    }

    # 2. Luồng trung chuyển (Forward)
    chain forward {
        type filter hook forward priority 0; policy drop;

        # Cho phép luồng phản hồi
        ct state established,related accept

        # HQ LAN <--> Branch LAN (Qua IPsec)
        ip saddr 192.168.10.0/24 ip daddr 192.168.20.0/24 accept
        ip saddr 192.168.20.0/24 ip daddr 192.168.10.0/24 accept

        # Remote Client <--> HQ LAN (Qua OpenVPN)
        ip saddr 10.8.0.0/24 ip daddr 192.168.10.0/24 accept
        ip saddr 192.168.10.0/24 ip daddr 10.8.0.0/24 accept

        # Remote Client <--> Branch LAN (Hub-and-Spoke qua A)
        ip saddr 10.8.0.0/24 ip daddr 192.168.20.0/24 accept
        ip saddr 192.168.20.0/24 ip daddr 10.8.0.0/24 accept
    }
}

# --- BẢNG NAT ---
table ip nat {
    chain postrouting {
        type nat hook postrouting priority 100; policy accept;
        
        # CHÚ Ý: Chỉ Masquerade khi đi ra Internet thật (giả sử qua interface enp18s0)
        # KHÔNG masquerade khi đi vào Tunnel VPN để tránh làm mất IP gốc của Client
        ip saddr 10.8.0.0/24 oifname "enp18s0" masquerade
    }
}
```

### Kích hoạt Tường lửa an toàn
#### 1. Kiểm tra và nạp cấu hình
```bash
nft -f /etc/nftables.conf
```
#### 2. Khởi động và kích hoạt dịch vụ
```bash
systemctl restart nftables
systemctl enable nftables
```
#### 3. Kiểm tra lại rule (phục vụ báo cáo)
```bash
nft list ruleset
```
