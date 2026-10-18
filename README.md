# Docker phpMyAdmin & MariaDB Setup

Konfigurasi Docker Compose untuk menjalankan **phpMyAdmin** dan **MariaDB (10.11)** secara terintegrasi dengan dukungan otomatisasi **CI/CD ke VPS Ubuntu**.

---

## 🚀 Fitur Konfigurasi
- **MariaDB 10.11 Container (`dev_mariadb`)**: Dilengkapi persistent volume sehingga data tersimpan aman meski kontainer dimatikan/dihapus.
- **Auto-Connect**: phpMyAdmin sudah langsung terhubung ke kontainer MariaDB secara otomatis.
- **PMA_ARBITRARY=1**: Masih dapat berpindah ke server database lain jika diinginkan.
- **Port Bebas Konflik**:
  - phpMyAdmin: `8086`
  - MariaDB: `3309` (port 3306, 3307, 3308 sudah digunakan kontainer lain di PC Anda)
- **Healthcheck Otomatis**: phpMyAdmin menunggu MariaDB siap menerima koneksi sebelum mulai melayani request.
- **CI/CD Auto-Deploy via GitHub Actions**: Setiap kali Anda melakukan `git push` ke branch `main`, konfigurasi otomatis divalidasi dan dideploy langsung ke VPS Ubuntu Anda.

---

## 🔑 Kredensial Default MariaDB

Kredensial dapat diatur melalui file [`.env`](file:///.env) (gunakan [`.env.example`](file:///.env.example) sebagai template):

| Parameter | Nilai Default |
|---|---|
| **Host dari phpMyAdmin** | `mariadb` (otomatis terisi) |
| **Host dari Host Windows / VPS (DBeaver / Navicat / App)** | `localhost` atau `127.0.0.1` |
| **Port Eksternal** | `3309` |
| **Root Password** | `root` |
| **Database Name** | `my_database` |
| **Username** | `dbuser` |
| **Password** | `dbpass` |

---

## 🛠️ Cara Menjalankan Secara Lokal

### 1. Menjalankan Kontainer
```bash
docker compose up -d
```

### 2. Membuka phpMyAdmin
Buka di browser:
👉 **[http://localhost:8086](http://localhost:8086)**

- **Server**: `mariadb` *(default)*
- **Username**: `root` *(atau `dbuser`)*
- **Password**: `root` *(atau `dbpass`)*

### 3. Menghentikan Kontainer
```bash
docker compose down
```

*(Catatan: Jika ingin menghapus data database secara total, gunakan perintah `docker compose down -v`)*

---

## 🔄 Otomatisasi CI/CD ke VPS Ubuntu

File workflow GitHub Actions terletak di [`.github/workflows/deploy.yml`](file:///.github/workflows/deploy.yml).

### Alur Kerja (Workflow):
1. **Push ke GitHub**: Setiap ada commit baru di branch `main` (atau ditrigger manual via tab Actions).
2. **Lint & Validation**: Runner GitHub Actions memvalidasi integritas sintaks [`docker-compose.yml`](file:///docker-compose.yml).
3. **SSH Remote Deploy**: Runner menghubungi VPS Ubuntu via SSH:
   - Membuat/menyiapkan direktori proyek di VPS (default: `/home/ubuntu/Docker_PHPMyAdmin`).
   - Melakukan sinkronisasi repositori terbaru (`git pull` / `git clone`).
   - Menyiapkan environment variable `.env`.
   - Menjalankan `docker compose pull` dan `docker compose up -d --remove-orphans`.
   - Membersihkan docker image lama yang tidak terpakai (`docker image prune -f`).

---

### ⚙️ Pengaturan GitHub Secrets

Buka repositori GitHub Anda di browser:
👉 **Settings** ➔ **Secrets and variables** ➔ **Actions** ➔ Klik **New repository secret**

Tambahkan secret berikut:

| Nama Secret | Wajib? | Keterangan & Contoh Nilai |
|---|---|---|
| `VPS_HOST` | **Wajib** | Alamat IP Publik atau domain VPS Anda (contoh: `103.123.45.67`) |
| `VPS_USERNAME` | **Wajib** | Username SSH VPS (contoh: `ubuntu` atau `root`) |
| `VPS_SSH_KEY` | **Pilihan 1 (Disarankan)** | Private SSH Key server VPS Anda (isi lengkap dari `~/.ssh/id_rsa` atau `~/.ssh/id_ed25519` mulai dari `-----BEGIN ...` sampai `... END -----`) |
| `VPS_PASSWORD` | **Pilihan 2** | Password user VPS Anda (jika tidak menggunakan SSH Key) |
| `VPS_PORT` | Opsional | Port SSH VPS jika bukan default (default: `22`) |
| `VPS_TARGET_DIR` | Opsional | Direktori deploy di VPS (default: `/home/ubuntu/Docker_PHPMyAdmin`) |
| `ENV_FILE` | Opsional | Isi lengkap file `.env` untuk server VPS produksi Anda |

---

### 🖥️ Persiapan Singkat di VPS Ubuntu (Sekali Saja)

Pastikan Docker dan Docker Compose telah terpasang di VPS Anda:

```bash
# 1. Update paket sistem
sudo apt update && sudo apt upgrade -y

# 2. Install Docker & Docker Compose Plugin
curl -fsSL https://get.docker.com | sh

# 3. Izinkan user non-root menjalankan docker tanpa sudo (contoh user: ubuntu)
sudo usermod -aG docker $USER

# 4. Terapkan perubahan grup (atau logout dan login kembali ke SSH)
newgrp docker

# 5. Buka port firewall (UFW) jika firewall aktif
sudo ufw allow 22/tcp
sudo ufw allow 8086/tcp
sudo ufw allow 3309/tcp
sudo ufw reload
```
