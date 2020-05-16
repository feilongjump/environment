ARG PHP_VERSION
FROM php:${PHP_VERSION}-fpm

ARG TZ=UTC
ENV TZ ${TZ}

ARG PHP_COMPOSER_URL
ENV PHP_COMPOSER_URL ${PHP_COMPOSER_URL}

ARG PHP_COMPOSER_REPO
ENV PHP_COMPOSER_REPO ${PHP_COMPOSER_REPO}

# 更改为阿里源
RUN sed -i 's/deb.debian.org/mirrors.tuna.tsinghua.edu.cn/' /etc/apt/sources.list \
    && sed -i 's/security.debian.org/mirrors.tuna.tsinghua.edu.cn/' /etc/apt/sources.list \
    && sed -i 's/security-cdn.debian.org/mirrors.tuna.tsinghua.edu.cn/' /etc/apt/sources.list

RUN apt-get update -y \
  && apt-get install -y apt-utils libfreetype6-dev libjpeg-dev libpng-dev libwebp-dev libxpm-dev libzip-dev unzip \
  # composer
  && curl $PHP_COMPOSER_URL -o /usr/local/bin/composer \
  && chmod a+x /usr/local/bin/composer \
  && composer config -g repo.packagist composer $PHP_COMPOSER_REPO \
  # php 扩展
  && docker-php-ext-install bcmath mysqli pdo_mysql zip \
  # 时区
  && ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

###########################################################################
# GD 库
###########################################################################
RUN if [ ${PHP_VERSION} = "7.4" ]; then \
      docker-php-ext-configure gd --with-webp --with-jpeg --with-xpm --with-freetype; \
    else \
      docker-php-ext-configure gd --with-webp-dir --with-jpeg-dir --with-xpm-dir --with-freetype-dir; \
    fi \
    && docker-php-ext-install gd;

EXPOSE 9000
