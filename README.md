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

### 4. Monitoring Stack - Prometheus + Grafana

Deploy monitoring stack lengkap menggunakan Docker Compose yang terdiri dari tiga komponen: Prometheus sebagai time-series database untuk mengumpulkan metrics, Node Exporter sebagai agent yang mengekspos metrics sistem (CPU, memory, disk), dan Grafana sebagai dashboard visualisasi.

**Arsitektur:**

```
Node Exporter (9100) --> Prometheus (9090) --> Grafana (3001)
```

Prometheus melakukan scraping metrics dari Node Exporter setiap 15 detik, lalu Grafana membaca data dari Prometheus dan menampilkannya dalam bentuk grafik interaktif.

**Konfigurasi `prometheus.yml`:**

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node-exporter'
    static_configs:
      - targets: ['node-exporter:9100']
```

**Konfigurasi `docker-compose.yml`:**

```yaml
version: '3'
services:
  prometheus:
    image: prom/prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    depends_on:
      - node-exporter

  node-exporter:
    image: prom/node-exporter
    ports:
      - "9100:9100"

  grafana:
    image: grafana/grafana
    ports:
      - "3001:3000"
    environment:
      GF_SECURITY_ADMIN_PASSWORD: your_password_here
    depends_on:
      - prometheus
```

**Dashboard yang dibuat:**
- Panel "CPU Usage" menggunakan query `node_cpu_seconds_total`
- Panel "Memory Available" menggunakan query `node_memory_MemAvailable_bytes`

---

## Ringkasan Port Forwarding

| Service | Host Port | Guest Port | Protokol |
|---------|-----------|-----------|----------|
| Nginx | 8080 | 8080 | TCP |
| WordPress | 8081 | 8081 | TCP |
| Prometheus | 9090 | 9090 | TCP |
| Node Exporter | 9100 | 9100 | TCP |
| Grafana (Monitoring) | 3001 | 3001 | TCP |

---

## Screenshots

Semua bukti dokumentasi tersedia di folder [`/screenshots`](./screenshots).

---

## Tools yang Digunakan

| Tools | Fungsi |
|-------|--------|
| Oracle VirtualBox 7.x | Virtualisasi untuk menjalankan Ubuntu Server |
| Ubuntu Server 24.04 LTS | Sistem operasi di dalam VM |
| Docker | Container runtime |
| Docker Compose | Orkestrasi multi-container |
| Nginx | Web server |
| WordPress + MySQL 5.7 | Content Management System (CMS) |
| Prometheus | Time-series database untuk metrics |
| Node Exporter | Agent pengumpul metrics sistem |
| Grafana | Dashboard monitoring dan visualisasi |

---

## Tentang

Repository ini dibuat sebagai dokumentasi proses belajar provisioning virtual machine, container, dan tools monitoring yang umum digunakan dalam pengelolaan infrastruktur cloud.
