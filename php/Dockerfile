FROM composer:2.4.4 as composer

FROM golang:1.18 as scc
WORKDIR /scc
COPY ./scc/ ./
RUN go mod download
RUN go build

FROM php:8.1-fpm-buster as main
VOLUME /var/www/project/
WORKDIR /var/www/project/
RUN apt-get update
RUN apt-get install -y curl
RUN curl -sSLf \
        -o /usr/local/bin/install-php-extensions \
        https://github.com/mlocati/docker-php-extension-installer/releases/latest/download/install-php-extensions && \
    chmod +x /usr/local/bin/install-php-extensions && \
    IPE_LZF_BETTERCOMPRESSION=1 install-php-extensions \
        pdo_mysql \
        amqp \
        pcntl \
        opcache \
        apcu \
        sodium \
        xsl \
        zip \
        intl

COPY ./app/ /var/www/project/

COPY --from=composer /usr/bin/composer /usr/bin/composer
## get node and npm
ENV NODE_VERSION=16.13.0

RUN curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
ENV NVM_DIR=/root/.nvm
RUN . "$NVM_DIR/nvm.sh" && nvm install ${NODE_VERSION}
RUN . "$NVM_DIR/nvm.sh" && nvm use v${NODE_VERSION}
RUN . "$NVM_DIR/nvm.sh" && nvm alias default v${NODE_VERSION}
ENV PATH="/root/.nvm/versions/node/v${NODE_VERSION}/bin/:${PATH}"
## get dependency files
## run dependency managers
RUN composer install --no-interaction --no-scripts
RUN npm install

RUN npm run build
###### PROD SECTION
RUN apt-get update
RUN apt-get install -y supervisor git
RUN printf '#!/bin/sh\nexit 0' > /usr/sbin/policy-rc.d
RUN mkdir -p /var/log/supervisor
COPY messenger-worker.conf /etc/supervisor/conf.d/supervisord.conf
COPY --from=scc ./scc/scc /usr/local/bin/scc
COPY ./app/.env /var/www/project/.env.local
RUN bin/console assets:install
RUN bin/console cache:clear -e prod
ENTRYPOINT ["bin/console", "do:mi:mi", "--no-interaction"]
CMD ["/usr/bin/supervisord"]
