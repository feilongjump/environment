ARG PHP_VERSION
FROM php:fpm

ARG TZ=UTC
ENV TZ ${TZ}

ARG PHP_COMPOSER_URL
ENV PHP_COMPOSER_URL ${PHP_COMPOSER_URL}

ARG PHP_COMPOSER_REPO
ENV PHP_COMPOSER_REPO ${PHP_COMPOSER_REPO}

RUN apt-get update -y \
  && apt-get install -y apt-utils libfreetype6-dev libjpeg-dev libpng-dev libwebp-dev libxpm-dev libzip-dev unzip \
  # composer
  && php -r "copy('$PHP_COMPOSER_URL', 'composer-setup.php');" \
  && php composer-setup.php --filename=composer --install-dir=/usr/local/bin/ \
  && php -r "unlink('composer-setup.php');" \
  && composer config -g repo.packagist composer $PHP_COMPOSER_REPO \
  # phpunit
  && curl -L -o /usr/local/bin/phpunit https://phar.phpunit.de/phpunit-9.phar \
  && chmod +x /usr/local/bin/phpunit \
  # php 扩展
  && pecl channel-update pecl.php.net \
  && pecl install -o -f mongodb redis \
  && rm -rf /tmp/pear \
  && docker-php-ext-enable mongodb redis \
  && docker-php-ext-configure gd --with-webp --with-jpeg --with-xpm --with-freetype \
  && docker-php-ext-install bcmath gd mysqli pcntl pdo_mysql zip \
  # 时区
  && ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

WORKDIR /var/www

EXPOSE 9000
