FROM php:${PHP_VERSION:+${PHP_VERSION}-}fpm

# Update packages and install necessary packages and PHP extensions
RUN apt-get update && \
    apt-get upgrade -y && \
    apt-get install -y \
        curl \
        libpng-dev \
        libjpeg-dev \
        libfreetype6-dev \
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