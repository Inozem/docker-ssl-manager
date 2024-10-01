# Docker SSL Manager

This plugin automatically renews SSL certificates using Certbot and restarts the Nginx container (`nginx_https`). The plugin is built using Python and Docker and includes a schedule for automatic certificate renewal.

## Installation and Usage

### Step 1: Docker Compose Setup

1. Сreate a Docker Compose file with the required structure. Here is an example:

```yaml
services:
  nginx_http:
    image: nginx:alpine
    container_name: nginx_http
    volumes:
      - ./nginx/nginx.http.conf:/etc/nginx/conf.d/default.conf
      - ./certbot_data:/var/www/certbot
    ports:
      - "80:80"

  certbot:
    container_name: certbot
    image: certbot/certbot:latest
    command: >- 
             certonly --webroot --webroot-path=/var/www/certbot
             --email ${EMAIL} --agree-tos --no-eff-email
             -d ${DOMAIN} --non-interactive --quiet
    volumes:
      - ./letsencrypt:/etc/letsencrypt
      - ./certbot_data:/var/www/certbot
    depends_on:
      - nginx_http

  nginx_https:
    image: nginx:alpine
    container_name: nginx_https
    volumes:
      - ./nginx/nginx.https.conf:/etc/nginx/conf.d/default.conf
      - ./nginx/proxy_params:/etc/nginx/proxy_params
      - ./letsencrypt:/etc/letsencrypt
    ports:
      - "443:443"
    depends_on:
      certbot:
        condition: service_completed_successfully

  docker_ssl_manager:
    image: inozem/docker_ssl_manager:v1.0
    environment:
      EMAIL: "example_email@gmail.com"
      DOMAIN: "example_docker.com"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./certbot_data:/var/www/certbot
      - ./letsencrypt:/etc/letsencrypt
    depends_on:
      - nginx_https
```
> **Don't forget to add your application's container.**
