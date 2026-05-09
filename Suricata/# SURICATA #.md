# \# SURICATA



1. Instalasi suricata

&#x20;  - update daftar paket, ketik: sudo apt update

&#x20;  - instal suricata dari repositori ubuntu, ketik: sudo apt install suricata -y

&#x20;  - verifikasi instalasi, ketik: suricata -V



2\. Identifikasi interface jaringan

&#x20;  - ketik: ip addr

&#x20;  - cari nama interface yang memegang IP, missal ens18



3\. Konfigurasi suricata.yaml

&#x20;  - ketik: sudo nano /etc/suricata/suricata.yaml lalu cari bagian "vars:" lalu sesuaikan HOME\_NET contoh: 

vars:

&#x20; address-groups:

&#x20;   HOME\_NET: "\[192.168.1.0/24]" # Sesuaikan dengan subnet Anda

&#x20;   EXTERNAL\_NET: "!$HOME\_NET"

&#x20;  - kemudian cari bagian af-packet. contoh:

&#x20;    af-packet:

&#x20;      - interface: ens18 # Ganti dengan nama interface Anda

&#x20;        cluster-id: 99

&#x20;        cluster-type: cluster\_flow

&#x20;        defrag: yes



4\. Manajemen rule

&#x20;  - ketik: sudo suricata-update



5\. Menjalankan pengujian

&#x20;  - cek validasi konfigurasi, ketik: sudo suricata -T -c /etc/suricata/suricata.yaml -v pastikan muncuk pesan successfully

&#x20;  - restart suricata, ketik: sudo systemctl restart suricata

&#x20;  - cek status suricata, ketik: sudo systemctl status suricata

&#x20;  - uji deteksi, ketik: sudo tail -f /var/log/suricata/fast.log



6\. Membuat custom rule untuk mendeteksi ping

&#x20;  - buka file rule local, ketik: sudo nano /var/lib/suricata/rules/local.rules

&#x20;  - masukkan aturan berikut: alert icmp any any -> $HOME\_NET any (msg:"isi message terserah(contoh:Ada yang PING server alfi.lab!)"; sid:1000001; rev:1;)

&#x20;  - buka: sudo nano /etc/suricata/suricata.yaml

&#x20;  - cari bagian "rule-files" tambahkan: - local.rules di bawah - suricata.rules

&#x20;  - restart suricata: sudo systemctl restart suricata

&#x20;  - pantau log: sudo tail -f /var/log/suricata/fast.log

