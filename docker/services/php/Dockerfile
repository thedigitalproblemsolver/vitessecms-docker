#https://hub.docker.com/_/php

FROM php:7.3-apache

ENV APACHE_DOCUMENT_ROOT /var/www/public_html

RUN sed -ri -e 's!/var/www/html!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/sites-available/*.conf
RUN sed -ri -e 's!/var/www/!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/apache2.conf /etc/apache2/conf-available/*.conf
RUN ln /etc/apache2/mods-available/expires.load /etc/apache2/mods-enabled/expires.load
RUN ln /etc/apache2/mods-available/headers.load /etc/apache2/mods-enabled/headers.load
RUN ln /etc/apache2/mods-available/rewrite.load /etc/apache2/mods-enabled/rewrite.load

RUN mv "$PHP_INI_DIR/php.ini-production" "$PHP_INI_DIR/php.ini"

RUN apt-get update && apt-get install -y \
    cron \
    anacron \
    git \
    libpcre3-dev \
    libmcrypt-dev \
    libxml2-dev \
    zlib1g-dev \
    libssh2-1 \
    libssh2-1-dev \
    libpng-dev \
    libzip-dev \
    --no-install-recommends \
    && docker-php-ext-install -j$(nproc) pdo_mysql mbstring soap zip gd intl \
    && cp /usr/local/bin/php /usr/bin/

RUN echo "zend_extension=/usr/local/lib/php/extensions/no-debug-non-zts-20180731/xdebug.so" > /usr/local/lib/php/php.ini

# Install imagick
RUN apt-get update && apt-get install -y libmagickwand-dev --no-install-recommends && rm -rf /var/lib/apt/lists/*
RUN printf "\n" | pecl install imagick
RUN docker-php-ext-enable imagick

# Install ssh2 php extension
RUN cd /tmp \
    && git clone https://git.php.net/repository/pecl/networking/ssh2.git \
    && cd /tmp/ssh2/ \
    && .travis/build.sh \
    && docker-php-ext-enable ssh2

# Install PECL extensions
RUN pecl install xdebug && pecl install mcrypt-1.0.2 && pecl install psr && pecl install mongodb
RUN docker-php-ext-enable xdebug mcrypt psr mongodb intl
RUN echo "extension=psr.so" > /usr/local/lib/php/php.ini
RUN echo "extension=mongodb.so" > /usr/local/lib/php/php.ini

RUN cd /opt/; git clone --branch v3.4.5 --depth 1 https://github.com/phalcon/cphalcon.git; cd cphalcon/build;  ./install; echo "extension=phalcon.so" > /usr/local/etc/php/conf.d/phalcon.ini

RUN docker-php-ext-enable phalcon

#COPY www.conf /usr/local/etc/php-fpm.d/www.conf
