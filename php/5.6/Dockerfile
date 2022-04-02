ARG PHP_VERSION
FROM php:5.6-fpm

ARG TZ=UTC
ENV TZ ${TZ}

ARG PHP_COMPOSER_URL
ENV PHP_COMPOSER_URL ${PHP_COMPOSER_URL}

ARG PHP_COMPOSER_REPO
ENV PHP_COMPOSER_REPO ${PHP_COMPOSER_REPO}

ARG PHP56_REDIS_VERSION
ENV PHP56_REDIS_VERSION ${PHP56_REDIS_VERSION}

# 更改为阿里源
RUN sed -i 's/deb.debian.org/mirrors.tuna.tsinghua.edu.cn/' /etc/apt/sources.list \
  && sed -i 's/security.debian.org/mirrors.tuna.tsinghua.edu.cn/' /etc/apt/sources.list \
  && sed -i 's/security-cdn.debian.org/mirrors.tuna.tsinghua.edu.cn/' /etc/apt/sources.list

RUN apt-get update -y \
  && apt-get install -y apt-utils libmcrypt-dev libfreetype6-dev libjpeg-dev libpng-dev libwebp-dev libxpm-dev libzip-dev unzip \
  # composer
  && curl $PHP_COMPOSER_URL -o /usr/local/bin/composer \
  && chmod a+x /usr/local/bin/composer \
  && composer config -g repo.packagist composer $PHP_COMPOSER_REPO \
  # Redis
  && curl -L -o /tmp/redis.tar.gz https://github.com/phpredis/phpredis/archive/$PHP56_REDIS_VERSION.tar.gz \
  && tar xfz /tmp/redis.tar.gz \
  && rm -r /tmp/redis.tar.gz \
  && docker-php-source extract \
  && mv phpredis-$PHP56_REDIS_VERSION /usr/src/php/ext/redis \
  # php 扩展
  && docker-php-ext-configure gd --with-webp-dir --with-jpeg-dir --with-xpm-dir --with-freetype-dir \
  && docker-php-ext-install bcmath gd mcrypt mysqli pdo_mysql redis zip \
  # 时区
  && ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

WORKDIR /var/www

EXPOSE 9000
