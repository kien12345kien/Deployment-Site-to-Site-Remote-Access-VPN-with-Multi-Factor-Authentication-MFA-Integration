Guide cài Prometheus, Grafana để monitoring Server dùng Node Exporter trên RHEL 9.7

# Cài Prometheus

## Bước 1: Update và Sửa lại DNS

|  |
| --- |
| sudo dnf update -y  sudo nmcli connection modify ens160 ipv4.dns "8.8.8.8 8.8.4.4"  sudo nmcli connection reload  sudo nmcli device reapply ens160  cat /etc/resolv.conf |

## Bước 2: Tải prometheus

* Download prometheus binary file
  + Vào trang <https://prometheus.io/download/>

![](data:image/png;base64...)

* + Chúng ta sẽ download bản LTS cho linux

![](data:image/png;base64...)

* + - Click chuột phải để copy link address
    - Để download chạy (nên download trong folder /opt)

|  |
| --- |
| sudo wget <https://github.com/prometheus/prometheus/releases/download/v3.5.1/prometheus-3.5.1.linux-amd64.tar.gz> |

* + - Kiểm tra checksum

|  |
| --- |
| sha256sum prometheus-3.5.1.linux-amd64.tar.gz |

* Giải nén binary file

|  |
| --- |
| tar -xvf prometheus-3.5.1.linux-amd64.tar.gz |

## Bước 3: Cấu hình prometheus

* Tạo user prometheus

|  |
| --- |
| sudo useradd --no-create-home --shell /bin/false prometheus  cat /etc/passwd |

* Tạo folder và cấp quyền truy cập cho prometheus user

|  |
| --- |
| sudo mkdir /etc/prometheus  sudo mkdir /var/lib/prometheus  sudo chown prometheus:prometheus /etc/prometheus  sudo chown prometheus:prometheus /var/lib/prometheus |

* Copy binary file của prometheus vào /usr/local/bin/ và update quyền sở hữu file binary cho user prometheus

|  |
| --- |
| sudo cp -v /opt/prometheus-3.5.1.linux-amd64/prometheus /usr/local/bin  sudo cp -v /opt/prometheus-3.5.1.linux-amd64/promtool /usr/local/bin  sudo chown prometheus:prometheus /usr/local/bin/prometheus  sudo chown prometheus:prometheus /usr/local/bin/promtool |

* Thêm đường dẫn của binary file vào PATH environment variable

|  |
| --- |
| sudo vi ~/.bashrc  Thêm dòng “export PATH="${PATH}":/usr/local/bin/” ở cuối cùng  source ~/.bashrc |

* Kiểm tra prometheus version

|  |
| --- |
| prometheus --version  promtool --version |

* Copy config **Prometheus configuration file** prometheus.yml vào folder /etc/prometheus/

|  |
| --- |
| **cp -v /opt/prometheus-3.5.1.linux-amd64/prometheus.yml /etc/prometheus/** |

* Tạo prometheus systemd file

|  |
| --- |
| sudo vi /etc/systemd/system/prometheus.service |

Với nội dung như sau:

|  |
| --- |
| [Unit]  Description=Prometheus  Wants=network-online.target  After=network-online.target  [Service]  User=prometheus  Group=prometheus  Type=simple  ExecStart=/usr/local/bin/prometheus \  --config.file=/etc/prometheus/prometheus.yml \  --storage.tsdb.path=/var/lib/prometheus \  --web.listen-address=0.0.0.0:9090  Restart=always  RestartSec=10s  [Install]  WantedBy=multi-user.target |

* Khởi chạy service

|  |
| --- |
| sudo systemctl daemon-reload (Nếu sửa file tạo service mới hoặc sửa file prometheus.service cần chạy lệnh này)  sudo systemctl start prometheus  sudo systemctl enable prometheus  sudo systemctl status prometheus |

* Mở rule nếu firewalld đang bật

|  |
| --- |
| sudo firewall-cmd --add-port=9090/tcp --permanent  sudo firewall-cmd --reload |

# Cài Grafana

* Import GPG key

|  |
| --- |
| wget -q -O gpg.key https://rpm.grafana.com/gpg.key  sudo rpm --import gpg.key |

* Tạo file /etc/yum.repos.d/grafana.repo với nội dung sau:

|  |
| --- |
| [grafana]  name=grafana  baseurl=https://rpm.grafana.com  repo\_gpgcheck=1  enabled=1  gpgcheck=1  gpgkey=https://rpm.grafana.com/gpg.key  sslverify=1  sslcacert=/etc/pki/tls/certs/ca-bundle.crt |

* Tải grafana

|  |
| --- |
| sudo dnf install Grafana |

* Khởi tạo grafana service

|  |
| --- |
| sudo systemctl status grafana-server  sudo systemctl enable grafana-server.service  sudo systemctl status grafana-server |

* Mở rule nếu firewalld đang bật

|  |
| --- |
| sudo firewall-cmd --add-port=3000/tcp --permanent  sudo firewall-cmd --reload |

* Truy cập vào http://<your\_ip\_server>:3000

Account mặc định ban đầu là admin/admin

## Cấu hình Prometheus là DataSource cho Grafana

* Chọn **DataSource** 🡪 **Add data source** 🡪 Chọn **Prometheus** 🡪 Điền URL của prometheus server

![](data:image/png;base64...)

![](data:image/png;base64...)

![](data:image/png;base64...)

![](data:image/png;base64...)

## Cài Node Exporter(cài trên máy bạn muốn monitor)

* Tải bộ cài trên Node Exporter trang <https://prometheus.io/download/> giống như prometheus

![](data:image/png;base64...)

|  |
| --- |
| sudo wget <https://github.com/prometheus/node_exporter/releases/download/v1.11.1/node_exporter-1.11.1.linux-amd64.tar.gz> |

* Giải nén bộ cài

|  |
| --- |
| sudo tar -xvf node\_exporter-1.11.1.linux-amd64.tar.gz |

* Tạo user node\_exporter

|  |
| --- |
| sudo useradd --no-create-home --shell /bin/false node\_exporter |

* Copy binary file của Node Exporter sang /usr/local/bin

|  |
| --- |
| cp -v /opt/node\_exporter-1.11.1.linux-amd64/node\_exporter /usr/local/bin |

* Tạo Node Exporter systemd service

|  |
| --- |
| sudo vi /etc/systemd/system/node\_exporter.service |

Với nội dung như sau:

|  |
| --- |
| [Unit]  Description=Node Exporter  Wants=network-online.target  After=network-online.target  [Service]  Type=simple  User=node\_exporter  Group=node\_exporter  ExecStart=/usr/local/bin/node\_exporter \  --collector.mountstats \  --collector.logind \  --collector.processes \  --collector.ntp \  --collector.systemd \  --collector.tcpstat \  --collector.wifi  Restart=always  RestartSec=10s  [Install]  WantedBy=multi-user.target |

* Khởi chạy service node\_exporter

|  |
| --- |
| sudo systemctl daemon-reload  sudo systemctl enable node\_exporter  sudo systemctl start node\_exporter  sudo systemctl status node\_exporter |

* Mở rule

|  |
| --- |
| sudo firewall-cmd --add-port=9100/tcp --permanent  sudo firewall-cmd --reload |

## Cấu hình prometheus để lấy metric từ Node Exporter

- Cấu hình thêm trong file /etc/prometheus/prometheus.yml trên host cài prometheus

|  |
| --- |
| - job\_name: "node-01"  static\_configs:  - targets: ["<ip\_monitor\_host>:9100"] |

* Restart prometheus service

|  |
| --- |
| sudo systemctl restart prometheus |

Trong ví dụ này chúng ta sẽ tạo alert thông báo khi service node-exporter trên máy monitor down

Có 3 bước để cấu hình Alert

* Define the condition that must be met before an alert rule fires
* Configure where the alert should be deliver
* Chọn Alert rules 🡪 New alert rule

![](data:image/png;base64...)

![](data:image/png;base64...)

![](data:image/png;base64...)

![](data:image/png;base64...)

![](data:image/png;base64...)

![](data:image/png;base64...)

* Tạo contact point

![](data:image/png;base64...)

* Vào Telegram tìm @BotFather, gõ /newbot,sau đặt tên cho bot, sẽ được gửi lại token

![](data:image/png;base64...)

![](data:image/png;base64...)
