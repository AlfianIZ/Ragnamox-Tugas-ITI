# Instalasi & Konfigurasi Suricata (Rev)
### 1. Instalasi Paket Utama
```
sudo apt update
```
### Instal Suricata langsung dari repositori resmi
```
sudo apt install suricata -y
```
#### 2. Identifikasi Interface Jaringan Aktif
> Jalankan perintah pengecekan IP:
```
ip addr
```
> Cari nama interface yang memegang IP lokal servermu (misalnya: ens18 atau eth0). Catat nama interface ini untuk langkah berikutnya.
### 3. Konfigurasi File Utama suricata.yaml
```
sudo nano /etc/suricata/suricata.yaml
```
#### a. cari bagian vars kemudian comment bagian lain , dan pakai "any"
```yaml
vars:
  address-groups:
    HOME_NET: "any"
    EXTERNAL_NET: "any"
```
#### b. Cari bagian af-packet: dan sesuaikan nama interface dengan hasil Langkah 2:
```
af-packet:
  - interface: ens18  # Sesuaikan dengan interface VM kamu (misal ens18)
    cluster-id: 99
    cluster-type: cluster_flow
    defrag: yes
```
#### c. Pastikan jalur utama berkas aturan mengarah ke direktori sistem /etc/:
```
default-rule-path: /etc/suricata/rules
```
#### d. Matikan (beri tanda #) pada berkas suricata.rules (rule bawaan)
> rule bawaan tidak disertakan untuk efisiensi
```
rule-files:
  # - suricata.rules
  - local.rules
```
### 4: Pembuatan custom rule (local.rules)
#### a. Buat berkas baru langsung di bawah direktori konvensional /etc/:
```
sudo nano /etc/suricata/rules/local.rules
```
#### b. Masukkan tiga aturan deteksi ini ke dalam berkas:
```
alert icmp any any -> any any (msg:"ICMP Ping Terdeteksi"; sid:1000001; rev:1;)
alert tcp any any -> any 22 (msg:"Percobaan Akses SSH Terdeteksi"; flow:to_server,established; sid:1000002; rev:1;)
alert tcp any any -> any 80 (msg:"HTTP TERAKSES"; sid:1000003; rev:1;)
```
> Simpan dan keluar (Ctrl + O, Enter, Ctrl + X).
### 5. Pengujian Validasi Konfigurasi & Memulai Layanan
#### a. Lakukan uji validasi sintaks secara menyeluruh untuk memastikan konfigurasi lolos uji beban:
```
sudo suricata -T -c /etc/suricata/suricata.yaml -v
```
> Pastikan di akhir log muncul status sukses: Configuration provided was successfully loaded.
#### b. Jalankan dan aktifkan layanan Suricata agar otomatis menyala saat reboot:
```
sudo systemctl restart suricata
sudo systemctl enable suricata
```
