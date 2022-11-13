FROM composer:2.4.4 as composer

FROM golang:1.18 as scc
WORKDIR /scc
COPY ./scc/ ./
RUN go mod download
RUN go build

FROM nizzlestin/thesis-fpm:1.0 as main


COPY ./app/ /var/www/project/
COPY --from=composer /usr/bin/composer /usr/bin/composer

RUN curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
###### PROD SECTION
RUN apt-get update
RUN apt-get install -y supervisor git
RUN printf '#!/bin/sh\nexit 0' > /usr/sbin/policy-rc.d
RUN mkdir -p /var/log/supervisor
COPY messenger-worker.conf /etc/supervisor/conf.d/supervisord.conf
COPY --from=scc ./scc/scc /usr/local/bin/scc
ENTRYPOINT ["/usr/bin/supervisord"]