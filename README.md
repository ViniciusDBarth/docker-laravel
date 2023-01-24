# Docker LEMP to CakePHP

Docker to CakePHP 3x running Nginx, PHP-FPM, Composer, MySQL, PHPMyAdmin and MailHog.

## Overview

1. [Install prerequisites](#install-prerequisites)

   Before installing project make sure the following prerequisites have been met.

2. [Clone the project](#clone-the-project)

   We’ll download the code from its repository on GitHub.

3. [Configure Nginx With SSL Certificates](#configure-nginx-with-ssl-certificates) [Optional]

   We'll generate and configure SSL certificate for nginx before running server.

4. [Run the application](#run-the-application)

   By this point we’ll have all the project pieces in place.

5. [Use Makefile](#use-makefile) [Optional]

   When developing, you can use `Makefile` for doing recurrent operations.

6. [Use Docker Commands](#use-docker-commands)

   When running, you can use docker commands for doing recurrent operations.

---

## Install prerequisites

For now, this project has been mainly created for Unix `(Linux/MacOS)`. Perhaps it could work on Windows.

All requisites should be available for your distribution. The most important are :

- [Git](https://git-scm.com/downloads)
- [Docker](https://docs.docker.com/engine/installation/)
- [Docker Compose](https://docs.docker.com/compose/install/)

Check if `docker-compose` is already installed by entering the following command :

```sh
which docker-compose
```

Check Docker Compose compatibility :

- [Compose file version 3 reference](https://docs.docker.com/compose/compose-file/)

The following is optional but makes life more enjoyable :

```sh
which make
```

On Ubuntu and Debian these are available in the meta-package build-essential. On other distributions, you may need to install the GNU C++ compiler separately.

```sh
sudo apt install build-essential
```

### Images to use

- [Nginx](https://hub.docker.com/_/nginx/)
- [MySQL](https://hub.docker.com/_/mysql/)
- [PHP-FPM](https://hub.docker.com/r/nanoninja/php-fpm/)
- [Composer](https://hub.docker.com/_/composer/)
- [PHPMyAdmin](https://hub.docker.com/r/phpmyadmin/phpmyadmin/)
- [MailHog](https://hub.docker.com/r/mailhog/mailhog/)
- [Generate Certificate](https://hub.docker.com/r/jacoelho/generate-certificate/)

You should be careful when installing third party web servers such as MySQL or Nginx.

This project use the following ports :

| Server     | Port |
| ---------- | ---- |
| Nginx      | 8000 |
| Nginx SSL  | 3000 |
| PHPMyAdmin | 8080 |
| MySQL      | 8089 |
| MailHog    | 8011 |

---

## Clone the project

To install [Git](http://git-scm.com/book/en/v2/Getting-Started-Installing-Git), download it and install following the instructions :

```sh
git clone https://github.com/andersoncorso/docker-cakephp-lemp.git
```

Go to the project directory :

```sh
cd docker-cakephp-lemp
```

### Project tree

```sh
.
├── .env
├── .gitignore
├── .travis.yml
├── docker-compose.yml
├── Makefile
├── README.md
├── data
│   └── db
│       ├── dumps
│       └── mysql
├── etc
│   ├── nginx
│   │   ├── default.conf
│   │   └── default.template.conf
│   ├── php
│   │   ├── Dockerfile
│   │   └── php.ini
│   └── ssl
└── web
    └── public
        └── index.php
```

---

## Configure Nginx With SSL Certificates

You can change the host name by editing the `.env` file.

If you modify the host name, do not forget to add it to the `/etc/hosts` file.

1. Generate SSL certificates

   ```sh
   source .env && sudo docker run --rm -v $(pwd)/etc/ssl:/certificates -e "SERVER=$NGINX_HOST" jacoelho/generate-certificate
   ```

2. Configure Nginx

   Do not modify the `etc/nginx/default.conf` file, it is overwritten by `etc/nginx/default.template.conf`

   Edit nginx file `etc/nginx/default.template.conf` and uncomment the SSL server section :

   ```sh
   # server {
   #     server_name ${NGINX_HOST};
   #
   #     listen 443 ssl;
   #     fastcgi_param HTTPS on;
   #     ...
   # }
   ```

---

## Run the application

PS: If you changed docker-compose.yml container_names, also change the .env file and replace "MYSQL_HOST=dcl_mysql" with container_name

1. Start the application :

   ```sh
   sudo docker-compose up -d
   ```

   **Please wait this might take a several minutes...**

   ```sh
   sudo docker-compose logs -f # Follow log output
   ```

2. Open your favorite browser :

   - [http://localhost:8000](http://localhost:8000/)
   - [https://localhost:3000](https://localhost:3000/) ([HTTPS](#configure-nginx-with-ssl-certificates) not configured by default)
   - [http://localhost:8080](http://localhost:8080/) PHPMyAdmin (username: dev, password: dev)
   - [http://localhost:8011](http://localhost:8011/) MailHog

3. Stop and clear services

   ```sh
   sudo docker-compose down -v
   ```

---

## Use Makefile

When developing, you can use [Makefile](<https://en.wikipedia.org/wiki/Make_(software)>) for doing the following operations :

| Name            | Description                           |
| --------------- | ------------------------------------- |
| clean           | Clean directories for reset           |
| composer-up     | Update PHP dependencies with composer |
| docker-start    | Create and start containers           |
| docker-stop     | Stop and clear all services           |
| gen-certs       | Generate SSL certificates for `nginx` |
| logs            | Follow log output                     |
| mysql-dump      | Create backup of whole database       |
| mysql-restore   | Restore backup from whole database    |
| cakephp-install | Install CakePHP in the public folder  |

### Examples

Start the application :

```sh
sudo make docker-start
```

Show help :

```sh
make help
```

Install CakePHP :

```sh
sudo make clean
make cakephp-install
```

---

## Use Docker commands

### Installing package with composer

```sh
sudo docker run --rm -v $(pwd)/web/public:/app composer require symfony/yaml
```

### Updating PHP dependencies with composer

```sh
sudo docker run --rm -v $(pwd)/web/public:/app composer update
```

### Checking installed PHP extensions

```sh
sudo docker-compose exec php php -m
```

### Handling database

#### MySQL shell access

```sh
sudo docker exec -it mysql bash
```

and

```sh
mysql -u"$MYSQL_ROOT_USER" -p"$MYSQL_ROOT_PASSWORD"
```

#### Backup of database

```sh
mkdir -p data/db/dumps
```

```sh
source .env && sudo docker exec $(sudo docker-compose ps -q mysqldb) mysqldump --all-databases -u"$MYSQL_ROOT_USER" -p"$MYSQL_ROOT_PASSWORD" > "data/db/dumps/db.sql"
```

or

```sh
source .env && sudo docker exec $(sudo docker-compose ps -q mysqldb) mysqldump test -u"$MYSQL_ROOT_USER" -p"$MYSQL_ROOT_PASSWORD" > "data/db/dumps/test.sql"
```

#### Restore Database

```sh
source .env && sudo docker exec -i $(sudo docker-compose ps -q mysqldb) mysql -u"$MYSQL_ROOT_USER" -p"$MYSQL_ROOT_PASSWORD" < "data/db/dumps/db.sql"
```

#### Connecting MySQL from [PDO](http://php.net/manual/en/book.pdo.php)

```php
<?php
    try {
        $dsn = 'mysql:host=mysql;dbname=test;charset=utf8;port=3306';
        $pdo = new PDO($dsn, 'dev', 'dev');
    } catch (PDOException $e) {
        echo $e->getMessage();
    }
?>
```

---
