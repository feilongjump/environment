FROM php:8.0-fpm

# 构建环境时 - 环境变量
ARG TZ

RUN sed -i "s/deb.debian.org/mirrors.aliyun.com/g" /etc/apt/sources.list 

RUN apt-get update -y \
  # 安装 composer
  && curl -sLS https://getcomposer.org/installer | php -- --install-dir=/usr/bin/ --filename=composer \
  # 未使用阿里云镜像源，有些包更新并不及时
  # && composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/ \
  # 这些软件的安装，是为了在 PHP 环境中支持各种图像处理和压缩功能
  && apt-get install -y libfreetype6-dev libjpeg-dev libmcrypt-dev libpng-dev libwebp-dev libxpm-dev libzip-dev unzip \
  # php 扩展
  # PHP8必须装1.0.4  参考 https://bugs.php.net/bug.php?id=80459
  && pecl install mcrypt-1.0.4 \
  && docker-php-ext-enable mcrypt \
  && pecl channel-update pecl.php.net \
  && pecl install -o -f redis \
  && docker-php-ext-enable redis \
  && docker-php-ext-configure gd --with-webp --with-jpeg --with-xpm --with-freetype \
  && docker-php-ext-install bcmath gd pdo_mysql zip \
  # 时区
  && ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone \
  # 清除包管理器缓存，以及安装后的非必要文件
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/*

COPY php.ini /usr/local/etc/php/conf.d/99-customize.ini

WORKDIR /var/www

EXPOSE 9000
