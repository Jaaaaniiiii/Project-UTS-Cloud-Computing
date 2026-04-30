# Project-UTS-Cloud-Computing
Project UTS Cloud Computing - Uptime Kuma

Nama: Janiar Rahma Putri
Kelas: TT-46-GAB1
MK: Cloud Computing
TELKOM UNIVERSITY

#**Kelompok 8 — Uptime Kuma**.
---

## Step 1: Pastikan Docker sudah terinstall
1. Update, Upgrade, dan install docker khusus untuk Ubuntu
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install docker.io docker-compose-v2 -y
sudo usermod -aG docker $USER
```
Note: usermod artinya disaat kita menjalan perintah, maka tidak perlu menggunakan sudo lagi alias bisa langsung 'docker [perintah yang dibutuhkan]'

2. Cek versi docker (pastikan yang terbaru)
```bash
docker --version
docker compose version
```
<img width="942" height="115" alt="image" src="https://github.com/user-attachments/assets/492bc423-7137-4d43-abd2-620d1ef276bf" />

3. Sebelum menjalankan docker ps, pastikan beri izin akses secara manual ke file socket-nya
```bash
sudo chmod 666 /var/run/docker.sock 
docker ps
```
<img width="739" height="70" alt="image" src="https://github.com/user-attachments/assets/9f311f5c-e5cb-4ea2-ac71-7c0d1d0e4e25" />


Kalau output normal (tidak ada error), lanjut. Kalau error "Cannot connect to Docker daemon":
```bash
sudo systemctl start docker
sudo systemctl status docker
```
<img width="646" height="355" alt="image" src="https://github.com/user-attachments/assets/07bd707b-04c4-44a6-aaa5-0dd1118642b0" />

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
<img width="512" height="70" alt="image" src="https://github.com/user-attachments/assets/fd17a9fc-3f7b-425e-b079-d676ab4f6c41" />

Isi dengan ini (ini sudah include volume untuk tantangan tambahan):

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
<img width="828" height="401" alt="image" src="https://github.com/user-attachments/assets/c6990769-1f72-4929-92f7-2eda1e1b0fc2" />

Simpan: **Ctrl+X → Y → Enter**

---

## Step 3: Jalankan container

```bash
docker compose up -d
```
<img width="858" height="141" alt="image" src="https://github.com/user-attachments/assets/b38ebb60-d1c5-4a06-b4fb-9b2b0cc5f5ab" />


Cek status:
```bash
docker ps
```
Harus muncul container `uptime-kuma` dengan status **Up**.
<img width="858" height="41" alt="image" src="https://github.com/user-attachments/assets/46d60cb3-b51c-41b2-9dd3-1dfe309f6517" />

---

## Step 4: Verifikasi di browser

Buka `http://localhost:3001` → akan muncul halaman setup akun admin pertama kali.

Buat username & password, lalu **catat baik-baik** karena nanti dipakai login ulang saat verifikasi persistensi data.
<img width="1000" height="1112" alt="Screenshot From 2026-04-30 12-28-53" src="https://github.com/user-attachments/assets/5830108a-41ea-4edf-b4ec-0c1a8e39583a" />
<img width="1920" height="1072" alt="Screenshot From 2026-04-30 12-30-27" src="https://github.com/user-attachments/assets/33d4c729-a901-4dd1-9fad-f204786a423a" />

---

## Step 5: Install & setup Ngrok

```bash
curl -sSL https://ngrok-agent.s3.amazonaws.com/ngrok.asc \
  | sudo tee /etc/apt/trusted.gpg.d/ngrok.asc > /dev/null

echo "deb https://ngrok-agent.s3.amazonaws.com buster main" \
  | sudo tee /etc/apt/sources.list.d/ngrok.list

sudo apt update && sudo apt install ngrok
```

- Daftar akun gratis di **ngrok.com**, 
<img width="1920" height="1072" alt="Screenshot From 2026-04-30 12-40-46" src="https://github.com/user-attachments/assets/1d2cf72c-0422-48ab-8744-32da3c138593" />

- copy authtoken-nya, lalu:

```bash
ngrok config add-authtoken TOKEN_KAMU
```

Jalankan tunnel:
```bash
ngrok http 3001
```
<img width="325" height="358" alt="image" src="https://github.com/user-attachments/assets/1914c77d-f5bd-4788-a9e8-4d13daad9947" />
<img width="1920" height="1072" alt="Screenshot From 2026-04-30 12-49-01" src="https://github.com/user-attachments/assets/ac4e9410-52f8-4473-a5a9-4add1f7498ba" />
<img width="1008" height="193" alt="image" src="https://github.com/user-attachments/assets/c0676cf5-9db3-4fd3-ac52-2c156784fbe2" />

Catat URL yang muncul di bagian Forwarding, contoh: `https://abc123.ngrok-free.app`

---

## Step 6: Tantangan Tambahan — Persistensi Data
**6a. Tambah beberapa monitor dulu di Uptime Kuma**
- Buka `localhost:3001` → login
- Klik **"Add New Monitor"**
- Isi URL apa saja, misalnya `https://google.com`
- Tambah 2-3 monitor
- **Screenshot dashboard** (ini bukti "before")
<img width="1920" height="1072" alt="Screenshot From 2026-04-30 12-51-48" src="https://github.com/user-attachments/assets/0964fe65-eca1-4753-a9cd-bfc92d1f4cda" />

**6b. Stop container tanpa hapus volume**
```bash
docker compose down
# JANGAN pakai -v !
```
<img width="854" height="195" alt="image" src="https://github.com/user-attachments/assets/33af0527-3d02-478b-9d1c-1a37d08c8a41" />

**6c. Verifikasi volume masih ada**
```bash
docker volume ls
# Harus muncul: uptime-kuma-deploy_uptime-kuma-data
```
<img width="854" height="92" alt="image" src="https://github.com/user-attachments/assets/0c80c255-3b27-4e0b-bc68-42bd00528f44" />


**6d. Jalankan ulang**
```bash
docker compose up -d
```

**6e. Buka lagi localhost:3001** → login dengan akun yang sama → monitor yang tadi dibuat **harus masih ada**.
<img width="1920" height="1072" alt="Screenshot From 2026-04-30 12-57-56" src="https://github.com/user-attachments/assets/3939a659-3400-49e6-8ee1-934cf1b11fac" />

---
