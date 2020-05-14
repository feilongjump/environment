FROM php:7.4-fpm

ARG TZ=UTC
ENV TZ ${TZ}

RUN docker-php-ext-install mysqli pdo pdo_mysql \
  && ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone


EXPOSE 9000
