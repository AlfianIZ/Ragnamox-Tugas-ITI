TUGAS PROXMOX ITI



1\. WEB SERVER:

&#x20;  - INSTALL NGINX: sudo apt install nginx -y

&#x20;  - cek : sudo systemctl status nginx

&#x20;  - cek daftar aplikasi di firewall : sudo ufw app list

&#x20;  - akses penuh firewall untuk nginx : sudo ufw allow 'Nginx full'

&#x20;  - pengujian awal via browser: di laptop/pc/hp yang satu jaringan ketik http://ipubuntuserver contoh:http://192.168.1.101 http://



2\. Database Server:

&#x20;  - Install mariaDB: sudo apt install mariadb-server -y

&#x20;  - verifikasi: sudo systemctl status mariadb

&#x20;  - mengamankan database: sudo mysql\_secure\_installation

&#x20;       Anda akan ditanya beberapa hal, ikuti saran ini:



&#x09;	Enter current password for root: Langsung tekan Enter (karena baru instal, passwordnya masih kosong).



&#x09;	Switch to unix\_socket authentication? Tekan n.



&#x09;	Change the root password? Tekan y, lalu masukkan password baru yang kuat untuk database Anda. (Catat password ini!).

&#x09;	(PW: IsZuAl5676)

&#x09;	Remove anonymous users? Tekan y.



&#x09;	Disallow root login remotely? Tekan y.



&#x09;	Remove test database and access to it? Tekan y.



&#x09;	Reload privilege tables now? Tekan y.

&#x20;  - akses database: sudo mysql -u root -p



3\. DNS Server:

&#x20;  - Instal paket Bind9: sudo apt install bind9 bind9utils bind9-doc -y

&#x20;  - menentukan nama domain: sudo nano /etc/bind/named.conf.local

&#x20;           isinya:

zone "tugas.lab" {

&#x20;   type master;

&#x20;   file "/etc/bind/db.tugas.lab";

};

&#x20;  - Membuat file konfigurasi zone:

&#x20;           1. salin file template bawaan agar tidak mengetik dari nol:

&#x20;              - sudo cp /etc/bind/db.local /etc/bind/db.tugas.lab

&#x20;           2. Edit file tersebut:

&#x20;              - sudo nano /etc/bind/db.tugas.lab

&#x20;           3. Ubah isinya menjadi seperti ini (Perhatikan titik di belakang nama domain!):

;

; BIND data file for tugas.lab

;

$TTL    604800

@       IN      SOA     tugas.lab. root.tugas.lab. (

&#x20;                     2         ; Serial

&#x20;                604800         ; Refresh

&#x20;                 86400         ; Retry

&#x20;               2419200         ; Expire

&#x20;                604800 )       ; Negative Cache TTL

;

@       IN      NS      tugas.lab.

@       IN      A       192.168.1.101

www     IN      A       192.168.1.101



&#x20;  - Cek konfigurasi dan restart

&#x20;              1. Jalankan perintah cek: sudo named-checkconf

&#x20;                 (Jika tidak muncul pesan apa pun, berarti aman).

&#x20;              2. Buka firewall untuk DNS (Port 53): sudo ufw allow bind9

&#x20;              3. Restart layanan Bind9: sudo systemctl restart bind9

&#x20;  - cara pengujian:

&#x20;              1. Ubah DNS di PC/Laptop Anda:

&#x20;                 Agar laptop Anda bisa mengenali tugas.lab, Anda harus mengubah pengaturan DNS di laptop tersebut menjadi alamat IP Ubuntu Server Anda (192.168.1.101).

&#x20;              2. Gunakan Perintah nslookup:

&#x20;                 Di terminal laptop Anda (CMD atau Linux Terminal), ketik: nslookup tugas.lab

&#x20;                 Hasil yang Benar: Akan muncul jawaban bahwa tugas.lab berada di alamat 192.168.1.101.



4\. Mail Server

&#x20;  - instalasi : sudo apt install postfix dovecot-core dovecot-imapd dovecot-pop3d -y

&#x20;  - lalu ditengah proses akan muncul layar ungu, pada General type of mail configuration pilih Internet Site (untuk memilihnya gunakan tombol panah dan tekan tab untuk mengarahkan ke tombol ok), lalu enter. selanjutnya pada System mail name, masukkan nama domain kamu, misal: alfi.lab, lalu tab, enter.

&#x20;  - Konfigurasi Postfix:

&#x20;             1. edit file konfigurasi: sudo nano /etc/postfix/main.cf

&#x20;             2. lalu cari baris myhostname, ubah menjadi: myhostname = mail.namaDomainKamu (mail.alfi.lab)

&#x20;             3. cari baris mydestination, ubah menjadi: mydestination = $myhostname, nama domain kamu (alfi.lab), lacalhost.local domain, localhost

&#x20;             4. tambahkan : home\_mailbox = Maildir/ pada baris paling bawah.

&#x20;             5. OPSIONAL: tambahkan subnet pada bagian mynetworks: mynetworks = 127.0.0.0/8 \[::ffff:127.0.0.0]/104 \[::1]/128, subnet (192.168.1.0/24)

&#x20;             6. simpan konfigurasinya lalu restart postfix: sudo systemctl restart postfix



&#x20;  - Konfigurasi Dovecot :

&#x20;             1. konfigurasi lokasi email: sudo nano /etc/dovecot/conf.d/10-mail.conf, cari baris mail\_location ubah menjadi: mail\_location = maildir:\~/Maildir lalu simpan dan keluar.

&#x20;             2. konfigurasi autentikasi: sudo nano /etc/dovecot/conf.d/10-auth.conf, cari baris disable\_plaintext\_auth hapus '#' yang ada di depannya dan ubah menjadi: disable\_plaintext\_auth = no simpan dan keluar.

&#x20;             3. restart dovecot; sudo systemctl restart dovecot

&#x20;  - Membuat pengguna:

&#x20;             1. buat pengguna pertama (admin): sudo adduser admin, buat password dan biarkan pengisian identitas kosong dengan menekan enter.

&#x20;             2. buat pengguna kedua (user1): sudo adduser user1, buat password dan biarkan identitas kosong

&#x20;  - allow firewall: sudo ufw allow Postfix, sudo ufw allow "Dovecot IMAP", sudo ufw allow "Dovecot POP3", sudo ufw enable (NOTE: pastikan OpenSSH dan ssh sudah diallow baru enable).

&#x20;  - Pengujian internal:

&#x20;             1. install telnet: sudo apt install telnet -y

&#x20;             2. tes mengirim email ke port 25(postfix): telnet localhost 25 (Jika berhasil terhubung, akan muncul tulisan Connected to localhost. Escape character is...) ketik: ehlo localhost lalu enter, ketik: quit

&#x20;             3. tes Dovecot POP3: telnet localhost 110 (jika berhasil, akan muncul: +OK Dovecot ready), ketik: quit.



&#x20;  - simulasi pengiriman email antar user melalui terminal:

&#x20;             1. simulasi pengiriman email(SMTP):

&#x20;                - hubungkan dulu, ketik: telnet localhost 25

&#x20;                - Sapa server, ketik: HELO namadomain(cth: alfi.lab)

&#x20;                - isi identitas pengirim, ketik: mail from: admin@alfi.lab

&#x20;                - isi identitas penerima, ketik: rcpt to: user1@alfi.lab

&#x20;                - ketik pesan dengan: data, lalu enter, ketik isi pesan contoh: Subject: Test Email Praktikum

&#x20;                                                                                         Halo user1, ini adalah email percobaan dari admin.

&#x20;                                                                                         .

&#x20;                                                                                         (jangan lupa titik di baris terakhir untuk mengakhiri pesan, lalu enter).

&#x20;                -ketik: quit untuk keluar.

&#x20;             2. Simulasi pengambilan(POP3):

&#x20;                - hubungkan dulu, ketik: telnet localhost 110

&#x20;                - login user, ketik: user user1, lalu masukkan password ketik: pass passwordKamu.

&#x20;                - lihat daftar email, ketik: list (jika muncul angka 1 dan ukuran file, maka email berhasil masuk)

&#x20;                - baca email, ketik: retr 1

&#x20;                - keluar: quit.



# \# Deploy Webapp

## (html js, menggunakan nginx)



1. Buka Kunci Folder Web (Di Terminal Ubuntu Server)

&#x20;   ketik :

&#x20;   - sudo chmod -R $USER:$USER /var/www/html/ lalu enter

&#x20;   - sudo chmod -R 755 /var/www/html/ lalu enter



2\. Menghapus halaman bawaan nginx

&#x20;  ketik:

&#x20;  - sudo rm /var/www/html/index.nginx-debian.html



3\. Pindah file html dari laptop ke ubuntu server (untuk cara mudah, pastikan nama file index.html)

&#x20;  - buka cmd lalu ketik: scp -r nama\_folder\_web\_anda namaubuntuserver@ipubuntuserver:/var/www/html (contoh: scp "D:\\semester 4\\praktikumPemweb\\code\\bab4\\index.html" alpi@192.168.1.101:/var/www/html/)

&#x20;  - ketik: sudo chmod 644 /var/www/html/index.html



4\. Buka browser

&#x20;  - ketik: http://namadomain contoh: http://alpi.lab

