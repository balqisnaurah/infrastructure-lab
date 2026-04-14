# Infrastructure Lab

Dokumentasi eksplorasi infrastruktur cloud menggunakan VirtualBox dan Docker. Repository ini berisi catatan konfigurasi dan screenshot dari setiap tahap yang dilakukan.

---

## Daftar Eksperimen

### 1. Setup Virtual Machine

Instalasi Ubuntu Server 24.04 LTS di Oracle VirtualBox.

| Parameter | Nilai |
|-----------|-------|
| RAM | 2048 MB |
| CPU | 2 Core |
| Storage | 20 GB (LVM) |
| Network | NAT dengan Port Forwarding |
| SSH | OpenSSH Server aktif |

Proses yang dilakukan meliputi konfigurasi jaringan, partisi storage dengan LVM, aktivasi SSH server, dan pembuatan user.

---

### 2. Docker - Nginx Web Server

Menjalankan web server Nginx di dalam Docker container dan mengaksesnya melalui browser via port forwarding.

**Perintah yang digunakan:**

```bash
docker run -d -p 8080:80 --name webserver nginx
```

**Port Forwarding (VirtualBox):**

| Host Port | Guest Port | Service |
|-----------|-----------|---------|
| 8080 | 8080 | Nginx |

Hasil: halaman "Welcome to nginx!" berhasil diakses melalui `http://localhost:8080` di browser Windows.

---

### 3. Docker Compose - WordPress + MySQL

Deploy aplikasi CMS WordPress beserta database MySQL menggunakan Docker Compose. Dua service dikonfigurasi agar saling terhubung melalui Docker network internal.

**Konfigurasi `docker-compose.yml`:**

```yaml
version: '3'
services:
  wordpress:
    image: wordpress:latest
    ports:
      - "8081:80"
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: wpuser
      WORDPRESS_DB_PASSWORD: your_password_here
      WORDPRESS_DB_NAME: wpdb
    depends_on:
      - db
  db:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD: your_password_here
      MYSQL_DATABASE: wpdb
      MYSQL_USER: wpuser
      MYSQL_PASSWORD: your_password_here
```

**Perintah yang digunakan:**

```bash
docker-compose up -d
```

Hasil: halaman setup WordPress berhasil diakses melalui `http://localhost:8081`.

---

### 4. Monitoring - Grafana

Menjalankan Grafana di Docker untuk eksplorasi tools monitoring dan visualisasi data.

**Perintah yang digunakan:**

```bash
docker run -d -p 3000:3000 --name grafana grafana/grafana
```

**Port Forwarding (VirtualBox):**

| Host Port | Guest Port | Service |
|-----------|-----------|---------|
| 3000 | 3000 | Grafana |

Hasil: dashboard Grafana berhasil diakses melalui `http://localhost:3000` dengan login default (admin/admin).

---

## Ringkasan Port Forwarding

| Service | Host Port | Guest Port | Protokol |
|---------|-----------|-----------|----------|
| Nginx | 8080 | 8080 | TCP |
| WordPress | 8081 | 8081 | TCP |
| Grafana | 3000 | 3000 | TCP |

---

## Screenshots

Semua bukti dokumentasi tersedia di folder [`/screenshots`](./screenshots).

---

## Tools yang Digunakan

| Tools | Versi | Fungsi |
|-------|-------|--------|
| Oracle VirtualBox | 7.2 | Virtualisasi untuk menjalankan Ubuntu Server |
| Ubuntu Server | 24.04 LTS | Sistem operasi di dalam VM |
| Docker | latest | Container runtime |
| Docker Compose | latest | Orkestrasi multi-container |
| Nginx | latest | Web server |
| WordPress | latest | Content Management System (CMS) |
| MySQL | 5.7 | Database server untuk WordPress |
| Grafana | latest | Dashboard monitoring dan visualisasi |

---

## Tentang

Repository ini dibuat sebagai dokumentasi proses belajar provisioning virtual machine, container, dan tools monitoring yang umum digunakan dalam pengelolaan infrastruktur cloud.
