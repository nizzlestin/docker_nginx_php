FROM php:8.1-fpm-alpine as main
VOLUME /var/www/project/
WORKDIR /var/www/project/
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

FROM composer:2.4.1 as composer

FROM golang:1.18 as scc
WORKDIR /scc
RUN apt-get update && apt-get install -y golang
COPY ./scc/ ./
RUN go mod download
RUN go build

###### BUILDING SECTION ######
FROM main as build-stage
COPY --from=composer /usr/bin/composer /usr/bin/composer
## get node and npm
ENV NODE_VERSION=16.13.0
RUN apt install -y curl
RUN curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
ENV NVM_DIR=/root/.nvm
RUN . "$NVM_DIR/nvm.sh" && nvm install ${NODE_VERSION}
RUN . "$NVM_DIR/nvm.sh" && nvm use v${NODE_VERSION}
RUN . "$NVM_DIR/nvm.sh" && nvm alias default v${NODE_VERSION}
ENV PATH="/root/.nvm/versions/node/v${NODE_VERSION}/bin/:${PATH}"
## get dependency files
COPY composer.json composer.lock ./
COPY package.json package.lock ./
## run dependency managers
RUN composer install --no-interaction --no-scripts
RUN npm install
COPY webpack.config.js ./
COPY assets/ assets/
COPY public/ public/
RUN encore

###### PROD SECTION
FROM main as prod
RUN apt-get update && apt-get install -y supervisor git
RUN mkdir -p /var/log/supervisor
COPY messenger-worker.conf /etc/supervisor/conf.d/messenger-worker.conf
COPY --from=scc ./scc /usr/local/bin/scc
COPY --from=build-stage /var/www/project/ /var/www/project/
COPY ./app/ /var/www/project/
COPY .env .env.local
RUN bin/console assets:install
RUN bin/console cache:clear -e prod
CMD ["/usr/bin/supervisord"]
