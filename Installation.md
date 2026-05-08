# HQGW & BRGW

```bash
#all
apt install openvpn-auth-radius strongswan strongswan-swanctl swanctl libcharon-extra-plugins libstrongswan-standard-plugins libstrongswan-extra-plugins strongswan-pki nftables -y

#HQGW obly
apt install openvpn easy-rsa -y
apt install libpam-radius-auth -y
apt install iperf3 -y
apt install sysstat -y
apt install irqbalance -y
apt install prometheus-node-exporter -y
```

# Radius

```bash
apt install freeradius freeradius-utils libpam-google-authenticator -y
apt install iperf3 -y
apt install prometheus -y
```

# Remote Client

```bash
apt install openvpn -y
apt install iperf3 -y
```

# HQ-Client

```bash
apt install iperf3 -y
```

# BR-Client

```bash
apt install iperf3 -y
```

# WAN