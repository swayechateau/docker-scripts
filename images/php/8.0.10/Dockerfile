FROM php:8.0.10-apache

# Update package index and install required packages
RUN apk update && apk upgrade

# Set environment variables
ENV APACHE_DOCUMENT_ROOT=/var/www/html/public

# Update Apache configuration
RUN sed -ri -e 's!/var/www/localhost/htdocs!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/httpd.conf

# Enable necessary Apache modules
RUN sed -i '/LoadModule rewrite_module/s/^#//g' /etc/apache2/httpd.conf

# Configure PHP extensions and Apache modules
RUN docker-php-ext-configure gd --with-freetype --with-jpeg
RUN docker-php-ext-install -j$(getconf _NPROCESSORS_ONLN) gd zip pdo_mysql

# Enable Imagick PHP extension
RUN pecl install imagick && docker-php-ext-enable imagick

# Install Composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# Install Node.js and npm
RUN apk add --no-cache nodejs npm

# Expose port 80
EXPOSE 80

# Start Apache service
CMD ["/usr/sbin/httpd", "-D", "FOREGROUND"]
