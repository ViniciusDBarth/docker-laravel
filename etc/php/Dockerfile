FROM php:7.4-fpm

LABEL maintainer="Vinicius DallOglio Barth <viniciusdbarth@gmail.com>"

# Composer
RUN curl -sS https://getcomposer.org/installer | php && mv composer.phar /usr/local/bin/composer

### EXEMPLOS DE CMDS:
# Install selected extensions and other stuff
RUN apt-get update && apt-get install -y zlib1g-dev libicu-dev g++
RUN apt-get update && \
    apt-get install -y software-properties-common && \
    rm -rf /var/lib/apt/lists/*
RUN apt-get install openssl
#RUN add-apt-repository ppa:ondrej/php
#RUN apt-get update && apt-get install php7.4-curl
RUN docker-php-ext-configure intl
RUN docker-php-ext-install intl
RUN docker-php-ext-configure pdo_mysql
RUN docker-php-ext-install pdo_mysql
RUN docker-php-ext-configure json
RUN docker-php-ext-install json

#     && apt-get clean; rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* /usr/share/doc/*

# # Install git
RUN apt-get update \
    && apt-get -y install git \
    && apt-get clean; rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* /usr/share/doc/*

# comentario que so eu quero ver no meu codigo