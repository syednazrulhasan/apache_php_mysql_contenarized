FROM httpd:${APACHE_VERSION}-alpine

# Install necessary packages and PHP extensions
RUN apk update && \
    apk add --no-cache \
        apache2-proxy \
        apache2-utils \
        libpng-dev \
        libjpeg-turbo-dev \
        freetype-dev \
        libzip-dev \
        zip \
        unzip \
        mariadb-connector-c \
        openssl && \
    docker-php-ext-install -j$(nproc) mysqli gd pdo_mysql zip && \
    docker-php-ext-configure zip --with-libzip && \
    docker-php-ext-install -j$(nproc) zip && \
    apk del libpng-dev libjpeg-turbo-dev freetype-dev libzip-dev && \
    apk cache clean && \
    rm -rf /var/cache/apk/*

# Enable necessary Apache modules
RUN sed -i '/<IfModule mime_module>/,/<\/IfModule>/ s/^\(\s*LoadModule mime_magic_module\)\(\s*options\).*$/\1\n\2 -DSENDFILE=0/' /etc/apache2/httpd.conf && \
    sed -i '/<IfModule proxy_module>/,/<\/IfModule>/ s/^\(\s*LoadModule proxy_module\)\(\s*options\).*$/\1\n\2 -DSENDFILE=0/' /etc/apache2/httpd.conf && \
    sed -i '/<IfModule proxy_fcgi_module>/,/<\/IfModule>/ s/^\(\s*LoadModule proxy_fcgi_module\)\(\s*options\).*$/\1\n\2 -DSENDFILE=0/' /etc/apache2/httpd.conf && \
    sed -i 's/#LoadModule rewrite_module/LoadModule rewrite_module/' /etc/apache2/httpd.conf && \
    sed -i 's/#LoadModule proxy_module/LoadModule proxy_module/' /etc/apache2/httpd.conf && \
    sed -i 's/#LoadModule proxy_fcgi_module/LoadModule proxy_fcgi_module/' /etc/apache2/httpd.conf && \
    a2enmod proxy proxy_fcgi && \
    apachectl configtest && \
    apachectl graceful

# Allow .htaccess overrides
RUN sed -i '/<Directory "\/var\/www\/html">/,/<\/Directory>/ s/AllowOverride None/AllowOverride All/' /etc/apache2/httpd.conf

# Copy apache vhost file to proxy php requests to php-fpm container
COPY demo.apache.conf /etc/apache2/sites-available/demo.apache.conf
RUN echo "Include /etc/apache2/sites-available/demo.apache.conf" >> /etc/apache2/httpd.conf

# Set the working directory to the root of the web application
WORKDIR /var/www/html

# Expose port 80
EXPOSE 80

# Start Apache service
CMD ["httpd-foreground"]