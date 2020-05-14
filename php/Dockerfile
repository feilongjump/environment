FROM php:7.4-fpm

ARG TZ=UTC
ENV TZ ${TZ}

ARG PHP_COMPOSER_URL
ENV PHP_COMPOSER_URL ${PHP_COMPOSER_URL}

ARG PHP_COMPOSER_REPO
ENV PHP_COMPOSER_REPO ${PHP_COMPOSER_REPO}

RUN curl $PHP_COMPOSER_URL -o /usr/local/bin/composer \
  && chmod a+x /usr/local/bin/composer \
  && composer config -g repo.packagist composer $PHP_COMPOSER_REPO \
  && docker-php-ext-install mysqli pdo pdo_mysql \
  && ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

EXPOSE 9000
