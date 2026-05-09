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

&#x20;  - cek daftar aplikasi di firewall : 
```bash
sudo ufw app list
```
&#x20;  - Izinkan akses penuh firewall untuk nginx : 
```bash
sudo ufw allow 'Nginx full'
```
&#x20;  - pengujian awal via browser: di laptop/pc/hp yang satu jaringan ketik : http://[IP_Ubuntu_Server] contoh:http://192.168.1.101 

----

### 2. Database Server:

&#x20;  - Install mariaDB: 
```bash
sudo apt install mariadb-server -y
```
&#x20;  - verifikasi: 
```bash
sudo systemctl status mariadb
```
**Pastikan ada tulisan active (running) berwarna hijau**
**Jika tidak hijau, jalankan : `sudo systemctl start mariadb`**

&#x20;  - Jalankan command ini untuk membuat password database:
```bash
sudo mysql_secure_installation
```
&#x20;       Nanti akan ditanya beberapa hal, ikuti langkah berikut:
```bash
Enter current password for root: Langsung tekan Enter (karena baru instal, passwordnya masih kosong).

Switch to unix\_socket authentication? Tekan n.
Change the root password? Tekan y, lalu masukkan password (Catat passwordnya).
Remove anonymous users? Tekan y.

Disallow root login remotely? Tekan y.

Remove test database and access to it? Tekan y.

Reload privilege tables now? Tekan y.
``` 
&#x20;  - akses database: 
```bash
sudo mysql -uroot -p
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
www     IN      A       IP_UBUNTU_SERVER

```

&#x20;  - Cek konfigurasi dan restart

1. Cek: 
```bash
sudo named-checkconf
```
**Jika tidak muncul pesan apa pun, berarti aman**

2. Buka firewall untuk DNS (Port 53): 
```bash
sudo ufw allow bind9
```
**Jika tidak muncul pesan apa pun, berarti aman**

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
sudo apt install postfix dovecot-core dovecot-imapd dovecot-pop3d -y
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

&#x20;  - Konfigurasi Dovecot :

&#x20;             1. konfigurasi lokasi email: 
```
sudo nano /etc/dovecot/conf.d/10-mail.conf
```
cari baris `mail_location` lalu ubah menjadi:
```bash
mail_location = maildir:~/Maildir
```

2. konfigurasi autentikasi: 
```bash
sudo nano /etc/dovecot/conf.d/10-auth.conf
```
cari baris `disable_plaintext_auth` hapus '#' yang ada di depannya dan ubah menjadi: 
```bash
disable_plaintext_auth = no
```
3. restart dovecot:
```bash
sudo systemctl restart dovecot
```
&#x20;  - Membuat pengguna:

1. buat pengguna pertama (admin): 
```bash
sudo adduser admin
```
buat password dan biarkan pengisian identitas kosong dengan menekan enter.

2. buat pengguna kedua (user1):
```bash
sudo adduser user1
```
buat password dan biarkan identitas kosong.

&#x20;  - allow firewall: 
```bash
sudo ufw allow Postfix
```
```bash
sudo ufw allow "Dovecot IMAP"
```
```bash
sudo ufw allow "Dovecot POP3"
```
&#x20;  - jika belum mengaktifkan UFW maka:
```bash
sudo ufw allow OpenSSH
```
atau:
```bash
sudo ufw allow ssh
```
```bash
sudo ufw enable
```


&#x20;  - Pengujian internal:

1. install telnet: 
```bash
sudo apt install telnet -y
```

2. tes postfix, ketik: 
```bash
telnet localhost 25 
```
(Jika berhasil terhubung, akan muncul tulisan Connected to localhost. Escape character is...) 
jika berhasil ketik: 
```bash
ehlo localhost 
```
lalu enter, 
ketik: 
```bash
quit
```

3. tes Dovecot POP3: 
```bash
telnet localhost 110 
```
(jika berhasil, akan muncul: +OK Dovecot ready), 
ketik: 
```bash
quit.
```


&#x20;  - simulasi pengiriman email antar user melalui terminal:

1. pengiriman email(SMTP):

- hubungkan dulu, ketik: 
```bash
telnet localhost 25
```

- Sapa server, ketik: 
```bash
HELO namadomain(cth: alfi.lab)
```

- isi identitas pengirim, ketik: 
```bash
mail from: admin@namadomain (contoh: admin@alfi.lab)
```

- isi identitas penerima, ketik: 
```bash
rcpt to: user1@namadomain (contoh user1@alfi.lab)
```

- ketik pesan dengan mengetikkan: 
```bash
data
```
lalu enter, ketik isi pesan contoh: 
```bash
Subject: Test Email
Halo user1, ini adalah email percobaan dari admin.
.
```
(jangan lupa titik di baris terakhir untuk mengakhiri pesan, lalu enter).

- ketik: 
```bash
quit
```


2. Simulasi penerima(POP3):

&#x20;                - hubungkan dulu, ketik: 
```bash
telnet localhost 110
```
&#x20;                - login user, ketik: 
```bash
user user1
```
lalu masukkan password ketik: 
```bash
pass masukkanPassword
```
&#x20;                - lihat daftar email, ketik: 
```bash
list #(jika muncul angka 1 dan ukuran file, maka email berhasil masuk)
```
&#x20;                - baca email, ketik: 
```bash
retr 1
```
&#x20;                - keluar: 
```bash
quit.
```


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

