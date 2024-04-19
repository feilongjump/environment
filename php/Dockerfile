FROM php:5.6-fpm

# 构建环境时 - 环境变量
ARG TZ

# 更改系统源
RUN sed -i -e 's/deb.debian.org/archive.debian.org/g' \
           -e 's|security.debian.org|archive.debian.org/|g' \
           -e '/stretch-updates/d' /etc/apt/sources.list

RUN apt-get update -y \
  && apt-get install -y apt-utils libmcrypt-dev libfreetype6-dev libjpeg-dev libpng-dev libwebp-dev libxpm-dev libzip-dev unzip \
  # 安装 composer
  && curl -sLS https://getcomposer.org/installer | php -- --install-dir=/usr/bin/ --filename=composer \
  # Redis
  && curl -L -o /tmp/redis.tar.gz https://github.com/phpredis/phpredis/archive/4.3.0.tar.gz \
  && tar xfz /tmp/redis.tar.gz \
  && rm -r /tmp/redis.tar.gz \
  && docker-php-source extract \
  && mv phpredis-4.3.0 /usr/src/php/ext/redis \
  # php 扩展
  && docker-php-ext-configure gd --with-webp-dir --with-jpeg-dir --with-xpm-dir --with-freetype-dir \
  && docker-php-ext-install bcmath gd mcrypt pdo_mysql redis zip \
  # 时区
  && ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone \
  # 清除包管理器缓存，以及安装后的非必要文件
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/*

COPY php.ini /usr/local/etc/php/conf.d/99-customize.ini

WORKDIR /var/www

EXPOSE 9000