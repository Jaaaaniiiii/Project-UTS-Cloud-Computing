# 🚀 Project UTS Cloud Computing - Uptime Kuma

<p align="center">
  <img src="https://img.shields.io/badge/Status-Completed-success?style=for-the-badge" alt="Status">
  <img src="https://img.shields.io/badge/Course-Cloud%20Computing-blue?style=for-the-badge" alt="Course">
  <img src="https://img.shields.io/badge/University-Telkom%20University-red?style=for-the-badge" alt="Uni">
</p>

---

### 📝 Deskripsi Proyek
Proyek ini dikembangkan untuk memenuhi tugas Ujian Tengah Semester mata kuliah **Cloud Computing**. Fokus utama proyek ini adalah implementasi dan monitoring menggunakan **Uptime Kuma**.

### 👥 Anggota Kelompok (TT-46-GAB1)

| NIM | Nama |
| :---: | :--- |
| **101012300091** | Janiar Rahma Putri |
| **101012300225** | Devdan Wisesa Putranto | 
| **101012300202** | Muhammad Atqiyya |
| *N/A* | Muhammad Azrilhumaidi |

---

### 🛠️ Tech Stack & Tools
- **Platform:** Cloud Computing
- **Monitoring Tool:** [Uptime Kuma](https://github.com/louislam/uptime-kuma)
- **Environment:** Ubuntu / Docker / Uptime Kuma / Ngrox

---

### 🏛️ Institusi
**Fakultas Teknik Elektro**  
**Telkom University**  

---

# Kelompok 8 — Uptime Kuma
<img width="1600" height="900" alt="image" src="https://github.com/user-attachments/assets/21a5ca08-f933-4fb4-b701-7e56910c900e" />

## Step 1: Pastikan Docker sudah terinstall

1. Update, Upgrade, dan install docker khusus untuk Ubuntu

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install docker.io docker-compose-v2 -y
sudo usermod -aG docker $USER
```

> **Note:** `usermod` artinya disaat kita menjalankan perintah, maka tidak perlu menggunakan `sudo` lagi alias bisa langsung `docker [perintah yang dibutuhkan]`

2. Cek versi docker (pastikan yang terbaru)

```bash
docker --version
docker compose version
```

![docker version](https://github.com/user-attachments/assets/492bc423-7137-4d43-abd2-620d1ef276bf)

3. Sebelum menjalankan `docker ps`, pastikan beri izin akses secara manual ke file socket-nya

```bash
sudo chmod 666 /var/run/docker.sock
docker ps
```

![docker ps awal](https://github.com/user-attachments/assets/9f311f5c-e5cb-4ea2-ac71-7c0d1d0e4e25)

Kalau output normal (tidak ada error), lanjut. Kalau error "Cannot connect to Docker daemon":

```bash
sudo systemctl start docker
sudo systemctl status docker
```

![systemctl status](https://github.com/user-attachments/assets/07bd707b-04c4-44a6-aaa5-0dd1118642b0)

---

## Step 2: Buat folder & docker-compose.yml

```bash
# buat direktori
mkdir uptime-kuma-deploy

# masuk ke dalam direktori yang telah dibuat
cd uptime-kuma-deploy

# edit file docker compose
nano docker-compose.yml
```

![mkdir cd nano](https://github.com/user-attachments/assets/fd17a9fc-3f7b-425e-b079-d676ab4f6c41)

Isi dengan konfigurasi berikut (sudah include volume untuk tantangan tambahan):

```yaml
version: '3.8'
services:
  uptime-kuma:
    image: louislam/uptime-kuma:1
    container_name: uptime-kuma
    volumes:
      - uptime-kuma-data:/app/data
    ports:
      - "3001:3001"
    restart: unless-stopped

volumes:
  uptime-kuma-data:
```

![nano docker-compose.yml](https://github.com/user-attachments/assets/c6990769-1f72-4929-92f7-2eda1e1b0fc2)

Simpan: **Ctrl+X → Y → Enter**

### Penjelasan isi docker-compose.yml

| Baris | Penjelasan |
|---|---|
| `version: '3.8'` | Versi format file Docker Compose yang digunakan |
| `services:` | Mendefinisikan daftar container yang akan dijalankan |
| `uptime-kuma:` | Nama service (sekaligus nama internal di Docker network) |
| `image: louislam/uptime-kuma:1` | Image Docker yang digunakan, diambil dari Docker Hub. Tag `:1` artinya versi major 1 terbaru |
| `container_name: uptime-kuma` | Memberi nama eksplisit pada container agar mudah dirujuk dengan perintah seperti `docker exec` |
| `volumes:` (dalam service) | Menghubungkan named volume `uptime-kuma-data` ke path `/app/data` di dalam container — inilah kunci persistensi data |
| `ports: - "3001:3001"` | Memetakan port 3001 di host ke port 3001 di container, format `[host]:[container]` |
| `restart: unless-stopped` | Container otomatis restart jika crash, kecuali jika sengaja di-stop manual |
| `volumes:` (di level atas) | Mendaftarkan named volume `uptime-kuma-data` agar dikelola oleh Docker (bukan bind mount ke folder host) |

---

## Step 3: Jalankan container

```bash
docker compose up -d
```

![docker compose up](https://github.com/user-attachments/assets/b38ebb60-d1c5-4a06-b4fb-9b2b0cc5f5ab)

Cek status:

```bash
docker ps
```

Harus muncul container `uptime-kuma` dengan status **Up**.

![docker ps](https://github.com/user-attachments/assets/46d60cb3-b51c-41b2-9dd3-1dfe309f6517)

---

## Step 4: Verifikasi di browser

Buka `http://localhost:3001` → akan muncul halaman setup akun admin pertama kali.

Buat username & password, lalu **catat baik-baik** karena nanti dipakai login ulang saat verifikasi persistensi data.

![setup akun](https://github.com/user-attachments/assets/5830108a-41ea-4edf-b4ec-0c1a8e39583a)

![dashboard awal](https://github.com/user-attachments/assets/33d4c729-a901-4dd1-9fad-f204786a423a)

---

## Step 5: Install & setup Ngrok

```bash
curl -sSL https://ngrok-agent.s3.amazonaws.com/ngrok.asc \
  | sudo tee /etc/apt/trusted.gpg.d/ngrok.asc > /dev/null

echo "deb https://ngrok-agent.s3.amazonaws.com buster main" \
  | sudo tee /etc/apt/sources.list.d/ngrok.list

sudo apt update && sudo apt install ngrok
```

Daftar akun gratis di **ngrok.com**, copy authtoken-nya:

![ngrok dashboard](https://github.com/user-attachments/assets/1d2cf72c-0422-48ab-8744-32da3c138593)

```bash
ngrok config add-authtoken TOKEN_KAMU
```

Jalankan tunnel:

```bash
ngrok http 3001
```

![ngrok config](https://github.com/user-attachments/assets/1914c77d-f5bd-4788-a9e8-4d13daad9947)

![ngrok running](https://github.com/user-attachments/assets/ac4e9410-52f8-4473-a5a9-4add1f7498ba)

![ngrok forwarding URL](https://github.com/user-attachments/assets/c0676cf5-9db3-4fd3-ac52-2c156784fbe2)

Catat URL yang muncul di bagian **Forwarding**, contoh: `https://abc123.ngrok-free.app`

### Verifikasi Ngrok Dashboard (localhost:4040)

Buka `http://localhost:4040` di browser untuk melihat request log yang masuk melalui tunnel secara real-time.
<img width="2160" height="1123" alt="image (2)" src="https://github.com/user-attachments/assets/fd610459-718e-4ffa-b4b9-3d0d52d5e6f4" />
> **Screenshot wajib:** Minta rekan kelompok mengakses URL Ngrok dari HP mereka agar ada request yang muncul di dashboard 4040.

Hasil:
![ngrok dashboard 4040](https://github.com/user-attachments/assets/28615a43-4709-42e6-a906-0a857793c84f)

### Verifikasi dari HP / Browser Lain

Buka URL Ngrok dari HP atau browser berbeda untuk membuktikan aplikasi bisa diakses publik.

![akses dari HP](https://github.com/user-attachments/assets/7dccf318-68a3-40bc-992e-90ed3beab69b)
![akses dari HP](https://github.com/user-attachments/assets/971ee282-c616-4bf1-9498-67cc7aca43e7)

---

## Step 6: Tantangan Tambahan — Persistensi Data

### 6a. Tambah beberapa monitor di Uptime Kuma

- Buka `localhost:3001` → login
- Klik **"Add New Monitor"**
- Isi URL apa saja, misalnya `https://google.com`
- Tambah 2–3 monitor
- **Screenshot dashboard** (bukti "before")

![dashboard before](https://github.com/user-attachments/assets/0964fe65-eca1-4753-a9cd-bfc92d1f4cda)

### 6b. Stop container tanpa hapus volume

```bash
docker compose down
# JANGAN pakai -v !
```

![docker compose down](https://github.com/user-attachments/assets/33af0527-3d02-478b-9d1c-1a37d08c8a41)

### 6c. Verifikasi volume masih ada

```bash
docker volume ls
# Harus muncul: uptime-kuma-deploy_uptime-kuma-data
```

![docker volume ls](https://github.com/user-attachments/assets/0c80c255-3b27-4e0b-bc68-42bd00528f44)

### 6d. Jalankan ulang

```bash
docker compose up -d
```

![docker compose up -d](https://github.com/user-attachments/assets/79678c03-f3bc-416b-9622-423977005686)

### 6e. Verifikasi data masih ada

Buka lagi `localhost:3001` → login dengan akun yang sama → monitor yang dibuat sebelumnya **harus masih ada**.

![dashboard after](https://github.com/user-attachments/assets/3939a659-3400-49e6-8ee1-934cf1b11fac)

> Data tetap ada setelah restart karena volume `uptime-kuma-data` tidak dihapus. Ini membuktikan persistensi data berhasil.

---

## Kendala dan Solusi

### Kendala 1: Permission denied saat menjalankan `docker ps`

**Error:**
```
permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock
```

**Penyebab:** User belum memiliki akses ke Docker socket.

**Solusi:**
```bash
sudo chmod 666 /var/run/docker.sock
```

Atau untuk solusi permanen (tidak perlu diulang setiap restart):
```bash
sudo usermod -aG docker $USER
newgrp docker
```

---

### Kendala 2: Docker daemon tidak berjalan

**Error:**
```
Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
```

**Penyebab:** Docker service belum dijalankan.

**Solusi:**
```bash
sudo systemctl start docker
sudo systemctl enable docker   # agar otomatis start saat boot
sudo systemctl status docker   # verifikasi sudah running
```

---

### Kendala 3: Port 3001 sudah dipakai proses lain

**Error:**
```
Error response from daemon: Ports are not available: exposing port TCP 0.0.0.0:3001 -> 0.0.0.0:0: listen tcp 0.0.0.0:3001: bind: address already in use
```

**Penyebab:** Port 3001 sudah digunakan proses lain di host.

**Solusi:**
```bash
# Cek proses yang menggunakan port 3001
sudo lsof -i :3001

# Matikan prosesnya, atau ganti port di docker-compose.yml
# Ubah "3001:3001" menjadi "3002:3001" misalnya
```

---

### Kendala 4: Ngrok error "authentication failed" / "invalid token"

**Error:**
```
ERROR: authentication failed: The authtoken you specified does not match...
```

**Penyebab:** Authtoken belum didaftarkan atau salah copy.

**Solusi:**
```bash
# Login ke dashboard.ngrok.com/get-started/your-authtoken
# Copy token, lalu:
ngrok config add-authtoken TOKEN_YANG_BENAR

# Verifikasi token tersimpan:
cat ~/.config/ngrok/ngrok.yml
```

---

### Kendala 5: Browser tampilkan "502 Bad Gateway" saat buka URL Ngrok

**Penyebab:** Ngrok sudah berjalan tapi aplikasi di dalam Docker belum merespons.

**Solusi:**
1. Test localhost dulu: buka `http://localhost:3001` di browser. Kalau localhost saja gagal, selesaikan masalah Docker dulu
2. Tunggu 30–60 detik untuk inisialisasi aplikasi, lalu refresh
3. Cek port sudah sesuai: `docker ps --format "table {{.Names}}\t{{.Ports}}"`
4. Jika pertama kali buka URL Ngrok, akan ada halaman interstitial — klik **"Visit Site"** untuk lanjut

---

## Analisis: Apa yang Dipelajari tentang Docker Networking & Tunneling

### Docker Networking

Pada project ini, Uptime Kuma berjalan di dalam sebuah container yang terisolasi dari host. Docker secara otomatis membuat internal network untuk container, sehingga container mendapat IP internal tersendiri. Karena kita menggunakan `ports: "3001:3001"`, Docker melakukan **port binding** — yaitu meneruskan traffic yang masuk ke port 3001 di host ke port 3001 di dalam container.

Konsep penting yang dipelajari:
- **Isolated network namespace:** Setiap container punya network stack sendiri, tidak langsung terekspos ke host kecuali kita eksplisit bind port
- **Named volumes vs bind mount:** Named volume (`uptime-kuma-data`) dikelola sepenuhnya oleh Docker dan persist meskipun container dihapus (selama tidak pakai `-v`). Berbeda dengan bind mount yang langsung terhubung ke folder host
- **`restart: unless-stopped`:** Container akan otomatis restart jika crash atau jika host di-reboot, kecuali jika kita sengaja stop manual — berguna untuk aplikasi produksi

### Tunneling dengan Ngrok

Secara default, aplikasi yang berjalan di `localhost` hanya bisa diakses dari mesin yang sama. Ngrok menyelesaikan masalah ini dengan cara:

1. Ngrok client (di laptop) membuka koneksi keluar ke Ngrok cloud server
2. Ngrok cloud memberi URL publik (`https://xxxx.ngrok-free.app`)
3. Setiap request yang masuk ke URL publik tersebut diteruskan melalui tunnel ke Ngrok client, lalu ke `localhost:3001`

Ini berbeda dari port forwarding biasa karena tidak membutuhkan IP publik atau konfigurasi router. Koneksi selalu diinisiasi dari dalam (outbound), sehingga bisa menembus NAT dan firewall. Di industri, alternatif yang lebih production-ready antara lain **Cloudflare Tunnel**, **Tailscale Funnel**, atau **AWS Session Manager Port Forwarding**.

---

## Pertanyaan Acuan Demo — Beserta Jawaban

### Docker

**Q1: Apa fungsi `depends_on` di docker-compose?**

`depends_on` digunakan untuk mengatur urutan startup antar container. Misalnya jika service `app` memiliki `depends_on: db`, maka Docker Compose akan menjalankan container `db` terlebih dahulu sebelum `app`. Namun penting dicatat: `depends_on` hanya menjamin urutan *start*, bukan menjamin service sudah *ready* menerima koneksi. Untuk itu biasanya dikombinasikan dengan `healthcheck`.

---

**Q2: Apa perbedaan `volumes` dan `bind mount`?**

| | Named Volume | Bind Mount |
|---|---|---|
| **Definisi** | Storage yang dikelola sepenuhnya oleh Docker | Menghubungkan folder spesifik dari host langsung ke container |
| **Sintaks** | `- nama-volume:/path/di/container` | `- /path/host:/path/container` |
| **Lokasi data** | Dikelola Docker, biasanya di `/var/lib/docker/volumes/` | Di lokasi folder host yang ditentukan |
| **Portabilitas** | Lebih portable, tidak bergantung struktur folder host | Bergantung pada path host yang spesifik |
| **Use case** | Database, data aplikasi yang perlu persist | Config file, source code saat development |

Pada project ini kita menggunakan **named volume** (`uptime-kuma-data`) sehingga data Uptime Kuma tetap ada meskipun container di-stop dan di-start ulang.

---

**Q3: Jelaskan cara kerja Ngrok secara konseptual.**

Ngrok bekerja sebagai **reverse tunnel**. Alurnya:
1. Ngrok client dijalankan di laptop dan membuka koneksi *outbound* ke Ngrok cloud server
2. Ngrok cloud menerbitkan URL publik unik (misal `https://abc123.ngrok-free.app`)
3. Ketika ada orang mengakses URL publik tersebut, request diterima oleh Ngrok cloud
4. Ngrok cloud meneruskan request tersebut melalui tunnel yang sudah terbuka ke Ngrok client di laptop
5. Ngrok client meneruskan request ke aplikasi lokal di `localhost:3001`
6. Response berjalan balik melalui jalur yang sama

Yang membuat ini istimewa: laptop tidak perlu IP publik, tidak perlu setting router/firewall, karena koneksi selalu dimulai dari dalam (outbound). Ngrok juga menyediakan dashboard di `localhost:4040` untuk memonitor semua request yang masuk.

---

**Q4: Apa yang terjadi pada data jika `docker compose down -v` dijalankan?**

Flag `-v` pada `docker compose down` akan **menghapus semua named volume** yang didefinisikan di `docker-compose.yml` beserta seluruh data di dalamnya secara permanen. Ini tidak bisa di-undo.

Perbandingan:
- `docker compose down` → hanya menghapus container dan network, **volume tetap ada**
- `docker compose down -v` → menghapus container, network, **dan semua volume** → **data hilang permanen**

Pada tantangan tambahan kelompok 8 ini, kita membuktikan bahwa menjalankan `docker compose down` (tanpa `-v`) lalu `docker compose up -d` kembali akan mempertahankan semua data monitor yang sudah dibuat di Uptime Kuma, karena volume `uptime-kuma-data` tidak ikut terhapus.

---

## Checklist Screenshot Laporan

- [x] `docker ps` semua container Up
- [x] Browser `localhost:3001` aplikasi jalan
- [x] Terminal Ngrok aktif + URL publik muncul
- [x] `localhost:4040` Ngrok dashboard (ada request masuk)
- [x] URL Ngrok dibuka dari HP/browser lain 
- [x] Dashboard Uptime Kuma sebelum restart (before)
- [x] `docker volume ls` setelah `docker compose down`
- [x] Dashboard Uptime Kuma setelah restart (after) — data masih ada
