---
title: "Nginx & SSL (Lets Encrypt) Installation & Configuration"
description: "This post will explain how to install & configure Nginx with ssl lets Encrypt"
publishDate: "03 Jul 2025"
updatedDate: "04 Jul 2025"
tags: ["nginx", "server", "ssl", "lets encrypt"]
---

## A. Konfigurasi nginx (tanpa docker)

1. Install Nginx
    ```bash
    sudo apt update
    sudo apt install nginx
    ```
2. Check nginx
    ```bash
    sudo systemctl status nginx
    ```
    jika belum aktif, jalankan
    ```bash
    sudo systemctl start nginx
    ```
2. Konfigurasi nginx misal mau menambahkan domain grafana.domainku.com
    ```bash
    sudo nano /etc/nginx/sites-available/grafana.domainku.com
    ```
    kemudian isi file konfigurasi tersebut dengan code seperti dibawah ini:
    ```
    server {
        server_name grafana.domainku.com;

        location / {
            proxy_pass http://localhost:3000;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
    ```
3. Aktifkan konfigurasi
    ```bash
    sudo ln -s /etc/nginx/sites-available/grafana.domainku.com /etc/nginx/sites-enabled/
    ```
4. Cek konfigurasi nginx
    ```bash
    sudo nginx -t
    ```
5. Restart nginx
    ```bash
    sudo systemctl restart nginx
    ```

## B. Konfigurasi SSL Let's Encrypt

1. Install certbot generator
    ```bash
    sudo apt update
    sudo apt install certbot python3-certbot-nginx
    ```
2. Generate certbot
    ```bash
    sudo certbot --nginx -d grafana.domainku.com
    ```
3. Check konfigurasi nginx
    ```bash
    sudo nginx -t
    ```
4. Restart nginx
    ```bash
    sudo systemctl reload nginx
    ```

## C. Setting nginx monitoring ke grafana prometheus
1. Install nginx-prometheus-exporter
    ```bash
    # Download nginx exporter
    wget https://github.com/nginx/nginx-prometheus-exporter/releases/download/v1.4.2/nginx-prometheus-exporter_1.4.2_linux_arm64.tar.gz
    # Extract nginx exporter
    tar -xzf nginx-prometheus-exporter_1.4.2_linux_arm64.tar.gz
    # Install nginx exporter
    sudo mv nginx-prometheus-exporter /usr/local/bin/
    ```

2. Buat config untuk nginx exporternya
    ```bash
    sudo tee /etc/systemd/system/nginx-exporter.service > /dev/null <<EOF
    [Unit]
    Description=Nginx Prometheus Exporter
    After=network.target

    [Service]
    Type=simple
    ExecStart=/usr/local/bin/nginx-prometheus-exporter -nginx.scrape-uri=http://localhost/nginx_status
    Restart=always
    User=nobody

    [Install]
    WantedBy=multi-user.target
    EOF
    ```

3. Enable dan start service nginx exporter
    ```bash
    sudo systemctl daemon-reload
    sudo systemctl enable nginx-exporter
    sudo systemctl start nginx-exporter
    ```

4. Buat config nginx status
    ```bash
    nano /etc/nginx/sites-available/nginx_status
    ```
    Isi dengan
    ```bash
    server {
        listen 80;
        server_name localhost;
        
        location /nginx_status {
            stub_status on;
            access_log off;
            allow 127.0.0.1;
            allow ::1;
            deny all;
        }
        
        location / {
            return 404;
        }
    }
    ```
    Aktifkan Konfigurasinya
    ```bash
    sudo ln -s /etc/nginx/sites-available/nginx_status /etc/nginx/sites-enabled/
    ```
5. Restart nginx
    ```bash
    sudo systemctl restart nginx
    ```
    
6. Tambahkan job untuk nginx exporter di file config `prometheus.yaml`
    ```bash
    - job_name: 'nginx'
      static_configs:
        - targets: ['172.17.0.1:9113']
      scrape_interval: 20s
      metrics_path: /metrics
    ```
7. Restart service/container prometheus
8. Import Dashboard 
    - Go to Create → Import
    - Import dashboard dengan ID: 12708 (Nginx Prometheus Exporter)