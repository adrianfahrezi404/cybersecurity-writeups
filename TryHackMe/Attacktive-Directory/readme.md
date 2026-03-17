# TryHackMe Write-up: Attacktive Directory
**Author:** Adrian  
**Date:** 16 Maret 2026  
**Platform:** TryHackMe  

## Pendahuluan
Pada lab Attacktive Directory ini, fokus utama adalah melakukan eksploitasi pada lingkungan Active Directory yang mensimulasikan skenario serangan dunia nyata. Langkah-langkah mencakup enumerasi awal, *user enumeration* menggunakan Kerberos, AS-REP Roasting, hingga melakukan DCSync attack untuk mendapatkan akses *Domain Admin*.

---

## Task 1 & 2: Setup dan Enumerasi Awal
Target IP: `[IP TARGET]`
Domain Target: `spookysec.local`

Langkah pertama yang dilakukan adalah menambahkan IP target ke dalam file `/etc/hosts`. Selanjutnya, dilakukan pemindaian port menggunakan Nmap untuk mengidentifikasi layanan yang berjalan.

**Perintah Nmap:**
`[COMMAND NMAP]`

**Hasil Nmap:**
`[HASIL NMAP]`

---

## Task 3: Enumerasi User dengan Kerbrute
Karena tidak memiliki kredensial, enumerasi user dilakukan menggunakan **Kerbrute** dan *wordlist* yang telah disediakan untuk mencari *username* yang valid di *domain controller*.

**Perintah Kerbrute:**
`[COMMAND KERBRUTE]`

**Hasil Enumerasi:**
`[USER VALID]`

---

## Task 4: Eksploitasi AS-REP Roasting
Setelah mendapatkan user yang valid, tahap selanjutnya adalah mencari akun yang memiliki miskonfigurasi *"Do not require Kerberos preauthentication"*. Pengecekan dan pengambilan *hash* dilakukan menggunakan `GetNPUsers.py` dari **Impacket**.

**Perintah GetNPUsers:**
`[COMMAND IMPACKET]`

**Hasil Hash & Cracking:**
Hash yang didapat kemudian di-crack secara offline menggunakan Hashcat / John The Ripper.
`[COMMAND & SCREENSHOT CRACK PASSWORD]`

---

## Task 5: Eksplorasi SMB
Dengan kredensial yang berhasil di-crack, akses ke SMB *shares* dapat dilakukan menggunakan `smbclient` untuk mencari informasi sensitif lebih lanjut.

**Perintah smbclient:**
`[COMMAND SMBCLIENT]`

**Temuan di SMB:**
`[SCREENSHOT ISI FOLDER / BACKUP FILE]`

---

## Task 6: Ekstraksi NTDS.dit (Elevating Privileges)
Memanfaatkan kredensial akun *backup* yang ditemukan, serangan DCSync dilakukan menggunakan `secretsdump.py` dari Impacket untuk menarik seluruh *hash password* dari *domain controller*.

**Perintah secretsdump:**
`[COMMAND SECRETSDUMP]`

**Hasil Dump:**
`[SCREENSHOT HASH ADMINISTRATOR]`

---

## Task 7: Pass-the-Hash & Root
Tahap terakhir adalah melakukan *Pass-the-Hash* menggunakan *hash* Administrator yang didapat dari langkah sebelumnya. Akses langsung ke server dilakukan menggunakan `evil-winrm`.

**Perintah evil-winrm:**
`[COMMAND EVIL-WINRM]`

**Root Flag:**
`[SCREENSHOT ROOT.TXT / FLAG]`

---
## Kesimpulan
Jujur, menyelesaikan room Attacktive Directory ini rasanya seperti naik *rollercoaster*. Ini pengalaman yang lumayan bikin pusing tapi sekaligus membuka mata saya tentang seberapa rentannya lingkungan Active Directory kalau tidak dikonfigurasi dengan benar. 

Ternyata, menyerang AD itu beda jauh *vibe*-nya dibandingkan eksploitasi mesin *stand-alone*. Hal yang paling bikin merinding (dalam artian positif) adalah menyadari bahwa miskonfigurasi yang terlihat sepele—seperti membiarkan akun tanpa *Kerberos pre-authentication*—bisa jadi jalan tol buat *attacker* untuk menguasai seluruh domain. Lewat lab ini, saya jadi benar-benar paham alur serangannya, mulai dari sekadar menebak *username* pakai Kerbrute, narik *hash* lewat AS-REP Roasting, sampai akhirnya melakukan DCSync attack yang berujung dapat akses Administrator lewat *Pass-the-Hash*. 

Meskipun prosesnya panjang dan harus *troubleshooting* sana-sini (terutama pas awal-awal *setup* DNS di `/etc/hosts` yang sempat bikin nyangkut), rasanya terbayar lunas pas *root flag* berhasil didapat. Lab yang sangat solid buat nambah fundamental *red teaming*!
