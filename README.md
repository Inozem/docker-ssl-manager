# Docker SSL Manager

This plugin automatically renews SSL certificates using Certbot and restarts the Nginx container (`nginx_https`). The plugin is built using Python and Docker and includes a schedule for automatic certificate renewal.

## Installation and Usage

### Step 1: Docker Compose Setup

Сreate a Docker Compose file with the required structure. Here is an example:

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
> **Don't forget to add your application's container. You'll also need to add the Let's Encrypt and Nginx files. You can copy the Let's Encrypt files and find examples of Nginx files from [this repository](https://github.com/Inozem/letsencrypt-nginx-docker-compose).**


### Step 2: Adding .env file
In the same directory, add an .env file with the variables EMAIL and DOMAIN to configure the certificate:
```.env
EMAIL=example@gmail.com
DOMAIN=example.com
```


### Step 3: Running the Setup
Once all containers and configurations are in place, simply run:

```bash
docker-compose up -d
```
This will start all services, automatically install SSL certificates, and schedule their renewal every two months.



## Configuring Certificate Renewal Schedule
By default, the certificates are renewed on the 1st of every two months. However, you can modify the schedule by specifying different values for the variables in your `docker-compose.yml`.

### Example: Setting the Schedule to Every 15 Days
To change the schedule to renew the certificates every 15 days, adjust your `docker-compose.yml` like this:

```yaml
docker_ssl_manager:
  image: inozem/docker_ssl_manager:v1.0
  environment:
    EMAIL: "example_email@gmail.com"
    DOMAIN: "example_docker.com"
    RENEW_MINUTE: "0"
    RENEW_SECOND: "0"
    RENEW_HOUR: "3"
    RENEW_DAY: "15"
    RENEW_DAY_OF_WEEK: "*"
    RENEW_MONTH: "*"
  volumes:
    - /var/run/docker.sock:/var/run/docker.sock
    - ./certbot_data:/var/www/certbot
    - ./letsencrypt:/etc/letsencrypt
  depends_on:
    - nginx_https
```
### Explanation of Possible Environment Variable Values

- **`EMAIL`**: Email address where Certbot will send notifications about the certificate status.
- **`DOMAIN`**: Domain for which SSL certificates should be generated and renewed.
- **`RENEW_MINUTE`**: Specifies the minute when the renewal should start.  
  - **Values**: From `0` to `59`. Default is `0` (start of the hour).
- **`RENEW_SECOND`**: Specifies the second when the renewal should start.  
  - **Values**: From `0` to `59`. Default is `0`.
- **`RENEW_HOUR`**: Specifies the hour when the certificate renewal task should run.  
  - **Values**: From `0` to `23`. Default is `3` (3:00 AM).
- **`RENEW_DAY`**: Specifies the day of the month when the renewal will occur.  
  - **Values**: From `1` to `31`, or `*` (any day). Default is `1`.
- **`RENEW_DAY_OF_WEEK`**: Specifies the day of the week when the renewal will occur.  
  - **Values**: From `0` (Sunday) to `6` (Saturday), or `*` to run the task on any day of the week.
- **`RENEW_MONTH`**: Specifies the month when the renewal will occur.  
  - **Values**: From `1` to `12`, `*` for any month, or something like `*/2` to run every two months.

### Example Variable Configurations

- **To run the task every day**:  
  - Set `RENEW_DAY="*"` and `RENEW_DAY_OF_WEEK="*"`
  
- **To run the task only on Mondays**:  
  - Set `RENEW_DAY_OF_WEEK="1"`

- **To run the task every 3 months**:  
  - Set `RENEW_MONTH="*/3"`
