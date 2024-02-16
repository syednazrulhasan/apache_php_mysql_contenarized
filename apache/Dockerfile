ARG APACHE_VERSION=""
FROM httpd:${APACHE_VERSION:+${APACHE_VERSION}-}alpine

# Update and install necessary packages
RUN apk update && \
    apk upgrade && \
    apk add --no-cache \
        apache2-ssl \
        apache2-utils \
        openssl \
        libpng-dev \
        libjpeg-turbo-dev \
        libfreetype6-dev \
        libzip-dev \
        zip \
        unzip \
        mariadb-dev \


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