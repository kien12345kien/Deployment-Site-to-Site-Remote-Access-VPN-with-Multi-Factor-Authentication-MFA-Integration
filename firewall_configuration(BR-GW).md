
Chi nhánh đóng vai trò là một điểm **Spoke**. Nhiệm vụ của nó nhẹ nhàng hơn: chỉ cần duy trì kết nối **IPsec** với Trụ sở và cho phép dữ liệu từ LAN đi ra, đồng thời đón khách từ Remote Client sang thăm.

---

## Hướng dẫn

Mở file /etc/nftables.conf và dán nội dung sau:

---

## Cấu hình nftables

```bash
#!/usr/sbin/nft -f

flush ruleset

table inet filter {
    chain input {
        type filter hook input priority 0; policy drop;

        iif "lo" accept
        ct state established,related accept

        # Cho phép Ping (ICMP) - Cần thiết để kiểm tra kết nối WAN
        ip protocol icmp accept 

        # Cho phép SSH
        tcp dport 22 accept

        # Mở cửa cho IPsec
        udp dport { 500, 4500 } accept
        ip protocol esp accept
        
        # Log lại các gói tin bị drop để debug (xem trong dmesg)
        # limit rate 5/minute log prefix "nft-input-drop: "
    }

    chain forward {
        type filter hook forward priority 0; policy drop;
        ct state established,related accept

        # HQ LAN <-> Branch LAN
        ip saddr 192.168.20.0/24 ip daddr 192.168.10.0/24 accept
        ip saddr 192.168.10.0/24 ip daddr 192.168.20.0/24 accept

        # Remote Client <-> Branch LAN
        ip saddr 192.168.20.0/24 ip daddr 10.8.0.0/24 accept
        ip saddr 10.8.0.0/24 ip daddr 192.168.20.0/24 accept
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
