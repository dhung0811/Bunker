## Use Bunkerweb behind LB (HAproxy)
### Install Bunkerweb with Docker compose:
Đây là file dockercompse:
```
x-bw-env: &bw-env
  # We use an anchor to avoid repeating the same settings for both services
  API_WHITELIST_IP: "127.0.0.0/8 10.20.30.0/24" # Make sure to set the correct IP range so the scheduler can send the configuration to the instance
  DATABASE_URI: "mariadb+pymysql://bunkerweb:changeme@bw-db:3306/db" # Remember to set a stronger password for the database

services:
  bunkerweb:
    # This is the name that will be used to identify the instance in the Scheduler
    image: bunkerity/bunkerweb:1.6.4
    ports:
      - "8081:8080/tcp"
      - "8443:8443/tcp"
      - "8443:8443/udp" # For QUIC / HTTP3 support
    environment:
      <<: *bw-env # We use the anchor to avoid repeating the same settings for all services
      USE_REAL_IP: "yes"
      USE_PROXY_PROTOCOL: "yes"
      REAL_IP_FROM: "1.2.3.0/24 100.64.0.0/10"
      REAL_IP_HEADER: "proxy_protocol"
    restart: "unless-stopped"
    networks:
      - bw-universe
      - bw-services

  bw-scheduler:
    image: bunkerity/bunkerweb-scheduler:1.6.4
    environment:
      <<: *bw-env
      BUNKERWEB_INSTANCES: "bunkerweb" # Make sure to set the correct instance name
      SERVER_NAME: ""
      MULTISITE: "yes"
      UI_HOST: "http://bw-ui:7000" # Change it if needed
      USE_REDIS: "yes"
      REDIS_HOST: "redis"
      USE_REAL_IP: "yes"
      USE_PROXY_PROTOCOL: "yes"
      REAL_IP_FROM: "1.2.3.0/24 100.64.0.0/10"
      REAL_IP_HEADER: "proxy_protocol"
    volumes:
      - bw-storage:/data # This is used to persist the cache and other data like the backups
    restart: "unless-stopped"
    networks:
      - bw-universe
      - bw-db

  bw-ui:
    image: bunkerity/bunkerweb-ui:1.6.4
    environment:
      <<: *bw-env
    restart: "unless-stopped"
    networks:
      - bw-universe
      - bw-db

  bw-db:
    image: mariadb:11
    # We set the max allowed packet size to avoid issues with large queries
    command: --max-allowed-packet=67108864
    environment:
      MYSQL_RANDOM_ROOT_PASSWORD: "yes"
      MYSQL_DATABASE: "db"
      MYSQL_USER: "bunkerweb"
      MYSQL_PASSWORD: "changeme" # Remember to set a stronger password for the database
    volumes:
      - bw-data:/var/lib/mysql
    restart: "unless-stopped"
    networks:
      - bw-db

  redis: # Redis service for the persistence of reports/bans/stats
    image: redis:7-alpine
    command: >
      redis-server
      --maxmemory 256mb
      --maxmemory-policy allkeys-lru
      --save 60 1000
      --appendonly yes
    volumes:
      - redis-data:/data
    restart: "unless-stopped"
    networks:
      - bw-universe

volumes:
  bw-data:
  bw-storage:
  redis-data:



networks:
  bw-universe:
    name: bw-universe
    ipam:
      driver: default
      config:
        - subnet: 10.20.30.0/24 # Make sure to set the correct IP range so the scheduler can send the configuration to the instance
  bw-services:
    name: bw-services
  bw-db:
    name: bw-db
```
Ban đầu, khi em setup em chưa có các config như:
```
      USE_REAL_IP: "yes"
      USE_PROXY_PROTOCOL: "yes"
      REAL_IP_FROM: "1.2.3.0/24 100.64.0.0/10"
      REAL_IP_HEADER: "proxy_protocol"
```

Em vẫn access được vào UI từ PC của em: 

![UI](./imgs/UI.png)

Sau đó em bật REAL_IP và PROXY_PROTOCOL ở đây:
![](./imgs/realIP_Proxy.png)

Config HAproxy: 
```
global
  log 127.0.0.1 local0
  maxconn 2000
  daemon

defaults
  log     global
  option  dontlognull
  mode    tcp
  timeout connect 5s
  timeout client  1m
  timeout server  1m

frontend fe_http
  bind *:80
  mode tcp
  default_backend be_http

backend be_http
  mode tcp
  server bw_http 172.31.100.154:8081 send-proxy check

frontend fe_https
  bind *:443
  mode tcp
  default_backend be_https

backend be_https
  mode tcp
  server bw_https 172.31.100.154:8443 send-proxy check
```

Sau khi apply và restart lại haproxy, em thử access từ PC của em thì nhận broken header:

![fromHost](./imgs/curlformHost.png)
![broken](./imgs/brokenheader.png)

Nhưng khi em curl từ Devs, thì traffic lại đi đúng flow:
HAproxy &rarr; Bunkerweb

![formDevs](./imgs/curlfromDevs.png)

Bunker đã nhận traffic:

![receive](./imgs/receive.png)