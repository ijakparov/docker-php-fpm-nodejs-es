FROM php:7.2-fpm-alpine

RUN apk add --no-cache --virtual .build-deps \
        $PHPIZE_DEPS \
        curl-dev \
        imagemagick-dev \
        libtool \
        libpng-dev \
        libxml2-dev \
        postgresql-dev \
        sqlite-dev \
    && apk add --no-cache \
        curl \
        git \
        openssh \
        imagemagick \
        mysql-client \
        postgresql-libs \
        libintl \
        icu \
        icu-dev \
    && pecl install imagick \
    && docker-php-ext-enable imagick \
    && docker-php-ext-install \
        gd \
        curl \
        iconv \
        mbstring \
        pdo \
        pdo_mysql \
        pdo_pgsql \
        pdo_sqlite \
        pcntl \
        tokenizer \
        xml \
        zip \
        intl \
    && apk del -f .build-deps

RUN docker-php-ext-install mysqli && docker-php-ext-enable mysqli

RUN docker-php-ext-install opcache

ENV PHP_OPCACHE_VALIDATE_TIMESTAMPS="0"
ADD opcache.ini "$PHP_INI_DIR/conf.d/opcache.ini"

COPY php.ini /usr/local/etc/php/php.ini

# Composer
COPY --from=composer:latest /usr/bin/composer /usr/local/bin/composer

# fix php 7 Notice: iconv(): Wrong charset, conversion from `UTF-8' to `UTF-8//TRANSLIT//IGNORE' is not allowed
RUN apk add --no-cache --repository http://dl-cdn.alpinelinux.org/alpine/edge/community/ --allow-untrusted gnu-libiconv
ENV LD_PRELOAD /usr/lib/preloadable_libiconv.so php

# install xdebug
RUN apk add --no-cache $PHPIZE_DEPS \
    && pecl install xdebug \
    && docker-php-ext-enable xdebug
COPY xdebug.ini $PHP_INI_DIR/conf.d/docker-php-ext-xdebug.ini
RUN echo "zend_extension=$(find /usr/local/lib/php/extensions/ -name xdebug.so)" >> $PHP_INI_DIR/conf.d/docker-php-ext-xdebug.ini



# add credentials on build
ARG SSH_PRIVATE_KEY
RUN mkdir /root/.ssh/
RUN echo "${SSH_PRIVATE_KEY}" > /root/.ssh/id_rsa
RUN chmod 600 /root/.ssh/id_rsa

# make sure your domain is accepted
RUN touch /root/.ssh/known_hosts
RUN ssh-keyscan bitbucket.org >> /root/.ssh/known_hosts

#RUN chown -R www-data:www-data /var/www/

RUN echo http://dl-2.alpinelinux.org/alpine/edge/community/ >> /etc/apk/repositories
RUN apk --no-cache add shadow && usermod -u 1000 www-data

WORKDIR /var/www/html


# ADD . /var/www/html
# RUN chown -R www-data:www-data /var/www/html
