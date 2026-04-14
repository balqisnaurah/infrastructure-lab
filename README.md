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
| Storage | 40 GB (LVM) |
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

### 5. ELK Stack - Elasticsearch + Kibana

Deploy Elasticsearch dan Kibana menggunakan Docker Compose untuk eksplorasi centralized logging. Elasticsearch berfungsi sebagai mesin pencarian dan penyimpanan log, sedangkan Kibana menyediakan antarmuka web untuk visualisasi dan eksplorasi data.

**Arsitektur:**

```
Data Log --> Elasticsearch (9200) --> Kibana (5601)
```

**Konfigurasi `docker-compose.yml`:**

```yaml
version: '3'
services:
  elasticsearch:
    image: elasticsearch:7.17.10
    environment:
      - discovery.type=single-node
      - "ES_JAVA_OPTS=-Xms256m -Xmx256m"
    ports:
      - "9200:9200"
    mem_limit: 512m

  kibana:
    image: kibana:7.17.10
    ports:
      - "5601:5601"
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
    depends_on:
      - elasticsearch
```

**Proses yang dilakukan:**
1. Menjalankan Elasticsearch dan Kibana menggunakan Docker Compose
2. Memasukkan data log contoh ke Elasticsearch menggunakan REST API (curl)
3. Membuat index pattern `server-logs*` di Kibana
4. Melihat data log melalui Discover di Kibana
5. Membuat visualisasi "Log Level Distribution" menggunakan Lens

**Contoh insert data log via curl:**

```bash
curl -X POST "localhost:9200/server-logs/_doc" -H "Content-Type: application/json" -d '{
  "timestamp": "2026-04-14T08:05:33",
  "level": "ERROR",
  "message": "Database connection timeout",
  "service": "api-service"
}'
```

---

### 6. Custom Dockerfile

Membuat Docker image sendiri yang berisi script monitoring kesehatan sistem. Menunjukkan pembuatan image dari Dockerfile, bukan hanya menjalankan image bawaan.

**Dockerfile:**

```dockerfile
FROM ubuntu:24.04

RUN apt-get update && apt-get install -y \
    curl \
    iproute2 \
    procps \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /scripts

COPY backup.sh .
COPY system-health-check.sh .

RUN sed -i 's/\r$//' backup.sh system-health-check.sh && \
    chmod +x backup.sh system-health-check.sh

CMD ["bash", "./system-health-check.sh"]
```

**Perintah yang digunakan:**

```bash
docker build -t system-health-check .
docker run --name health-check system-health-check
```

Hasil: script berjalan di dalam container yang terisolasi, menampilkan informasi sistem, CPU, memory, disk, dan status jaringan dari perspektif container.

---

## Ringkasan Port Forwarding

| Service | Host Port | Guest Port | Protokol |
|---------|-----------|-----------|----------|
| SSH | 2222 | 22 | TCP |
| Nginx | 8080 | 8080 | TCP |
| WordPress | 8081 | 8081 | TCP |
| Prometheus | 9090 | 9090 | TCP |
| Node Exporter | 9100 | 9100 | TCP |
| Grafana (Monitoring) | 3001 | 3001 | TCP |
| Elasticsearch | 9200 | 9200 | TCP |
| Kibana | 5601 | 5601 | TCP |

---

## Screenshots

Semua bukti dokumentasi tersedia di folder [`/screenshots`](./screenshots).

---

## Tools yang Digunakan

| Tools | Fungsi |
|-------|--------|
| Oracle VirtualBox 7.2 | Virtualisasi untuk menjalankan Ubuntu Server |
| Ubuntu Server 24.04 LTS | Sistem operasi di dalam VM |
| Docker | Container runtime |
| Docker Compose | Orkestrasi multi-container |
| Nginx | Web server |
| WordPress + MySQL 5.7 | Content Management System (CMS) |
| Prometheus | Time-series database untuk metrics |
| Node Exporter | Agent pengumpul metrics sistem |
| Grafana | Dashboard monitoring dan visualisasi |
| Elasticsearch 7.17.10 | Mesin pencarian dan penyimpanan log |
| Kibana 7.17.10 | Visualisasi dan eksplorasi data log |

---

## Tentang

Repository ini dibuat sebagai dokumentasi proses belajar provisioning virtual machine, container, monitoring stack, dan centralized logging yang umum digunakan dalam pengelolaan infrastruktur cloud.
