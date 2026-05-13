# Ini Adalah Langkah-langkah Instalasi Web Server, Database Server, DNS Server, Mail Server

## Sebelum melakukan langkah-langkah dibawah pastikan update terlebih dahulu:
```bash
sudo apt update && apt upgrade -y
```

### 1. WEB SERVER

&#x20;  - INSTALL NGINX: 
```bash 
sudo apt install nginx -y
```

&#x20;  - cek status : 
```bash
sudo systemctl status nginx
```

&#x20;  - cek status firewall : 
```bash
sudo ufw status
```
&#x20;  - jika nyala matikan firewall dengan command : 
```bash
sudo ufw disable
```
&#x20;  - pengujian awal via browser: di laptop/pc/hp yang satu jaringan ketik : http://[IP_Ubuntu_Server] contoh:http://192.168.1.101 

----

### 2. Database Server:

&#x20;  - Install mysql-server: 
```bash
sudo apt install mysql-server -y
```
&#x20;  - verifikasi: 
```bash
sudo systemctl status mysql
```
**Pastikan ada tulisan active (running) berwarna hijau**
**Jika tidak hijau, jalankan : `sudo systemctl start mysql-server`**
&#x20;  - akses database: 
```bash
sudo mysql -u root -p
```

-------


### 3. DNS Server:

&#x20;  - Instal paket Bind9: 
```bash
sudo apt install bind9 bind9utils bind9-doc -y
```
&#x20;  - menentukan nama domain, dengan mengedit `named.conf.local`: 
```bash
sudo nano /etc/bind/named.conf.local
```
&#x20;           isinya:
```bash
zone "nama domain kamu (contoh: alfi.lab)" {

type master;

file "/etc/bind/db.nama domain kamu (contoh: db.alfi.lab)";

};
```
&#x20;  - Membuat file konfigurasi zone:

1. salin file template bawaan agar tidak mengetik dari nol:
```bash
sudo cp /etc/bind/db.local /etc/bind/db.namadomain
```
2. Edit file tersebut:
```bash
sudo nano /etc/bind/db.namadomain
```
3. Ubah isinya menjadi seperti berikut:
```bash
;
; BIND data file for namadomain
;
$TTL    604800
@       IN      SOA     namadomain. root.namadomain. (
                     2         ; Serial
                604800         ; Refresh

                 86400         ; Retry

               2419200         ; Expire

                604800 )       ; Negative Cache TTL
;
@       IN      NS      namadomain.
@       IN      A       IP_UBUNTU_SERVER
@       IN      MX 10   mail.namadomain
www     IN      A       IP_UBUNTU_SERVER

```

&#x20;  - Cek konfigurasi dan restart

1. Cek: 
```bash
sudo named-checkconf
```
**Jika tidak muncul pesan apa pun, berarti aman**

2. tambahkan local nameserver di ubuntu /etc/resolv.conf: 
```bash
nano /etc/resolv.conf
```
**rubah 172.0.0.53 menjadi 172.0.0.53 dan tambahkan 1.1.1.1**
```
nameserver 172.0.0.1
nameserver 1.1.1.1
```
3. Restart layanan Bind9: 
```bash
sudo systemctl restart bind9
```
**Jika tidak muncul pesan apa pun, berarti aman**

&#x20;  - cara pengujian:

1. Ubah DNS di PC/Laptop:

     Agar laptop Anda bisa mengenali nama domain yang dibuat, ubah pengaturan DNS di laptop tersebut menjadi alamat IP Ubuntu Server.
     
     Caranya: buka settings -> network & internet -> cari DNS server assignment -> edit -> ubah menjadi manual -> pada prefered dns masukkan IP_UBUNTU_SERVER, pada alternate DNS masukkan 8.8.8.8 -> lalu save.

2. Gunakan Perintah nslookup:

&#x20;                - Di CMD (windows) atau terminal linux, ketik: 
```bash
nslookup namadomain
```
&#x20;                Hasil yang Benar: Akan muncul jawaban bahwa nama domain berada di alamat IP_UBUNTU_SERVER

3.  Coba buka browser ketikkan: http://namadomain

4. Jika tidak bisa maka(untuk windows):
   - flush dulu dns cache, di cmd ketik:
   ```
   ipconfig /flushdns
   ```
   - buka notepad sebagai administrator -> file -> open -> dik C -> Windows -> System32 -> drivers -> etc -> ubah Type file menjadi All Files -> buka file yang bernama host, lalu edit file hosts, letakkan baris ini dibawah sendiri, lalu save: 
   ```
   [IP_UBUNTU_SERVER]    namadomainkamu
   [IP_UBUNTU_SERVER]    www.namadomainkamu
   ```
   - buka lagi browser dan ketik nama domain (contoh: http://alfi.lab).
   

----

### 4. Mail Server

&#x20;  - instalasi : 
```bash
sudo apt install postfix -y
```
&#x20;  - lalu ditengah proses akan muncul layar ungu, pada General type of mail configuration pilih Internet Site (untuk memilihnya gunakan tombol panah dan tekan tab untuk mengarahkan ke tombol ok), lalu enter. selanjutnya pada System mail name, masukkan nama domain kamu (contoh: alfi.lab), lalu tab, enter.

&#x20;  - Konfigurasi Postfix:
1. edit file konfigurasi: 
```bash
sudo nano /etc/postfix/main.cf
```

2. lalu cari baris myhostname, ubah menjadi: 
```bash
myhostname = mail.namaDomain (mail.alfi.lab)
```

3. cari baris mydestination, ubah menjadi: 
```bash
mydestination = $myhostname, nama domain (contoh: alfi.lab), lacalhost.local domain, localhost
```

4. tambahkan pada baris paling bawah: 
```bash
home_mailbox = Maildir/
```
 
5. OPSIONAL, tambahkan subnet pada bagian mynetworks: 
```
mynetworks = 127.0.0.0/8 [::ffff:127.0.0.0]/104 [::1]/128, subnet (contoh:192.168.1.0/24)
```

6. simpan konfigurasinya lalu restart postfix: 
```bash
sudo systemctl restart postfix
```
7. tes kirim email ke root
```
echo "Setup Infrastructure Selesai" | mail -s "Final Test" root
```
8. baca email (cek inbox)
```
mail
```
> untuk keluar ketik ctrl+z


## Deploy Webapp (html tok)
### Note: Disini aku pake kode html tugas praktikum pemweb yang kalkulator.


### 1. Buka Kunci Folder Web (Di Terminal Ubuntu Server)

&#x20;   ketik :
```bash
sudo chmod -R $USER:$USER /var/www/html/
```

```bash
sudo chmod -R 755 /var/www/html/
```


### 2. Menghapus halaman bawaan nginx, ketik:
```bash
sudo rm /var/www/html/index.nginx-debian.html
```

### 3. Pindah file html dari laptop ke ubuntu server (untuk cara mudah, pastikan nama file index.html)

buka cmd lalu ketik: 
```bash
scp -r nama_folder_web_anda namaubuntuserver@ipubuntuserver:/var/www/html 
#(contoh: scp "D:\semester 4\praktikumPemweb\code\bab4\index.html" alpi@192.168.1.101:/var/www/html/)
# Jika yang dipindah lebih dari satu file gunakan '-r' untuk menyalin satu folder beserta isinya, jika hanya satu file maka tidak usah menggunakan '-r'.
```

ketik di ubuntu server:
```bash
sudo chmod 644 /var/www/html/index.html
```


### 4. Buka browser

&#x20;  - ketik: http://namadomain contoh: http://alpi.lab
jika berhasil maka akan muncul halaman web yang sudah di upload.
