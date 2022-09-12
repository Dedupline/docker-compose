![image1](https://workspace.dedupline.ml/apps/sharingpath/lance/CDN/Banner.png)
# Docker Compose Boilerplates [Multi-Architecture Images]
Compose is a tool for defining and running multi-container Docker applications. With Compose, you use a YAML file to configure your application's services.

## Prerequisites

- It's assumed that you have basic Docker and container knwoledge.
- It's assumed that you have basic Linux command-line knowledge.
- It's assumed that you have set up your Linux machine on-premise or in the cloud.

## Server Hardware Requirements (optional)

The configurations were tested on an Ubuntu Linux 20.04 LTS with ARM-based architecture, hosted on Oracle Cloud Infrastructure.

<table>
  <tr>
    <th>Shape</th>
    <th>OCPU Count</th>
    <th>Memory (GB)</th>
  </tr>
  <tr>
    <td>VM.Standard.A1.Flex</td>
    <td>4</td>
    <td>24</td>
  </tr>
</table>

### Install Docker Engine on Linux
To install Docker Engine on Linux, please follow this documentation provided by Docker.
<br>
Here is the link: https://docs.docker.com/engine/install/ubuntu/

### Install Portainer
To use the boilerplates easily, you may install Portainer. First, create the volume that Portainer Server will use to store its database:

```
docker volume create portainer_data
```

Then, download and install the Portainer Server container:

```
docker run -d -p 8000:8000 -p 9443:9443 --name portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce:latest
```

Once the installation is done, go to **`Stack`** tab, press the **`Add stack`** button, and copy/paste the configuration below that you choose.

![image](https://user-images.githubusercontent.com/58999917/177238677-41e2cb1b-dbb6-4e52-9a03-81caa694fa67.png)

# Available Images
- AdGuard Home
- Firefly III
- Ghost CMS
- Plex
- SearXNG
- Vaultwarden
- WordPress

Releasing more soon

## AdGuard Home
```
version: '3.9'

volumes:
  work_data:
  conf_data:
  
networks:
  default:
    name: main
    driver: bridge

services:

  adguard:
    image: adguard/adguardhome
    container_name: <CHANGE_NAME>
    restart: always
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "67:67/udp"
      - "<CHANGE_PORT>:68/udp"
      - "<CHANGE_PORT>:80/tcp"
      - "443:443/tcp"
      - "443:443/udp"
      - "784:784/udp"
      - "853:853/tcp"
      - "853:853/udp"
      - "3000:3000/tcp"
      - "8853:8853/udp"
      - "5443:5443/tcp"
      - "5443:5443/udp"
    volumes:
      - work_data:/opt/adguardhome/work
      - conf_data:/opt/adguardhome/conf
```

## Firefly III

To generate an `APP_KEY`, use this command: `sudo head /dev/urandom | LC_ALL=C tr -dc 'A-Za-z0-9' | head -c 32 && echo`

- To install it on an `amd64` architecture, use the tag `mysql:latest`.
- To install it on an `arm64v8` architecture, use the tag `mysql:oracle`.

```
version: '3.9'

volumes:
  app_data:
  db_data:
  
networks:
  default:
    name: firefly
    driver: bridge

services:

  firefly:
    image: fireflyiii/core:latest
    container_name: firefly_app
    restart: always
    ports:
        - <CHANGE_PORT>:8080
    environment:
      APP_KEY: <CHANGE_APP_KEY>
      DB_HOST: db
      DB_PORT: 3306
      DB_CONNECTION: mysql
      DB_DATABASE: <CHANGE_DATABASE_NAME>
      DB_USERNAME: <CHANGE_USER>
      DB_PASSWORD: <CHANGE_PASSWORD>
    volumes:
      - app_data:/var/www/html/storage/upload

  db:
    image: mysql:<tag>
    container_name: firefly_db
    restart: always
    environment:
      MYSQL_DATABASE: <CHANGE_DATABASE_NAME>
      MYSQL_USER: <CHANGE_USER>
      MYSQL_PASSWORD: <CHANGE_PASSWORD>
      MYSQL_RANDOM_ROOT_PASSWORD: '1'
    volumes:
      - db_data:/var/lib/mysql
```

## Ghost CMS

```
version: '3.9'

volumes:
  app_data:

networks:
  default:
    name: main
    driver: bridge

services:

  ghost:
    image: ghost:latest
    container_name: <CHANGE_NAME>
    restart: always
    ports:
      - "<CHANGE_PORT>:2368"
    environment:
      - URL=<YOUR_DOMAIN.TLD>
    volumes:
      - app_data:/var/lib/ghost/content
```

## Plex
Optionally you can obtain a claim token from https://plex.tv/claim and input here `PLEX_CLAIM=<REDEEM YOUR OWN CODE>`.
<br>
Keep in mind that the claim tokens expire within 4 minutes.

```
version: "3.9"
services:
  plex:
    image: lscr.io/linuxserver/plex:latest
    container_name: plex
    network_mode: bridge
    ports:
      - "<CHANGE_PORT>:32400"
    environment:
      - PUID=1000
      - PGID=1000
      - VERSION=docker
      - PLEX_CLAIM=<REDEEM YOUR OWN CODE>
    volumes:
      - /path/to/library:/config
      - /path/to/tvseries:/tv
      - /path/to/movies:/movies
    restart: unless-stopped
```

## SearXNG

```
version: '3.9'

networks:
  default:
    name: main
    driver: bridge

services:

  searxng:
    image: searxng/searxng:latest
    container_name: searxng_app
    ports:
     - "<CHANGE_PORT>:8080"
    volumes:
      - ./searxng:/etc/searxng:rw
    environment:
      - SEARXNG_BASE_URL=<YOUR_DOMAIN.TLD>
    cap_drop:
      - ALL
    cap_add:
      - CHOWN
      - SETGID
      - SETUID
      - DAC_OVERRIDE
    logging:
      driver: "json-file"
      options:
        max-size: "1m"
        max-file: "1"
```

## WordPress

- To install it on an `amd64` architecture, use the tag `mysql:latest`.
- To install it on an `arm64v8` architecture, use the tag `mysql:oracle`.

```
version: '3.9'

networks:
  default:
    name: wordpress
    driver: bridge
  
volumes:
  app_data:
  db_data:

services:

  wordpress:
    image: wordpress:latest
    container_name: wordpress_app
    restart: always
    ports:
      - <CHANGE_PORT>:80
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_NAME: <CHANGE_DATABASE_NAME>
      WORDPRESS_DB_USER: <CHANGE_USER>
      WORDPRESS_DB_PASSWORD: <CHANGE_PASSWORD>
    volumes:
      - app_data:/var/www/html

  db:
    image: mysql:<tag>
    container_name: wordpress_db
    restart: always
    environment:
      MYSQL_DATABASE: <CHANGE_DATABASE_NAME>
      MYSQL_USER: <CHANGE_USER>
      MYSQL_PASSWORD: <CHANGE_PASSWORD>
      MYSQL_RANDOM_ROOT_PASSWORD: '1'
    volumes:
      - db_data:/var/lib/mysql
```

## Issues Encountered
- If you plan to host multiple apps with database (e.g. MySQL, MariaDB, etc.), please assign those apps to its own network interface. The apps faced an issue regarding selection of database when using the same network interface.
