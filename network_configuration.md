### Bước 1: Kích hoạt IP Forwarding (Định tuyến)

Trên Linux, để máy chủ hoạt động như một Router cho phép các dải mạng đi qua nó, bạn phải bật tính năng IPv4 Forwarding.

  

Thực hiện trên cả **HQ-GW** và **BR-GW**:

```bash
echo "net.ipv4.ip_forward = 1" | tee -a /etc/sysctl.conf
sysctl -p
```

  

### Bước 2: Cấu hình IP Interface
Chỉnh sửa cấu hình mạng trên các Gateway.
#### HQ-GW
```bash
# /etc/network/interfaces

allow-hotplug enp2s0
iface enp2s0 inet dhcp

#WAN
auto enp18s0
iface enp18s0 inet static
        address 10.10.10.1
        netmask 255.255.255.252

# LAN
auto enp10s0
iface enp10s0 inet static
        address 192.168.10.1
        netmask 255.255.255.0
```

#### BR-GW
```bash
# /etc/network/interfaces
# WAN
#WAN
auto enp18s0
iface enp18s0 inet static
        address 10.10.20.1
        netmask 255.255.255.252

# LAN
auto enp10s0
iface enp10s0 inet static
        address 192.168.20.1
        netmask 255.255.255.0
```

#### WAN
```bash
# /etc/network/interfaces

auto enp18s0
iface enp18s0 inet static
        address 10.10.10.2
        netmask 255.255.255.252

auto enp10s0
iface enp10s0 inet static
        address 10.10.20.2
        netmask 255.255.255.252

auto enp11s0
iface enp11s0 inet static
        address 10.10.30.2
        netmask 255.255.255.252
```

#### HQ FREERADIUS SERVER
```bash
# /etc/network/interfaces
# LAN
auto enp10s0
iface enp10s0 inet static
	address 192.168.10.10
	netmask 255.255.255.0
	gateway 192.168.10.1
```

#### HQ CLIENT
```bash
# /etc/network/interfaces
# LAN
auto enp10s0
iface enp10s0 inet static
	address 192.168.10.100
	netmask 255.255.255.0
	gateway 192.168.10.1
```

#### BR-CLIENT
```bash
# /etc/network/interfaces
# LAN
auto enp10s0
iface enp10s0 inet static
	address 192.168.20.10
	netmask 255.255.255.0
	gateway 192.168.20.1
```

#### BR-CLIENT
```bash
# /etc/network/interfaces
# LAN
auto enp10s0
iface enp10s0 inet static
	address 192.168.10.5
	netmask 255.255.255.0
	gateway 192.168.10.1
```

#### Remote-CLIENT
```bash
# /etc/network/interfaces
# WAN
auto enp18s0
iface enp18s0 inet static
        address 10.10.30.1
        netmask 255.255.255.252
```

#### Khởi động lại dịch vụ mạng:
```bash
systemctl restart networking.service
```

#### Routing 

hq - br
```bash
ip route add 10.10.20.0/30 via 10.10.10.2
```

br - hq
```bash
ip route add 10.10.10.0/30 via 10.10.20.2
```

br - rmcl
```bash
ip route add 10.10.30.0/30 via 10.10.20.2
```

rmcl - br
```bash
ip route add 10.10.20.0/30 via 10.10.30.2
```

hq - rmcl
```bash
ip route add 10.10.30.0/30 via 10.10.10.2
```

rmcl - hq
```bash
ip route add 10.10.10.0/30 via 10.10.30.2
```
