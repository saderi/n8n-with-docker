# Self host n8n deploy with Docker Compose
This repository contains a Docker Compose configuration to deploy n8n. 

## Prerequisites
- Docker and Docker Compose installed on your server.
- Basic knowledge of Docker and Docker Compose.
- A DNS record for your domain pointing to your server's IP address for webhook functionality.
- A reverse proxy (like Nginx or Traefik) to handle HTTPS and route traffic to n8n.


## Setup Instructions
Clone this repository and create a `.env` file based on the `.env.example` file provided, and customize it according to your needs.

Becuase we need vector extension in postgres, we are building a custom postgres image using the provided Dockerfile.

Start the services using Docker Compose:
```bash
docker compose build postgres
docker compose up -d
```

## Nginx Reverse Proxy Configuration and SSL
To secure your n8n instance with HTTPS, you can use Nginx as a reverse proxy. Make sure to replace `YOUR_DOMAIN.COM` with your actual domain and adjust the proxy settings as needed.

```nginx
server {
    server_name YOUR_DOMAIN.COM;
    listen 80;
    # This is for certbot verification
    location ~ /.well-known/acme-challenge/ {
        allow all;
        root /var/www/html;
    }
    location / {
        client_max_body_size 15M;
        proxy_pass http://127.0.0.1:5678; # YOUR n8n instance address
        proxy_http_version 1.1;
        proxy_set_header Upgrade         $http_upgrade;
        proxy_set_header Connection      'upgrade';
        proxy_set_header X-Real-Ip       $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host            $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```
