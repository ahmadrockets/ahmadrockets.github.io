---
title: "Nginx & SSL (Lets Encrypt) Installation & Configuration"
description: "This post will explain how to install & configure Nginx with ssl lets Encrypt"
publishDate: "03 Jul 2025"
updatedDate: "03 Jul 2025"
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