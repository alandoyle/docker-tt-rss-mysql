
> **IMPORTANT NOTE:** MySQL support was removed from TT-RSS on 28th April 2025 (https://git.tt-rss.org/fox/tt-rss.git/commit/?id=4cb8a84df46d46bc325b6638defbdc4dc34151ed)

> The latest version of this image in [Github](https://github.com/alandoyle/docker-tt-rss-mysql) and [DockerHub](https://hub.docker.com/r/alandoyle/tt-rss-mysql) is (https://git.tt-rss.org/fox/tt-rss.git/commit/?id=0e4b8bd6538f3062d34a3a06ab5531c70042de78) which is the *last* version to support MySQL.

---

# Docker container for TT-RSS (MySQL)
[![Docker Image Size](https://img.shields.io/docker/image-size/alandoyle/tt-rss-mysql/latest?logo=docker&style=for-the-badge)](https://hub.docker.com/r/alandoyle/tt-rss-mysql/tags)
[![Docker Pulls](https://img.shields.io/docker/pulls/alandoyle/tt-rss-mysql?label=Pulls&logo=docker&style=for-the-badge)](https://hub.docker.com/r/alandoyle/tt-rss-mysql)
[![Source](https://img.shields.io/badge/Source-GitHub-blue?logo=github&style=for-the-badge)](https://github.com/alandoyle/docker-tt-rss-mysql)

This is a Docker container for [TT-RSS (MySQL)](https://tt-rss.org/).

---

[![TT-RSS (MySQL) logo](https://images.weserv.nl/?url=raw.githubusercontent.com/alandoyle/docker-tt-rss-mysql/main/TT-RSS-logo.png&w=110)](https://tt-rss.org/)[![TT-RSS (MySQL)](https://images.placeholders.dev/?width=420&height=110&fontFamily=monospace&fontWeight=400&fontSize=52&text=TT-RSS (MySQL)&bgColor=rgba(0,0,0,0.0)&textColor=rgba(121,121,121,1))](https://tt-rss.org/)

A simple Tiny Tiny RSS image which only supports MySQL with integrated feed updates.

+ Support MySQL server.
+ Built in Feed updating.
+ Built-in TT-RSS updating.

---

## IMPORTANT NOTES

This Docker image has several assumptions/prerequisites which need to be  fulfilled, ignoring them *will* bring failure.

1. This Docker Image is for MySQL _ONLY_.
1. This image is for domains or sub-domains with **/tt-rss/** in the URL. e.g. **http://reader.mydomain.tld**
1. MySQL needs to be installed in a separate Docker container.
1. MySQL needs to be configured and setup **BEFORE** this image is deployed (explained below).
1. If a previous MySQL instance is used and and old TT-RSS instance was using PHP 7.x then a "Data Fix" will need to be applied to the database as this image used PHP 8.3 and generates JSON differently. Failure to "Data Fix" the database could result in duplicate posts appearing in TT-RSS (explained below)

## MySQL Setup

This image _requires_ a database and database user be set up **PRIOR** to bringing up the image.

A new database and user can be created using the _mysql_ command.

e.g. If you're running the MySQL container from the Docker Compose example below you will need to run the following commands to create the database and user.

```bash
docker exec -it <MYSQL_CONTAINER> /bin/bash
```
Once inside the container run the following command to access MySQL.
```bash
mysql -u root -p
```
You will be prompted for the MYSQL_ROOT_PASSWORD (see Docker Compose example)
Once in _mysql_ run the following SQL commands to set up the database and user (see Docker Compose example to match up the values).
```sql
CREATE DATABASE <TTRSS_DB_NAME>;
CREATE USER '<TTRSS_DB_USER>' IDENTIFIED BY '<TTRSS_DB_PASS>';
GRANT ALL ON <TTRSS_DB_NAME>.* TO '<TTRSS_DB_USER>';
FLUSH PRIVILEGES;
\q
```

Now the `tt-rss-mysql` image can be started.

## PHP 7 -> PHP 8 "Data Fix"

**NOTE:** If this is a fresh install of Tiny Tiny RSS then ignore this section.

PHP 7 stores the unique GUID used by each article in the following format:
```
{"ver":2,"uid":"2","hash":"SHA1:2b10b494802dc70e9d9d7676cdef0cf0f9969b78"}
```

PHP 8 stores the unique GUID used by each article in the following format:
```
{"ver":2,"uid":2,"hash":"SHA1:2b10b494802dc70e9d9d7676cdef0cf0f9969b78"}
```

Notice that the "uid" value is quoted with PHP 7 but not PHP 8.

### The fix
To fix a previous MySQL database populated by PHP 7 the following SQL commands need to be used via the _mysql_ commandline tool.
```sql
USE <TTRSS-DATABASE>;

UPDATE ttrss_entries
SET guid = replace(replace(guid,'"uid":"', '"uid":'),'", "hash":', ',"hash":')
WHERE guid LIKE '%"uid":"%"%';
```
----

## Docker

Available on [DockerHub](https://hub.docker.com/r/alandoyle/tt-rss-mysql)
```bash
docker pull alandoyle/tt-rss-mysql
```

## Usage

```bash
docker run --name=tt-rss-mysql \
  -d --init \
  -v <MY_CONF_PATH>:/opt/tt-rss/config.d\
  -v <MY_WEB_PATH>:/var/www/tt-rss\
  -p 8000:80/tcp \
  alandoyle/tt-rss-mysql:latest
```

Docker compose example:

```yaml
services:
  mysql:
    image: mysql:8.0
    container_name: mysql
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: SecureSecretPassword
    volumes:
      - ./mysql/data:/var/lib/mysql
  tt-rss-mysql:
   image: alandoyle/tt-rss-mysql:latest
   container_name: tt-rss-mysql
   restart: unless-stopped
   init: true
   ports:
     - "8000:80/tcp"
    volumes:
      - ./tt-rss/web:/var/www/tt-rss
      - ./tt-rss/config:/opt/tt-rss/config.d
    environment:
      TTRSS_SELF_URL_PATH: https://reader.mydomain.tld
      TTRSS_DB_HOST: mysql
      TTRSS_DB_USER: ttrss_user
      TTRSS_DB_NAME: ttrss_database
      TTRSS_DB_PASS: ttrss_password
      TTRSS_DB_PORT: 3306
      TTRSS_FEED_UPDATE_CHECK: 600
```

### Ports

| Port     | Description           |
|----------|-----------------------|
| `80/tcp` | HTTP                  |

### Volumes

| Path    | Description                           |
|---------|---------------------------------------|
| `/var/www/tt-rss` | path for tt-rss web files |
| `/opt/tt-rss/config.d` | path for tt-rss configuration files          |
