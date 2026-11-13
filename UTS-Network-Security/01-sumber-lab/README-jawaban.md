# LAPORAN HASIL PRAKTIKUM UTS NETWORK SECURITY
## KASUS 2

| Identitas | Keterangan |
|---|---|
| **Nama** | NAZWA ZAHIRA SYA'BANA |
| **NIM** | 24552011110 |
| **Kelas** | TIF RP 24 CNS E |
| **Mata Kuliah** | Network Security |
| **Kasus** | Kasus 2 |

---

# A. PENDAHULUAN

Praktikum Ujian Tengah Semester (UTS) Network Security Kasus 2 dilaksanakan untuk memahami konsep keamanan jaringan melalui praktik langsung menggunakan Kali Linux, Docker, dan berbagai tools keamanan jaringan.

Praktikum ini terdiri dari tujuh tugas:

1. Docker Compose dan Docker Volume.
2. Kriptografi menggunakan OpenSSL.
3. ARP Spoofing.
4. Analisis WPA2 Handshake.
5. Pengiriman ICMP menggunakan Scapy.
6. TCP SYN dan Three-Way Handshake.
7. DNS Exfiltration.

Seluruh pengujian dilakukan pada lingkungan laboratorium sendiri dan jaringan yang diizinkan untuk keperluan pembelajaran.

# B. LINGKUNGAN PRAKTIKUM

| Komponen | Keterangan |
|---|---|
| Sistem operasi utama | Windows 11 |
| Virtualisasi | Oracle VirtualBox |
| Sistem operasi virtual | Kali Linux |
| Container | Docker |
| Bahasa pemrograman | Python |
| Kriptografi | OpenSSL |
| Analisis Wi-Fi | Aircrack-ng |
| Analisis paket | tcpdump |
| Pengujian jaringan | hping3 |
| Pembuatan paket jaringan | Scapy |

**Lokasi folder praktikum:**

```bash
/root/Network-Security-Course-Bank/UTS-Network-Security/01-sumber-lab
```

Untuk masuk ke folder praktikum:

```bash
cd /root/Network-Security-Course-Bank/UTS-Network-Security/01-sumber-lab
```

# C. HASIL PRAKTIKUM

## TUGAS 1 — DOCKER COMPOSE DAN DOCKER VOLUME

### 1. Tujuan

Memahami penggunaan Docker Compose untuk menjalankan layanan dalam container serta Docker Volume untuk menyimpan data agar tetap tersedia meskipun container dihapus.

### 2. Langkah Praktikum

Memeriksa konfigurasi Docker Compose:

```bash
docker compose -p k2 -f b/b1-compose/docker-compose.yml config
```

Menjalankan layanan:

```bash
docker compose -p k2 -f b/b1-compose/docker-compose.yml up -d
```

Membuat Docker Volume:

```bash
docker volume create vol-k2
```

Container sementara bernama `w-k2` digunakan untuk menulis file `laporan-k2.txt` ke dalam volume `vol-k2`. Setelah proses penulisan selesai, container sementara dihapus.

Selanjutnya, volume diperiksa kembali menggunakan container baru untuk memastikan file masih tersimpan.

### 3. Hasil Pengujian

- Konfigurasi Docker Compose berhasil diperiksa.
- Layanan Docker Compose berhasil dijalankan.
- Docker Volume `vol-k2` berhasil dibuat.
- File `laporan-k2.txt` tetap tersedia setelah container penulis dihapus.
- Pengujian layanan web melalui container attacker berhasil dilakukan.

### 4. Kesimpulan

Tugas 1 berhasil. Docker Compose dapat digunakan untuk menjalankan layanan, sedangkan Docker Volume dapat mempertahankan data meskipun container yang menulis data telah dihapus.

**Dokumentasi screenshot:**

- `bukti/Tugas 1/1.png`
- `bukti/Tugas 1/2.png`

---

## TUGAS 2 — KRIPTOGRAFI MENGGUNAKAN OPENSSL

### 1. Tujuan

Memahami penggunaan OpenSSL untuk membuat sertifikat, mengenkripsi file, mendekripsi file, dan memeriksa integritas data menggunakan SHA-256.

### 2. Langkah Praktikum

Membangun image Docker khusus OpenSSL:

```bash
docker build -t uts-openssl images/uts-openssl
```

Membuat private key dan sertifikat:

```bash
docker run --rm -v "$PWD":/work uts-openssl sh -c 'openssl req -x509 -newkey rsa:2048 -nodes -keyout key-k2.pem -out cert-k2.pem -days 30 -subj /CN=k2.uts.local && openssl x509 -in cert-k2.pem -noout -subject -issuer -dates'
```

Perintah tersebut menghasilkan file `key-k2.pem` dan `cert-k2.pem`.

Membuat file laporan, melakukan enkripsi AES-256-CBC, menghitung hash, melakukan dekripsi, dan membandingkan hash file:

```bash
docker run --rm -v "$PWD":/work uts-openssl sh -c 'echo isi laporan k2 > laporan-k2.txt && openssl enc -aes-256-cbc -pbkdf2 -salt -in laporan-k2.txt -out laporan-k2.enc -pass pass:kunci-k2-2026 && sha256sum laporan-k2.txt && openssl enc -d -aes-256-cbc -pbkdf2 -in laporan-k2.enc -out hasil-k2.txt -pass pass:kunci-k2-2026 && sha256sum hasil-k2.txt'
```

### 3. Hasil Pengujian

Pembuatan private key dan sertifikat berhasil dilakukan.

File `laporan-k2.txt` berhasil dienkripsi menjadi `laporan-k2.enc`, kemudian berhasil didekripsi kembali menjadi `hasil-k2.txt`.

Hasil SHA-256 file asli dan file hasil dekripsi sama:

```text
752b81872b283c3c0e6f65245b7f2c508034847a89049dc4312dbb75366a4c23
```

### 4. Kesimpulan

Tugas 2 berhasil. OpenSSL dapat digunakan untuk membuat sertifikat, melakukan enkripsi dan dekripsi, serta memeriksa integritas file menggunakan SHA-256.

**Dokumentasi screenshot:**

- `bukti/Tugas 2/1.png`
- `bukti/Tugas 2/2.png`

---

## TUGAS 3 — ARP SPOOFING

### 1. Tujuan

Memahami cara kerja ARP Spoofing dengan mengamati perubahan pemetaan alamat IP dan MAC pada tabel ARP korban dalam jaringan laboratorium.

### 2. Langkah Praktikum

Membangun image tools keamanan:

```bash
docker build -t uts-kali-tools images/uts-kali-tools
```

Membuat jaringan Docker khusus:

```bash
docker network create lab-arp-k2
```

Menjalankan tiga container:

```bash
docker run -d --name korban-k2 --network lab-arp-k2 --cap-add NET_ADMIN uts-kali-tools
docker run -d --name gateway-k2 --network lab-arp-k2 --cap-add NET_ADMIN uts-kali-tools
docker run -d --name penyerang-k2 --network lab-arp-k2 --cap-add NET_ADMIN uts-kali-tools
```

Memeriksa alamat IP masing-masing container:

```bash
docker inspect -f '{{(index .NetworkSettings.Networks "lab-arp-k2").IPAddress}}' korban-k2
docker inspect -f '{{(index .NetworkSettings.Networks "lab-arp-k2").IPAddress}}' gateway-k2
docker inspect -f '{{(index .NetworkSettings.Networks "lab-arp-k2").IPAddress}}' penyerang-k2
```

Alamat IP yang diperoleh:

| Container | IP Address |
|---|---|
| korban-k2 | 172.19.0.2 |
| gateway-k2 | 172.19.0.3 |
| penyerang-k2 | 172.19.0.4 |

Memeriksa tabel ARP korban sebelum pengujian:

```bash
docker exec korban-k2 cat /proc/net/arp
```

Menjalankan ARP Spoofing selama 20 detik pada jaringan laboratorium:

```bash
docker exec -d penyerang-k2 timeout 20 arpspoof -i eth0 -t 172.19.0.2 172.19.0.3
```

Memeriksa kembali tabel ARP korban:

```bash
docker exec korban-k2 cat /proc/net/arp
```

### 3. Hasil Pengujian

Sebelum pengujian, alamat IP gateway `172.19.0.3` memiliki alamat MAC:

```text
da:48:77:85:fe:c2
```

Setelah ARP Spoofing, alamat IP gateway tersebut berubah dan mengarah ke alamat MAC penyerang:

```text
86:30:9b:57:76:ba
```

Perubahan tersebut terlihat pada tabel ARP milik container korban.

### 4. Kesimpulan

Tugas 3 berhasil. Pengujian membuktikan bahwa ARP Spoofing dapat memengaruhi pemetaan alamat IP ke MAC pada tabel ARP korban dalam jaringan laboratorium.

**Dokumentasi screenshot:**

- `bukti/Tugas 3/1.png`
- `bukti/Tugas 3/2.png`
- `bukti/Tugas 3/3.png`

---

## TUGAS 4 — ANALISIS WPA2 HANDSHAKE

### 1. Tujuan

Memahami proses penangkapan WPA2 Handshake dan menguji file hasil tangkapan menggunakan Aircrack-ng dengan wordlist pada jaringan Wi-Fi milik sendiri.

### 2. Langkah Praktikum

Memeriksa proses yang dapat mengganggu mode monitor:

```bash
airmon-ng check kill
```

Mengaktifkan mode monitor pada interface Wi-Fi:

```bash
airmon-ng start wlan0
```

Melakukan pemantauan jaringan Wi-Fi pada channel 1. Gunakan BSSID jaringan milik sendiri:

```bash
airodump-ng --bssid [BSSID_WIFI_SENDIRI] -c 1 -w handshake-k2 wlan0
```

Menunggu hingga handshake berhasil tertangkap.

Setelah file capture tersedia, melakukan pengujian menggunakan Aircrack-ng dan wordlist:

```bash
docker run --rm -v "$PWD":/work uts-kali-tools aircrack-ng -w /work/wordlist-k2.txt /work/handshake-k2-01.cap
```

### 3. Hasil Pengujian

Hasil pengujian terakhir menunjukkan:

```text
ESSID: Cyber
WPA (1 handshake)
8/8 keys tested
KEY FOUND!
```

Hasil tersebut menunjukkan bahwa:

- File capture berhasil dibaca.
- Aircrack-ng mendeteksi satu WPA handshake.
- Sebanyak 8 kandidat password berhasil diuji.
- Aircrack-ng menampilkan hasil `KEY FOUND!`.

### 4. Kesimpulan

Tugas 4 berhasil. Proses penangkapan WPA2 Handshake dan pengujian menggunakan wordlist berjalan dengan baik. Aircrack-ng berhasil menemukan password pada pengujian jaringan Wi-Fi milik sendiri.

**Dokumentasi screenshot:**

- `bukti/Tugas 4/1.png`
- `bukti/Tugas 4/2.png`
- `bukti/Tugas 4/3.png`

---

## TUGAS 5 — PENGIRIMAN ICMP MENGGUNAKAN SCAPY

### 1. Tujuan

Memahami pembuatan dan pengiriman paket ICMP menggunakan Python dan library Scapy.

### 2. Langkah Praktikum

File program yang digunakan:

```text
kasus/k5-scapy/kirim_icmp.py
```

Program digunakan untuk membuat paket ICMP Echo Request dan mengirimkannya ke alamat tujuan dalam jaringan laboratorium. Program kemudian memeriksa respons yang diterima dari target.

### 3. Hasil Pengujian

Hasil terminal menunjukkan:

```text
Mengirim: IP / ICMP 172.19.0.3 > 172.19.0.2 echo-request 0
TTL balasan: 64
Tipe ICMP balasan: 0
```

Hasil tersebut menunjukkan bahwa:

- Paket ICMP Echo Request berhasil dikirim.
- Target memberikan balasan Echo Reply.
- TTL balasan tercatat sebesar 64.
- Tipe ICMP 0 menunjukkan Echo Reply.

### 4. Kesimpulan

Tugas 5 berhasil. Scapy dapat digunakan untuk membuat dan mengirim paket ICMP serta menerima dan memeriksa respons dari target dalam jaringan laboratorium.

**Dokumentasi screenshot:**

- `bukti/Tugas 5/1.png`

---

## TUGAS 6 — TCP SYN DAN THREE-WAY HANDSHAKE

### 1. Tujuan

Memahami proses TCP Three-Way Handshake serta mengamati paket SYN, SYN-ACK, dan ACK menggunakan tools jaringan.

### 2. Langkah Praktikum

Memeriksa konfigurasi Docker Compose:

```bash
docker compose -p k2 -f b/b3-syn-flood/docker-compose.yml config
```

Menjalankan lingkungan pengujian:

```bash
docker compose -p k2 -f b/b3-syn-flood/docker-compose.yml up -d
```

Memeriksa jumlah koneksi dengan status `SYN_RECV` pada container target:

```bash
docker compose -p k2 -f b/b3-syn-flood/docker-compose.yml exec target netstat -ant | grep -c SYN_RECV
```

Pengujian dilakukan menggunakan container attacker dengan pengiriman paket SYN yang dibatasi untuk kebutuhan demonstrasi laboratorium. Pengamatan paket dilakukan menggunakan tcpdump.

### 3. Hasil Pengujian

Hasil pengujian menunjukkan:

- Sebelum pengujian, jumlah koneksi berstatus `SYN_RECV` tercatat 0.
- Pengiriman paket SYN terbatas berhasil dilakukan.
- tcpdump memperlihatkan paket SYN, SYN-ACK, dan ACK.
- Terlihat pula proses HTTP GET dan respons HTTP 200 OK.
- Setelah pengujian selesai, jumlah koneksi `SYN_RECV` kembali menjadi 0.

### 4. Analisis

TCP Three-Way Handshake terdiri dari tiga tahapan utama:

1. **SYN:** Client mengirim permintaan koneksi kepada server.
2. **SYN-ACK:** Server memberikan respons terhadap permintaan koneksi.
3. **ACK:** Client mengonfirmasi respons server sehingga koneksi dapat terbentuk.

Pengamatan menggunakan tcpdump memperlihatkan proses komunikasi TCP tersebut.

Pengujian dilakukan secara terbatas dan tidak dimaksudkan untuk membuat layanan di luar laboratorium terganggu.

### 5. Kesimpulan

Tugas 6 berhasil. Pengujian menunjukkan bahwa paket TCP SYN dapat dikirim dan proses komunikasi TCP dapat diamati melalui tcpdump. Hasil ini membantu memahami mekanisme Three-Way Handshake dan hubungan antara permintaan koneksi dengan respons server.

Pengujian ini tidak dijadikan bukti bahwa server berhasil mengalami denial of service.

**Dokumentasi screenshot:**

- `bukti/Tugas 6/1.png`
- `bukti/Tugas 6/2.png`
- `bukti/Tugas 6/3.png`
- `bukti/Tugas 6/4.png`
- `bukti/Tugas 6/5.png`

---

## TUGAS 7 — DNS EXFILTRATION

### 1. Tujuan

Memahami konsep DNS Exfiltration, yaitu penyisipan informasi ke dalam bagian nama domain pada query DNS, serta mengamati data tersebut melalui log DNS pada lingkungan laboratorium.

### 2. Langkah Praktikum

Memeriksa konfigurasi Docker Compose:

```bash
docker compose -p k2 -f b/b4-dns-exfil/docker-compose.yml config
```

Menjalankan lingkungan DNS Exfiltration:

```bash
docker compose -p k2 -f b/b4-dns-exfil/docker-compose.yml up -d
```

Domain yang digunakan dalam pengujian laboratorium:

```text
k2.exfil.lab
```

Query DNS dikirim melalui lingkungan praktikum dan aktivitasnya diamati melalui log DNS.

### 3. Hasil Pengujian

Hasil pengujian menunjukkan:

- Lingkungan Docker Compose berhasil dijalankan.
- Query DNS menggunakan domain laboratorium berhasil diuji.
- Respons DNS yang diamati menunjukkan alamat `10.0.0.99`.
- Log DNS memperlihatkan label data dalam bentuk hexadecimal.

### 4. Analisis

DNS Exfiltration merupakan konsep pengiriman data melalui query DNS dengan menempatkan informasi pada bagian label nama domain.

Pada praktikum ini, aktivitas query DNS dapat diamati melalui log DNS. Label data dalam bentuk hexadecimal menunjukkan bahwa informasi ditempatkan pada bagian nama domain yang dikirimkan.

Hasil yang tercatat membuktikan pengiriman dan pengamatan query DNS pada lingkungan laboratorium. Proses decoding data tidak dinyatakan berhasil apabila belum ada hasil decoding yang terverifikasi.

### 5. Kesimpulan

Tugas 7 berhasil. Pengujian membuktikan bahwa query DNS dapat membawa data pada bagian nama domain dan aktivitas tersebut dapat diamati melalui log DNS. Praktikum ini memperlihatkan konsep dasar DNS Exfiltration pada lingkungan laboratorium.

**Dokumentasi screenshot:**

- `bukti/Tugas 7/1.png`

---

# D. REKAPITULASI HASIL PRAKTIKUM

| No. | Nama Tugas | Hasil Pengujian | Status |
|---:|---|---|---|
| 1 | Docker Compose dan Volume | Data tetap tersedia setelah container penulis dihapus | Berhasil |
| 2 | Kriptografi OpenSSL | Enkripsi, dekripsi, dan pencocokan SHA-256 berhasil | Berhasil |
| 3 | ARP Spoofing | Pemetaan IP gateway berubah ke MAC penyerang | Berhasil |
| 4 | WPA2 Handshake | Handshake tertangkap dan Aircrack-ng menampilkan `KEY FOUND!` | Berhasil |
| 5 | ICMP menggunakan Scapy | Echo Request dikirim dan Echo Reply diterima | Berhasil |
| 6 | TCP SYN dan Three-Way Handshake | Paket TCP berhasil diamati menggunakan tcpdump | Berhasil |
| 7 | DNS Exfiltration | Query DNS dan label hexadecimal terlihat pada log | Berhasil |

# E. KESIMPULAN UMUM

Berdasarkan hasil praktikum UTS Network Security Kasus 2, seluruh tujuh tugas telah diuji pada lingkungan laboratorium.

Melalui praktikum ini, diperoleh pemahaman mengenai:

1. Penggunaan Docker Compose dan Docker Volume.
2. Penggunaan OpenSSL untuk enkripsi, dekripsi, pembuatan sertifikat, dan pemeriksaan integritas data.
3. Cara kerja ARP Spoofing dan dampaknya terhadap tabel ARP.
4. Proses penangkapan WPA2 Handshake dan pengujian menggunakan wordlist.
5. Pembuatan dan pengiriman paket ICMP menggunakan Scapy.
6. Mekanisme TCP Three-Way Handshake dan pengamatan paket menggunakan tcpdump.
7. Konsep DNS Exfiltration dan pengamatan query DNS melalui log.

Seluruh pengujian dilakukan dalam lingkungan yang diizinkan untuk tujuan pembelajaran keamanan jaringan.

# F. DOKUMENTASI SCREENSHOT

Seluruh screenshot praktikum berasal dari arsip `Screenshot-K2.zip`. Sebanyak 17 file gambar telah disalin ke dalam folder `bukti/` pada direktori project dan dikelompokkan berdasarkan Tugas 1 sampai Tugas 7.

Susunan folder dan file yang tersedia adalah:

```text
bukti/
├── Tugas 1/
│   ├── 1.png
│   └── 2.png
├── Tugas 2/
│   ├── 1.png
│   └── 2.png
├── Tugas 3/
│   ├── 1.png
│   ├── 2.png
│   └── 3.png
├── Tugas 4/
│   ├── 1.png
│   ├── 2.png
│   └── 3.png
├── Tugas 5/
│   └── 1.png
├── Tugas 6/
│   ├── 1.png
│   ├── 2.png
│   ├── 3.png
│   ├── 4.png
│   └── 5.png
└── Tugas 7/
    └── 1.png
```

Jumlah screenshot berdasarkan masing-masing tugas:

| Tugas | Jumlah Screenshot |
|---|---:|
| Tugas 1 | 2 |
| Tugas 2 | 2 |
| Tugas 3 | 3 |
| Tugas 4 | 3 |
| Tugas 5 | 1 |
| Tugas 6 | 5 |
| Tugas 7 | 1 |
| Total   |17 |

# G. PENUTUP

Demikian laporan hasil praktikum UTS Network Security Kasus 2 ini dibuat sebagai dokumentasi proses pengerjaan dan hasil pengujian tujuh tugas praktikum.

Laporan ini diharapkan dapat menunjukkan pemahaman terhadap konsep keamanan jaringan, penggunaan tools keamanan, serta analisis hasil pengujian dalam lingkungan laboratorium.
