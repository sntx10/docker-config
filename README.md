# Nginx Docker Project with SSL and Non-SSL Configurations

This project sets up a web application using Nginx, Docker, and Docker Compose. It includes configurations for both non-SSL (HTTP) and SSL/TLS (HTTPS) setups, supporting a Django backend, a frontend, PostgreSQL database, Redis, Celery, and Certbot for SSL certificates. The project is designed to serve a domain with reverse proxy capabilities.

---

## 📑 Table of Contents

- [Prerequisites](#prerequisites)
- [Project Structure](#project-structure)
- [Setup Instructions](#setup-instructions)
- [Nginx Configuration](#nginx-configuration)
- [SSL Setup](#ssl-setup)
- [Docker Compose Configuration](#docker-compose-configuration)
- [Running the Project](#running-the-project)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)

---

## ✅ Prerequisites

Before starting, ensure you have the following installed:

- Docker
- Docker Compose
- A registered domain (required for SSL setup)
- Certbot for SSL certificates (if using Let's Encrypt)
- npm (for frontend development)

---

## 🗂️ Project Structure

```bash New structure
 DockerConfig/ 🌐
+├── docker-config/ 📂
+|   ├── no-ssl/ 🚫🔒
+|   │   ├── docker-compose.prod.yml  🎛️ # Docker Compose configuration for non-SSL production environment
+|   │   ├── Dockerfile.prod          🐍 # Dockerfile for Django backend (non-SSL setup)
+|   │   └── nginx/ 🌐
+|   │       ├── Dockerfile           🐳 # Dockerfile for Nginx (non-SSL configuration)
+|   │       └── nginx.conf           ⚙️ # Nginx configuration file for non-SSL setup
+|   ├── ssl/ ✅🔒
+|   │   ├── docker-compose.prod.yml  🎛️ # Docker Compose configuration for SSL production environment
+|   │   ├── Dockerfile.prod          🐍 # Dockerfile for Django backend (SSL setup)
+|   │   └── nginx/ 🌐
+|   │       ├── Dockerfile           🐳 # Dockerfile for Nginx (SSL configuration)
+|   │       └── nginx.conf           ⚙️ # Nginx configuration file for SSL setup
+└── README.md                        📝 # Project documentation and guide
```

---

## ⚙️ Setup Instructions

1. **Clone the repository:**
   ```bash
   git clone https://github.com/yourusername/DockerConfig.git
   cd DockerConfig

    Choose your setup:

        For non-SSL (HTTP only):
        cd docker-config/no-ssl

        For SSL (HTTPS):
        cd docker-config/ssl

    Create a .env file with the following variables:

    DB_NAME=your_db_name
    DB_USERNAME=your_db_username
    DB_PASSWORD=your_db_password
    DJANGO_SETTINGS_MODULE=config.settings

    Configure your domain (for SSL):

        Update your domain's DNS records (e.g., A record) to point to your server's IP.

    Prepare SSL certificates (see SSL Setup).

    Customize Nginx configuration (see Nginx Configuration).

🧩 Nginx Configuration
🔓 Non-SSL (docker-config/no-ssl/nginx/nginx.conf)

server {
    listen 80;
    server_name backend;

    client_max_body_size 1000M;

    location /static/ {
        alias /app/staticfiles/;
    }

    location /media/ {
        alias /app/media/;
    }

    location / {
        proxy_pass http://backend:80;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}

🔐 SSL (docker-config/ssl/nginx/nginx.conf)

# Redirect HTTP to HTTPS
server {
    listen 80;
    server_name yourdomain.com;

    location ^~ /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }

    location / {
        return 301 https://$host$request_uri;
    }
}

# Backend
server {
    listen 443 ssl;
    server_name yourdomain.com;

    ssl_certificate     /etc/letsencrypt/live/yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem;

    ssl_protocols       TLSv1.2 TLSv1.3;
    ssl_ciphers         HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;

    client_max_body_size 100M;

    location /static/ {
        alias /app/staticfiles/;
    }

    location /media/ {
        alias /app/media/;
    }

    location / {
        proxy_pass http://backend:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

# Frontend
server {
    listen 443 ssl;
    server_name yourdomain.com;

    ssl_certificate     /etc/letsencrypt/live/yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem;

    ssl_protocols       TLSv1.2 TLSv1.3;
    ssl_ciphers         HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;

    location / {
        proxy_pass http://frontend:90;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}

🔒 SSL Setup

    Navigate to the docker-config/ssl/ directory.

    Run Certbot to generate certificates:

docker-compose -f docker-compose.prod.yml up certbot

Follow the prompts to generate certs for yourdomain.com.

Certificates will be located at:

    docker-config/ssl/certbot/conf/live/yourdomain.com/

🐳 Docker Compose Configuration
🔓 Non-SSL

See docker-config/no-ssl/docker-compose.prod.yml.
🔐 SSL

See docker-config/ssl/docker-compose.prod.yml.

Both configurations include services for:

    Django backend (web or backend)

    PostgreSQL database

    Redis

    Celery worker and beat

    Nginx reverse proxy

    Certbot (for SSL)

    Frontend (optional)

▶️ Running the Project

    Navigate to the appropriate directory:

cd docker-config/no-ssl  # or cd docker-config/ssl

Build and start services:

docker-compose -f docker-compose.prod.yml up -d --build

Verify:

docker-compose -f docker-compose.prod.yml ps

Access the app:

    Non-SSL: http://yourdomain.com

    SSL: https://yourdomain.com

Stop the containers:

    docker-compose -f docker-compose.prod.yml down

🛠️ Troubleshooting

    Nginx fails to start:
    Run docker exec -it nginx nginx -t to check config.

    SSL errors:
    Ensure certificate paths are correct.

    Domain not resolving:
    Check DNS records.

    Port conflicts:
    Ensure ports 80, 443, 5432, 6379, 8000 are free.

    Database connection issues:
    Double-check .env credentials.

    Permission issues:
    Nginx user must have access to mounted certs/static.

🤝 Contributing

Contributions are welcome! Follow these steps:

    Fork the repository

    Create a new branch:
    git checkout -b feature/your-feature

    Commit your changes:
    git commit -m "Add your feature"

    Push to GitHub:
    git push origin feature/your-feature

    Open a pull request
