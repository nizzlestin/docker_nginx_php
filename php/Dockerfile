## DOCS
## https://hub.docker.com/_/php
FROM php:8.1-fpm-alpine

VOLUME /var/www/project/
WORKDIR /var/www/project/

### --- Install: PHP-Extensions --- ###
## docs: https://github.com/mlocati/docker-php-extension-installer
RUN curl -sSLf \
        -o /usr/local/bin/install-php-extensions \
        https://github.com/mlocati/docker-php-extension-installer/releases/latest/download/install-php-extensions && \
    chmod +x /usr/local/bin/install-php-extensions && \
    IPE_LZF_BETTERCOMPRESSION=1 install-php-extensions \
        pdo_mysql \
        amqp \
        opcache \
        apcu \
        zip \
        xsl \
        intl

## For furhter configuration see example here: 
##   https://github.com/ventureleap/fckcrn-cockpit/blob/main/docker/Dockerfile.php



## Last step to copy the source code into Docker image

COPY ./app/ /var/www/project
