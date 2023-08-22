FROM php:8.1.0RC2-apache
# update base container ubuntu OS
RUN apt-get update -y
# change apache root
ENV APACHE_DOCUMENT_ROOT=/var/www/html/public
# update apache default conf
RUN sed -ri -e 's!/var/www/html!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/sites-available/*.conf
RUN sed -ri -e 's!/var/www/!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/apache2.conf /etc/apache2/conf-available/*.conf

RUN apt-get update && apt-get install -y libmcrypt-dev \
    libmagickwand-dev libzip-dev zip libfreetype6-dev \
    libjpeg62-turbo-dev libpng-dev --no-install-recommends \
    && docker-php-ext-install zip \
    && pecl install imagick \
    && docker-php-ext-enable imagick \
    && docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install -j$(nproc) gd \ 
    && docker-php-ext-install pdo_mysql

# install composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
# update composer
RUN composer self-update
# install git
RUN apt install git -y
# install nodejs
RUN curl -sL https://deb.nodesource.com/setup_16.x | bash
RUN apt install nodejs -y

EXPOSE 80