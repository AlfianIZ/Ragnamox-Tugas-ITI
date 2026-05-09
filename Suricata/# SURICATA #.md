# \# SURICATA

## Note: Pada langkah ini, Suricata diinstal di ubuntu server

### 1. Instalasi suricata

&#x20;  - update daftar paket, ketik: 
```bash
sudo apt update
```

&#x20;  - instal suricata dari repositori ubuntu, ketik: 
```bash
sudo apt install suricata -y
```

&#x20;  - verifikasi instalasi, ketik: 
```bash
suricata -V
```
jika berhasil maka akan muncul versi suricata.

2\. Identifikasi interface jaringan

&#x20;  - ketik: 
```bash
ip addr
```
&#x20;  - cari nama interface yang memegang IP, misal ens18, contoh: 



### 3. Konfigurasi suricata.yaml

&#x20;  - ketik: 
```bash
sudo nano /etc/suricata/suricata.yaml
```
lalu cari bagian `vars:` cari `HOME\_NET` dan `EXTERNAL\_NET`
contoh:
```bash
vars:
address-groups:
   HOME_NET: "[192.168.1.0/24]" # Sesuaikan dengan subnet kamu
   EXTERNAL_NET: "!$HOME_NET"
```

&#x20;  - kemudian cari bagian af-packet. contoh:

```bash
af-packet:
- interface: ens18 # Ganti dengan nama interface kamu 
     cluster-id: 99
     cluster-type: cluster_flow
     defrag: yes
```
Note: untuk nama interface, ketik `ip addr` pada ubuntu server, cari baris `inet IP_UBUNTU_SERVER/24` pada baris tersebut bagian paling belakang adalah nama interface contoh:

4\. memperbarui ruleset suricata, ketik: 
```bash
sudo suricata-update
```

5\. Menjalankan pengujian

- cek validasi konfigurasi, ketik: 
```bash
sudo suricata -T -c /etc/suricata/suricata.yaml -v 
```
pastikan muncuk pesan successfully

- restart suricata, ketik: 
```bash
sudo systemctl restart suricata
```

- cek status suricata, ketik: 
```bash
sudo systemctl status suricata
```
- uji deteksi, ketik: 
```bash
sudo tail -f /var/log/suricata/fast.log
```


6\. Membuat custom rule untuk mendeteksi ping

- buka file rule local, ketik: 
```bash
sudo nano /var/lib/suricata/rules/local.rules
```

- masukkan aturan berikut: 
```bash
alert icmp any any -> $HOME_NET any (msg:"isi message terserah(contoh:Ada yang PING server alfi.lab!)"; sid:1000001; rev:1;)
```

- buka: 
```bash
sudo nano /etc/suricata/suricata.yaml
```
- cari bagian `rule-files` tambahkan di bawah `- suricata.rules`: 
```bash
- local.rules 
```
- restart suricata: 
```bash
sudo systemctl restart suricata
```
- pantau log: 
```bash
sudo tail -f /var/log/suricata/fast.log
```
