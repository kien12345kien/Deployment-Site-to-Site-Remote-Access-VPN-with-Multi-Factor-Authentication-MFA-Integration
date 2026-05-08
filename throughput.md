# VPN Measurement Data Compilation

This file contains the raw output logs for all four testing scenarios: 1 Client, 3 Clients, 5 Clients, and 9 Clients.

---

#  REMOTE ACCESS

### [Thiết lập VPN Remote Access](openvpn_and_freeradius))

### Before

#### HQGW

```bash
mpstat -I SUM -P ALL 1 30 
```

```bash
sar -u 1 30
```
#### HQ-Client

```bash
iperf3 -s
```

#### RMCL

```bash
iperf3 -c 192.168.10.100 -t 30 -P 1
```


## After

#### Cài đặt / Gỡ  IRQ Balance & Receive Packet Steering (RPS)

**Cài đặt**

```bash
apt update
apt install irqbalance -y
```

**Khởi động và cho phép chạy ngầm:**

```bash
systemctl enable --now irqbalance
```

**Kiểm tra trạng thái:**

```bash
systemctl status irqbalance
```

**Dừng dịch vụ ngay lập tức:**

```bash
systemctl stop irqbalance
```

**Ngăn dịch vụ tự khởi động cùng hệ thống:**

```bash
systemctl disable irqbalance
```

**Kiểm tra trạng thái:**

```bash
systemctl status irqbalance
```

***Gỡ bỏ hoàn toàn (Uninstall)**

```bash
apt-get purge irqbalance
```
#### HQGW

```bash
mpstat -I SUM -P ALL 1 30 
```

```bash
sar -u 1 30
```
#### HQ-Client

```bash
iperf3 -s
```

#### RMCL

```bash
iperf3 -c 192.168.10.100 -t 30 -P 1
```


#  SITE-TO-SITE 

### [Thiết lập VPN Site-to-Site](vpn_s2s_configuration)

### Before

#### HQGW

```bash
mpstat -I SUM -P ALL 1 30 
```

```bash
sar -u 1 30
```
#### HQ-Client

```bash
iperf3 -s
```

#### RMCL

```bash
iperf3 -c 192.168.10.100 -t 30 -P 1
```


## After

#### Cài đặt / Gỡ  IRQ Balance & Receive Packet Steering (RPS)

**Cài đặt**

```bash
sudo apt update
sudo apt install irqbalance -y
```

**Khởi động và cho phép chạy ngầm:**

```bash
sudo systemctl enable --now irqbalance
```

**Kiểm tra trạng thái:**

```bash
systemctl status irqbalance
```

**Dừng dịch vụ ngay lập tức:**

```bash
systemctl stop irqbalance
```

**Ngăn dịch vụ tự khởi động cùng hệ thống:**

```bash
systemctl disable irqbalance
```

**Kiểm tra trạng thái:**

```bash
systemctl status irqbalance
```

***Gỡ bỏ hoàn toàn (Uninstall)**

```bash
apt-get purge irqbalance
```
#### HQGW

```bash
mpstat -I SUM -P ALL 1 30 
```

```bash
sar -u 1 30
```
#### HQ-Client

```bash
iperf3 -s
```

#### RMCL

```bash
iperf3 -c 192.168.10.100 -t 30 -P 1
```
