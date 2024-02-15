# Containerize Apache/PHP/MySQL
This repo contains code that helps to contenarise Apache/PHP/MySQL and can be spun up with *docker-compose up -d*
===================================

### Intro
Learn how to containerize Apache,PHP and MySQL quickly for testing or demo purposes using Docker Compose and Dockerfile. Deploy your application environment in moments with flexibility of PHP.ini and Apache.conf files, mirroring your development computer's structure with just one command ```docker-compose up -d```

There are 6 simple files that you can clone from this repo's main branch

```
/apache_php_mysql_contenarized/
├── apache
│├── demo.apache.conf
│└── Dockerfile
├── docker-compose.yml
├── php
│├── Dockerfile
│└── user.ini
├── public_html
 └── index.php
```

Once this structure is replicated next step is to install docker on server OS followed by docker-compose followed by running command ```docker-compose up -d``` to spin up the Apache PHP MySQL servers as docker containers on the host machine.


The following code attempts to connect to a MySQL database using the mysqli interface from PHP. If successful, it prints a success. If not, it prints a failed message.

#### index.php
```
<h1>Hello Cloudreach!</h1>
<h4>Attempting MySQL connection from php...</h4>
<?php
$host = 'mysql';
$user = 'root';
$pass = 'Y1ZzhkWkD7U';
$conn = new mysqli($host, $user, $pass);

if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
} else {
  echo "Connected to MySQL successfully!";
}
?>
```

The following is a simple docker-compose.yml that facilitates the contenarization of Apache/PHP/MySQL
#### docker-compose.yml
```
version: "3.9"
services:
  php:
    build: 
      context: './php/'
      args:
       PHP_VERSION: ${PHP_VERSION}
    networks:
      - backend
    volumes:
      - ${PROJECT_ROOT}/:/var/www/html/
    container_name: php
  apache:
    build:
      context: './apache/'
      args:
       APACHE_VERSION: ${APACHE_VERSION}
    depends_on:
      - php
      - mysql
    networks:
      - frontend
      - backend
    ports:
      - "80:80"
    volumes:
      - ${PROJECT_ROOT}/:/var/www/html/
    container_name: apache
  mysql:
    image: mysql:${MYSQL_VERSION:-latest}
    restart: always
    ports:
      - "3306:3306"
    volumes:
            - data:/var/lib/mysql
    networks:
      - backend
    environment:
      MYSQL_ROOT_PASSWORD: "${DB_ROOT_PASSWORD}"
      MYSQL_DATABASE: "${DB_NAME}"
      MYSQL_USER: "${DB_USERNAME}"
      MYSQL_PASSWORD: "${DB_PASSWORD}"
    container_name: mysql
networks:
  frontend:
  backend:
volumes:
    data:
```

#### apache/Dockerfile enable necessary Apache modules and .htaccess overide
```
ARG APACHE_VERSION=""
FROM httpd:${APACHE_VERSION:+${APACHE_VERSION}-}alpine

# Update and install necessary packages
RUN apk update && \
    apk upgrade && \
    apk add --no-cache \
        apache2-ssl \
        apache2-utils \
        openssl

# Enable necessary Apache modules
RUN sed -i '/LoadModule rewrite_module/s/^#//g' /usr/local/apache2/conf/httpd.conf && \
    sed -i '/LoadModule ssl_module/s/^#//g' /usr/local/apache2/conf/httpd.conf

# Allow .htaccess overrides
RUN sed -i '/<Directory "\/usr\/local\/apache2\/htdocs">/,/<\/Directory>/ s/AllowOverride None/AllowOverride All/' /usr/local/apache2/conf/httpd.conf

# Copy apache vhost file to proxy php requests to php-fpm container
COPY demo.apache.conf /usr/local/apache2/conf/demo.apache.conf
RUN echo "Include /usr/local/apache2/conf/demo.apache.conf" \
    >> /usr/local/apache2/conf/httpd.conf

# Set the working directory to the root of the web application
WORKDIR /var/www/html

# Expose port 80
EXPOSE 80
```

#### php/Dockerfile enable neccesary php extentions and custom user.ini file to overide php defaults
```
FROM php:${PHP_VERSION:+${PHP_VERSION}-}fpm-alpine

# Install necessary packages and PHP extensions
RUN apk update && \
    apk upgrade && \
    apk add --no-cache \
        curl \
        libpng-dev \
        libjpeg-turbo-dev \
        freetype-dev \
        libzip-dev \
        zip \
        unzip

RUN docker-php-ext-install \
    mysqli \
    gd \
    pdo_mysql \
    zip

# Copy custom PHP configuration
COPY user.ini /usr/local/etc/php/conf.d/user.ini
```

#### apache/demo.apache.conf
```
ServerName localhost

LoadModule deflate_module /usr/local/apache2/modules/mod_deflate.so
LoadModule proxy_module /usr/local/apache2/modules/mod_proxy.so
LoadModule proxy_fcgi_module /usr/local/apache2/modules/mod_proxy_fcgi.so
LoadModule rewrite_module /usr/local/apache2/modules/mod_rewrite.so
LoadModule expires_module /usr/local/apache2/modules/mod_expires.so

<VirtualHost *:80>
    # Proxy .php requests to port 9000 of the php-fpm container
    ProxyPassMatch ^/(.*\.php(/.*)?)$ fcgi://php:9000/var/www/html/$1
    DocumentRoot /var/www/html/
    <Directory /var/www/html/>
        DirectoryIndex index.php
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
    
    # Enable mod_rewrite
    RewriteEngine On

    # Example RewriteRule
    # RewriteRule ^/example/(.*)$ /index.php?page=$1 [L,QSA]

    # Enable mod_expires
    ExpiresActive On
    ExpiresDefault "access plus 1 month"

    # Send apache logs to stdout and stderr
    CustomLog /proc/self/fd/1 common
    ErrorLog /proc/self/fd/2
</VirtualHost>
```

### Volumes
We have demonstrated use of persistent volume in case your container crashes your data is safe in persistent volumes but make sure to keep everything backup there is no best alternate ever to it(backups)

### Demonstration of docker-compose up!
