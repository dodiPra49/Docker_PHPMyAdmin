# Docker phpMyAdmin & MariaDB Setup

Konfigurasi Docker Compose untuk menjalankan **phpMyAdmin** dan **MariaDB (10.11)** secara terintegrasi.

---

## 🚀 Fitur Konfigurasi
- **MariaDB 10.11 Container (`dev_mariadb`)**: Dilengkapi persistent volume sehingga data tersimpan aman meski kontainer dimatikan/dihapus.
- **Auto-Connect**: phpMyAdmin sudah langsung terhubung ke kontainer MariaDB secara otomatis.
- **PMA_ARBITRARY=1**: Masih dapat berpindah ke server database lain jika diinginkan.
- **Port Bebas Konflik**:
  - phpMyAdmin: `8086`
  - MariaDB: `3309` (port 3306, 3307, 3308 sudah digunakan kontainer lain di PC Anda)
- **Healthcheck Otomatis**: phpMyAdmin menunggu MariaDB siap menerima koneksi sebelum mulai melayani request.

---

## 🔑 Kredensial Default MariaDB

Kredensial dapat diubah melalui file [`.env`](file:///.env):

| Parameter | Nilai Default |
|---|---|
| **Host dari phpMyAdmin** | `mariadb` (otomatis terisi) |
| **Host dari Host Windows (DBeaver / Navicat / App)** | `localhost` atau `127.0.0.1` |
| **Port Eksternal** | `3309` |
| **Root Password** | `root` |
| **Database Name** | `my_database` |
| **Username** | `dbuser` |
| **Password** | `dbpass` |

---

## 🛠️ Cara Menjalankan

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
