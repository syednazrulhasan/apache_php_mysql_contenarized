ARG APACHE_VERSION=""
FROM httpd:${APACHE_VERSION:+${APACHE_VERSION}-}2.4

# Install necessary packages and PHP extensions
RUN apt-get update && \
    apt-get install -y \
        curl \
        libapache2-mod-php7.4 \
        libpng-dev \
        libjpeg-dev \
        libfreetype6-dev \
        libzip-dev \
        zip \
        unzip \
        openssl && \
    docker-php-ext-install -j$(nproc) mysqli gd pdo_mysql zip && \
    docker-php-ext-configure zip --with-libzip && \
    docker-php-ext-install -j$(nproc) zip && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Enable necessary Apache modules
RUN a2enmod rewrite && \
    a2enmod ssl

# Allow .htaccess overrides
RUN sed -i '/<Directory "\/var\/www\/html">/,/<\/Directory>/ s/AllowOverride None/AllowOverride All/' /etc/apache2/apache2.conf

# Copy apache vhost file to proxy php requests to php-fpm container
COPY demo.apache.conf /etc/apache2/sites-available/demo.apache.conf
RUN a2ensite demo.apache.conf

# Set the working directory to the root of the web application
WORKDIR /var/www/html

# Expose port 80
EXPOSE 80

# Start Apache service
CMD ["httpd-foreground"]