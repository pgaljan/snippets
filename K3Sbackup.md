#### Prep the VM
```shell
sudo apt-get update
sudo apt upgrade -y
sudo apt install -y curl wget
```
#### Create minio-docker.yaml
```shell
vi minio-docker.yaml
```
#### minio-docker.yaml contents
*copy function may not work with this snippet*
```yaml
---
version: '3.7'

# Settings and configurations that are common for all containers
x-minio-common: &minio-common
  image: quay.io/minio/minio:RELEASE.2023-02-10T18-48-39Z
  command: server --console-address ":9001" http://minio{1...4}/data{1...2}
  expose:
    - "9000"
    - "9001"
  environment:
    MINIO_ROOT_USER: minioadmin
    MINIO_ROOT_PASSWORD: minio123!
    #MINIO_KMS_KES_ENDPOINT: https://play.min.io:7373
    #MINIO_KMS_KES_KEY_FILE: root.key
    #MINIO_KMS_KES_CERT_FILE: root.cert
    #MINIO_KMS_KES_KEY_NAME: minio-poc-key
  healthcheck:
    test: ["CMD", "curl", "-f", "http://localhost:9000/minio/health/live"]
    interval: 30s
    timeout: 20s
    retries: 3

# starts 4 docker containers running minio server instances.
# using nginx reverse proxy, load balancing, you can access
# it through port 9000.
services:
  minio1:
    <<: *minio-common
    hostname: minio1
    volumes:
      - /appdata/minio/root.key:/root.key
      - /appdata/minio/root.cert:/root.cert
      - /appdata/minio/data1-1:/data1
      - /appdata/minio/data1-2:/data2

  minio2:
    <<: *minio-common
    hostname: minio2
    volumes:
      - /appdata/minio/root.key:/root.key
      - /appdata/minio/root.cert:/root.cert
      - /appdata/minio/data2-1:/data1
      - /appdata/minio/data2-2:/data2

  minio3:
    <<: *minio-common
    hostname: minio3
    volumes:
      - /appdata/minio/root.key:/root.key
      - /appdata/minio/root.cert:/root.cert
      - /appdata/minio/data3-1:/data1
      - /appdata/minio/data3-2:/data2

  minio4:
    <<: *minio-common
    hostname: minio4
    volumes:
      - /appdata/minio/root.key:/root.key
      - /appdata/minio/root.cert:/root.cert
      - /appdata/minio/data4-1:/data1
      - /appdata/minio/data4-2:/data2

  nginx:
    image: nginx:1.19.2-alpine
    hostname: nginx
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    ports:
      - "9000:9000"
      - "9001:9001"
    depends_on:
      - minio1
      - minio2
      - minio3
      - minio4
## By default this config uses default local driver,
## For custom volumes replace with volume driver configuration.
volumes:
  data1-1:
  data1-2:
  data2-1:
  data2-2:
  data3-1:
```
#### Create minio-nginx.conf
```shell
vi minio-nginx.conf
```
#### minio-nginx.conf contents
*copy function may not work with this snippet*
```shell
user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;

events {
    worker_connections  4096;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;
    sendfile        on;
    keepalive_timeout  65;

    # include /etc/nginx/conf.d/*.conf;

    upstream minio {
        server minio1:9000;
        server minio2:9000;
        server minio3:9000;
        server minio4:9000;
    }

    upstream console {
        ip_hash;
        server minio1:9001;
        server minio2:9001;
        server minio3:9001;
        server minio4:9001;
    }

    server {
        listen       9000;
        server_name  localhost;

        # To allow special characters in headers
        ignore_invalid_headers off;
        # Allow any size file to be uploaded.
        # Set to a value such as 1000m; to restrict file size to a specific value
        client_max_body_size 0;
        # To disable buffering
        proxy_buffering off;
        proxy_request_buffering off;

        location / {
            proxy_set_header Host $http_host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;

            proxy_connect_timeout 300;
            # Default is HTTP/1, keepalive is only enabled in HTTP/1.1
            proxy_http_version 1.1;
            proxy_set_header Connection "";
            chunked_transfer_encoding off;

            proxy_pass http://minio;
        }
    }

    server {
        listen       9001;
        server_name  localhost;

        # To allow special characters in headers
        ignore_invalid_headers off;
        # Allow any size file to be uploaded.
        # Set to a value such as 1000m; to restrict file size to a specific value
        client_max_body_size 0;
        # To disable buffering
        proxy_buffering off;
        proxy_request_buffering off;

        location / {
            proxy_set_header Host $http_host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            proxy_set_header X-NginX-Proxy true;

            # This is necessary to pass the correct IP to be hashed
            real_ip_header X-Real-IP;

            proxy_connect_timeout 300;
            
            # To support websocket
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
            
            chunked_transfer_encoding off;

            proxy_pass http://console;
        }
    }
}
```
##### Start minio
```shell
sudo docker compose up -d
```
