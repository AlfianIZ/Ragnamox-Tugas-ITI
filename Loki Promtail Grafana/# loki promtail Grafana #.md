#  Loki, Promtail, Grafana #



## Loki

### 1. buat direktori konfigurasi

&#x20;   - ketik: 
```bash
sudo mkdir -p /etc/loki
```

### 2. download loki

&#x20;  - pindah direktori ke tmp: 
```bash
cd /tmp
```

&#x20;  - lalu jalankan command berikut satu persatu:
```bash
curl -s https://api.github.com/repos/grafana/loki/releases/latest | grep browser_download_url | grep linux-amd64 | cut -d '"' -f 4 | wget -qi -

unzip loki-linux-amd64.zip

sudo mv loki-linux-amd64 /usr/local/bin/loki

sudo chmod +x /usr/local/bin/loki
```

### 3. buat file konfigurasi

&#x20;  - ketik: 
```bash
sudo nano /etc/loki/loki-config.yaml
```
&#x20;  - isi loki-config.yaml dengan: 
```bash
auth_enabled: false

server:
  http_listen_port: 3100

common:
  instance_addr: 127.0.0.1
  path_prefix: /tmp/loki
  storage:
    filesystem:
      chunks_directory: /tmp/loki/chunks
      rules_directory: /tmp/loki/rules
  replication_factor: 1
  ring:
    kvstore:
      store: inmemory

schema_config:
  configs:
    - from: 2020-10-24
      store: boltdb-shipper
      object_store: filesystem
      schema: v11
      index:
        prefix: index_
        period: 24h

# Tambahkan bagian ini untuk mematikan fitur yang menyebabkan error
limits_config:
  allow_structured_metadata: false
```


### 4. cek status loki
```bash
sudo systemctl status loki
```
**Pastikan loki active (running)**

_____

## Promtail

### 1. buat direktori konfigurasi
```bash
sudo mkdir -p /etc/promtail
```

### 2. unduh file zip promtail (versi 2.9.10)
```bash
wget https://github.com/grafana/loki/releases/download/v2.9.10/promtail-linux-amd64.zip
```

&#x20;  - setelah mengunduh, jalankan command berikut satu persatu:
```bash
unzip promtail-linux-amd64.zip

sudo mv promtail-linux-amd64 /usr/local/bin/promtail

sudo chmod +x /usr/local/bin/promtail
```

&#x20;  - verifikasi: 
```bash
promtail --version
```
**Jika berhasil terinstal akan muncul tulisan versi promtail**

### 3. konfigurasi

&#x20;  - edit file `promtail-config.yaml`:
```bash
sudo nano /etc/promtail/promtail-config.yaml
```
&#x20;  - isi dengan:
```bash
server:
  http_listen_port: 9080
  grpc_listen_port: 0

clients:
  - url: http://localhost:3100/loki/api/v1/push

scrape_configs:
  - job_name: suricata
    static_configs:
    - targets: [localhost]
      labels:
        job: suricata
        __path__: /var/log/suricata/eve.json

  - job_name: nginx
    static_configs:
    - targets: [localhost]
      labels:
        job: nginx
        __path__: /var/log/nginx/*.log

  - job_name: mail
    static_configs:
    - targets: [localhost]
      labels:
        job: mail_server
        __path__: /var/log/mail.log
``` 

### 4. buat service system
```bash
sudo nano /etc/systemd/system/promtail.service
```
&#x20;  - isi dengan:

```bash
[Unit]
Description=Promtail service
After=network.target

[Service]
Type=simple
User=root
ExecStart=/usr/local/bin/promtail -config.file=/etc/promtail/promtail-config.yaml
Restart=on-failure

[Install]
WantedBy=multi-user.target
```
&#x20;   - jalankan service: 
```bash
sudo systemctl daemon-reload 
```
lalu:
```bash
sudo systemctl enable --now promtail
```


### 5. cek status
```bash
sudo systemctl status promtail
```

```bash
tail -f /var/log/suricata/eve.json
```

------

## Grafana



### 1. Tambah repositori

&#x20;  - jalankan command berikut satu persatu:
```bash
sudo apt-get install -y apt-transport-https software-properties-common wget

sudo wget -q -O /usr/share/keyrings/grafana.key https://apt.grafana.com/gpg.key

echo "deb \[signed-by=/usr/share/keyrings/grafana.key] https://apt.grafana.com stable main" | sudo tee /etc/apt/sources.list.d/grafana.list
```


### 2. Update daftar paket dan install Grafana
```bash
sudo apt-get update

sudo apt-get install grafana -y
```



### 3. Jalankan Grafana server
```bash
sudo systemctl daemon-reload

sudo systemctl enable --now grafana-server
```


### 4. Buka port 3000 di firewall
```bash
sudo ufw allow 3000/tcp

sudo ufw reload
```

### 5. Buka Grafana di browser
```
http://IP_UBUNTU_SERVER:3000
```