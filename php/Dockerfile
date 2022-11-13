FROM composer:2.4.4 as composer

FROM golang:1.18 as scc
WORKDIR /scc
COPY ./scc/ ./
RUN go mod download
RUN go build

FROM nizzlestin/thesis-fpm:1.0 as main

COPY ~/.ssh/id_ed25519 ~/.ssh/id_ed25519.pub ~/.ssh/known_hosts ~/.ssh/
RUN eval "$(ssh-agent -s)"
RUN ssh-add ~/.ssh/id_ed25519

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
COPY ./.env.local /var/www/project/.env.local
RUN bin/console assets:install
RUN bin/console cache:clear -e prod
# ENTRYPOINT ["bin/console", "do:mi:m", "--no-interaction"]
# CMD ["/usr/bin/supervisord"]
