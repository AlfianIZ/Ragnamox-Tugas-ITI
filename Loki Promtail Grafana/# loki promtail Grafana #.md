# \# loki promtail Grafana #



### \# loki

1. buat direktori konfigurasi 

&#x20;   - ketik: sudo mkdir -p /etc/loki



2\. download loki

&#x20;  - pindah direktori ke tmp: cd /tmp

&#x20;  - curl -s https://api.github.com/repos/grafana/loki/releases/latest | grep browser\_download\_url | grep linux-amd64 | cut -d '"' -f 4 | wget -qi -

&#x20;  - unzip loki-linux-amd64.zip

&#x20;  - sudo mv loki-linux-amd64 /usr/local/bin/loki

&#x20;  - sudo chmod +x /usr/local/bin/loki



3\. buat file konfigurasi

&#x20;  - ketik: sudo nano /etc/loki/loki-config.yaml

&#x20;  - isi loki-config.yaml dengan: 

auth\_enabled: false



server:

&#x20; http\_listen\_port: 3100



common:

&#x20; instance\_addr: 127.0.0.1

&#x20; path\_prefix: /tmp/loki

&#x20; storage:

&#x20;   filesystem:

&#x20;     chunks\_directory: /tmp/loki/chunks

&#x20;     rules\_directory: /tmp/loki/rules

&#x20; replication\_factor: 1

&#x20; ring:

&#x20;   kvstore:

&#x20;     store: inmemory



schema\_config:

&#x20; configs:

&#x20;   - from: 2020-10-24

&#x20;     store: boltdb-shipper

&#x20;     object\_store: filesystem

&#x20;     schema: v11

&#x20;     index:

&#x20;       prefix: index\_

&#x20;       period: 24h



4\. buat file service loki

&#x20;  - sudo nano /etc/systemd/system/loki.service

&#x20;  - isi dengan: 

\[Unit]

Description=Loki log aggregation system

After=network.target



\[Service]

ExecStart=/usr/local/bin/loki -config.file=/etc/loki/loki-config.yaml

Restart=always

User=root



\[Install]

WantedBy=multi-user.target



5\. jalankan service loki

&#x20;  - sudo systemctl daemon-reload

&#x20;  - sudo systemctl enable loki

&#x20;  - sudo systemctl start loki



6\. cek status loki

&#x20;  - sudo systemctl status loki



### \# promtail



1. buat direktori konfigurasi

&#x20;   - sudo mkdir -p /etc/promtail



2\. unduh file zip promtail (versi 2.9.10)

&#x20;  - wget https://github.com/grafana/loki/releases/download/v2.9.10/promtail-linux-amd64.zip



&#x20;  - unzip promtail-linux-amd64.zip

&#x20;  - sudo mv promtail-linux-amd64 /usr/local/bin/promtail

&#x20;  - sudo chmod +x /usr/local/bin/promtail

&#x20;  - verifikasi: promtail --version



3\. konfigurasi

&#x20;  - sudo nano /etc/promtail/promtail-config.yaml

&#x20;  - isi dengan:

server:

&#x20; http\_listen\_port: 9080

&#x20; grpc\_listen\_port: 0



clients:

&#x20; - url: http://localhost:3100/loki/api/v1/push



scrape\_configs:

&#x20; - job\_name: suricata

&#x20;   static\_configs:

&#x20;   - targets: \[localhost]

&#x20;     labels:

&#x20;       job: suricata

&#x20;       \_\_path\_\_: /var/log/suricata/eve.json



&#x20; - job\_name: nginx

&#x20;   static\_configs:

&#x20;   - targets: \[localhost]

&#x20;     labels:

&#x20;       job: nginx

&#x20;       \_\_path\_\_: /var/log/nginx/\*.log



&#x20; - job\_name: mail

&#x20;   static\_configs:

&#x20;   - targets: \[localhost]

&#x20;     labels:

&#x20;       job: mail\_server

&#x20;       \_\_path\_\_: /var/log/mail.log



4\. buat service system

&#x20;  - sudo nano /etc/systemd/system/promtail.service

&#x20;  - isi dengan:

\[Unit]

Description=Promtail service

After=network.target



\[Service]

Type=simple

User=root

ExecStart=/usr/local/bin/promtail -config.file=/etc/promtail/promtail-config.yaml

Restart=on-failure



\[Install]

WantedBy=multi-user.target

&#x20;   - jalankan service: sudo systemctl daemon-reload lalu sudo systemctl enable --now promtail



5\. cek status

&#x20;  - sudo systemctl status promtail

&#x20;  - tail -f /var/log/suricata/eve.json





### \# Grafana



1. tambah repositori

&#x20;  - sudo apt-get install -y apt-transport-https software-properties-common wget

&#x20;  - sudo wget -q -O /usr/share/keyrings/grafana.key https://apt.grafana.com/gpg.key

&#x20;  - echo "deb \[signed-by=/usr/share/keyrings/grafana.key] https://apt.grafana.com stable main" | sudo tee /etc/apt/sources.list.d/grafana.list



2\. update daftar paket dan install Grafana

&#x20;  - sudo apt-get update

&#x20;  - sudo apt-get install grafana -y



3\. jalankan Grafana server

&#x20;  - sudo systemctl daemon-reload

&#x20;  - sudo systemctl enable --now grafana-server



4\. buka port 3000 di firewall

&#x20;  - sudo ufw allow 3000/tcp

&#x20;  - sudo ufw reload

