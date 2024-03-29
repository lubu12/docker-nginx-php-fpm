# Dockerfile Best Practice - https://docs.docker.com/develop/develop-images/dockerfile_best-practices/

# Advantage to have nginx and php in different containers
#   https://devops.stackexchange.com/questions/2089/what-are-the-advantages-of-dockerizing-nginx-and-php-in-different-containers

ARG version

FROM php:${version}
MAINTAINER Andrew Yeung <andrew@compandsave.com>

ARG version
ARG html_volume

LABEL app.name="php-fpm"
LABEL app.version="${version}"

RUN apk update --no-cache

# Install dependencies with set -ex; - exits as soon as any line in the bash script fails with print on each command at screen
# --virtual - to build as virutal packages - https://github.com/gliderlabs/docker-alpine/blob/master/docs/usage.md
RUN set -ex; \
  \
  apk add --no-cache --virtual .build-deps \
    bzip2-dev \
    freetype-dev \
    libjpeg-turbo-dev \
    libpng-dev \
    libwebp-dev \
    libxpm-dev \
    libzip-dev \
  ; \
  \
  docker-php-ext-configure gd --with-freetype-dir=/usr --with-jpeg-dir=/usr --with-webp-dir=/usr --with-png-dir=/usr --with-xpm-dir=/usr; \
  docker-php-ext-install bz2 gd mysqli opcache zip; \
  \
  runDeps="$( \
    scanelf --needed --nobanner --format '%n#p' --recursive /usr/local/lib/php/extensions \
      | tr ',' '\n' \
      | sort -u \
      | awk 'system("[ -e /usr/local/lib/" $1 " ]") == 0 { next } { print "so:" $1 }' \
  )"; \
  apk add --virtual .phpmyadmin-phpexts-rundeps $runDeps; \
  apk del .build-deps

# set recommended PHP.ini settings
# see https://secure.php.net/manual/en/opcache.installation.php
RUN { \
        echo 'opcache.memory_consumption=128'; \
        echo 'opcache.interned_strings_buffer=8'; \
        echo 'opcache.max_accelerated_files=4000'; \
        echo 'opcache.revalidate_freq=2'; \
        echo 'opcache.fast_shutdown=1'; \
        echo 'opcache.enable_cli=1'; \
    } > /usr/local/etc/php/conf.d/opcache-recommended.ini; \
    \
    { \
        echo 'session.cookie_httponly = 1'; \
        echo 'session.use_strict_mode = 1'; \
    } > /usr/local/etc/php/conf.d/session-strict.ini; \
    \
    { \
        echo 'allow_url_fopen = Off'; \
        echo 'max_execution_time = 600'; \
        echo 'memory_limit = 512M'; \
    } > /usr/local/etc/php/conf.d/phpmyadmin-misc.ini


COPY www.conf /usr/local/etc/php-fpm.d/
COPY php.ini /usr/local/etc/php/
COPY php-fpm.conf /usr/local/etc/

RUN addgroup -g 3000 app \
 && adduser -D -G app -u 3000 -H app

USER app

VOLUME ${html_volume}
