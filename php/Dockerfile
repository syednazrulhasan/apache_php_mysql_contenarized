FROM php:${PHP_VERSION:+${PHP_VERSION}-}fpm-debian

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
