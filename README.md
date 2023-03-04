# Portainer container recopilation

## Portainer installation:
```
apt update && apt upgrade -y && apt install docker.io -y && apt install docker-compose -y
sudo docker volume create portainer_data
docker run -d -p 8000:8000 -p 9443:9443 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest
```
If there is “unexpected EOF” error, try:
```
sudo service docker stop
sudo dockerd --max-concurrent-downloads 1 --max-download-attempts 10
```

## Err:1 http://deb.debian.org/debian bullseye InRelease
When "exec console" in a container, it does not update, run `cd /etc` and edit the file:
```
cat > /etc/resolv.conf << EOL
nameserver 8.8.8.8
options edns0 trust-ad ndots:0
EOL
```
## 1. Rust Desk:
```
version: '3'
networks:
  rustdesk-net:
    external: false
services:
  hbbs:
    container_name: hbbs
    ports:
      - 21115:21115
      - 21116:21116
      - 21116:21116/udp
      - 21118:21118
    image: rustdesk/rustdesk-server:latest
    command: hbbs -r 100.100.100.100:21117 -k _
    volumes:
      - /home/ubuntu/rustdesk/hbbs:/root
    networks:
      - rustdesk-net
    depends_on:
      - hbbr
    restart: unless-stopped
  hbbr:
    container_name: hbbr
    ports:
      - 21117:21117
      - 21119:21119
    image: rustdesk/rustdesk-server:latest
    command: hbbr -k _
    volumes:
      - /home/ubuntu/rustdesk/hbbr:/root
    networks:
      - rustdesk-net
    restart: unless-stopped
```
> Use your public IP address

From **Portainer** exec console for **hbbr** container, and run:
```
apt update && apt install nano -y
nano id_edxxxxx.pub
```
Use this key to setup rustdesk clients.

## 2. Wordpress
```
version: '2'
services:
   db:
     container_name: wordpress_db
     image: mysql:5.7
     volumes:
       - /home/ubuntu/wordpress/db:/var/lib/mysql
     environment:
       MYSQL_ROOT_PASSWORD: password
       MYSQL_DATABASE: wordpress
       MYSQL_USER: wordpress
       MYSQL_PASSWORD: wordpress
     restart: always

   wordpress:
     container_name: wordpress_container
     image: wordpress:latest
     volumes: 
      - /home/ubuntu/wordpress/wordpress:/var/www/html
     environment:
       WORDPRESS_DB_HOST: db:3306
       WORDPRESS_DB_USER: wordpress
       WORDPRESS_DB_PASSWORD: wordpress
     depends_on:
       - db
     ports:
       - 81:80
     restart: always
      
volumes:
    db_data:
```
> Choose a strong MYSQL_ROOT_PASSWORD and WORDPRESS_DB_PASSWORD

To increase file size and memory:
```
nano php.ini
```
Add the next lines:
```
upload_max_filesize = 256M
post_max_size = 256M
php_value memory_limit 256M
```
## 3. NextCloud
```
version: '2'
volumes:
  nextcloud:
  db:
services:
  db:
    image: mariadb:10.5
    restart: always
    command: --transaction-isolation=READ-COMMITTED --binlog-format=ROW
    volumes:
      - /home/ubuntu/nextcloud/db:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD=password1
      - MYSQL_PASSWORD=password2
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
  app:
    image: nextcloud
    restart: always
    ports:
      - 82:80
    links:
      - db
    volumes:
      - /home/ubuntu/nextcloud/nextcloud:/var/www/html
    environment:
      - MYSQL_PASSWORD=password2
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
      - MYSQL_HOST=db
```
> Choose a strong MYSQL_ROOT_PASSWORD and MYSQL_PASSWORD

Once you have done the first login, from portainer exec console in nextcloud-app-1 and connect as root, then run:
```
nano config/config.php
```
Add this line in the penultimate row
```
'overwriteprotocol' => 'https',
```
Increase the memory limits `nano .htaccess` and add the next rows to the end of the file:
```
php_value memory_limit 1G
php_value upload_max_filesize 16G
php_value post_max_size 16G
php_value max_input_time 3600
php_value max_execution_time 3600
```
Restart the container

## 4. Filezilla
```
version: "2.1"
services:
  filezilla:
    image: lscr.io/linuxserver/filezilla:latest
    container_name: filezilla
    security_opt:
      - seccomp:unconfined #optional
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
    volumes:
      - /path/to/config:/config
    ports:
      - 3000:3000
    restart: unless-stopped
```
> obtain PUID and PGID running: `id -u username` and `id -g username`, where **username** is the actual server username

## 5. QBittorrent
```
version: "2.1"
services:
  qbittorrent:
    image: lscr.io/linuxserver/qbittorrent:latest
    container_name: qbittorrent
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
      - WEBUI_PORT=8080
    volumes:
      - /home/docker/QbitTorrent:/config
      - /media/docker/Torrents:/downloads
      - /media/docker/IncompleteTorrents:/incomplete
    ports:
      - 8080:8080
      - 6881:6881
      - 6881:6881/udp
restart: unless-stopped

```
> Edit the location of config, downloas and incomplete

## 6. Babybuddy
```
version: "2.1"
services:
  babybuddy:
    image: lscr.io/linuxserver/babybuddy:latest
    container_name: babybuddy
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Europe/London
      - CSRF_TRUSTED_ORIGINS=http://127.0.0.1:83,https://babybuddy.domain.com
    volumes:
      - /home/ubuntu/babybuddy:/config
    ports:
      - 83:8000
    restart: unless-stopped
```
> Edit your domain. User: admin, Password: admin
## 7. Guacamole
```
version: "3"
services:
  guacamole:
    image: abesnier/guacamole
    container_name: guacamole
    volumes:
      - /home/ubuntu/guacamole/postgres:/config
    ports:
      - 84:8080
volumes:
  postgres:
    driver: local
```
> User: guacadmnin, Password: guacadmin
