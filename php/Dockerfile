FROM composer:2 AS composer

WORKDIR /var/www/html

COPY . .

RUN if [ -f composer.json ]; then \
      composer install --no-dev --prefer-dist --no-interaction --no-scripts --no-autoloader; \
    fi

FROM php:8.4-fpm-alpine AS builder

RUN apk add --no-cache \
      libzip-dev \
      icu-dev \
      oniguruma-dev \
      freetype-dev \
      libjpeg-turbo-dev \
      libpng-dev \
    && docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install -j$(nproc) \
      pdo_mysql \
      zip \
      intl \
      opcache \
      bcmath \
      pcntl \
      gd

WORKDIR /var/www/html

COPY --from=composer /usr/bin/composer /usr/bin/composer

COPY . .

RUN rm -rf nginx php-fpm Dockerfile docker-compose.yml .dockerignore .env.example

COPY --from=composer /var/www/html /tmp/composer-html

RUN if [ -d /tmp/composer-html/vendor ]; then \
      cp -r /tmp/composer-html/vendor ./vendor; \
    fi \
    && rm -rf /tmp/composer-html \
    && if [ -f composer.json ]; then \
      composer dump-autoload --optimize --no-dev --no-interaction; \
    fi

FROM php:8.4-fpm-alpine AS runtime

ARG APP_PORT=8000
ARG APP_UID=1000
ARG APP_USER=www-data
ARG WEB_ROOT=public

RUN apk add --no-cache \
      nginx \
      libzip-dev \
      icu-dev \
      oniguruma-dev \
      freetype-dev \
      libjpeg-turbo-dev \
      libpng-dev \
    && docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install -j$(nproc) \
      pdo_mysql \
      zip \
      intl \
      opcache \
      bcmath \
      pcntl \
      gd \
    && deluser www-data 2>/dev/null || true \
    && delgroup www-data 2>/dev/null || true \
    && addgroup -g ${APP_UID} -S ${APP_USER} \
    && adduser -S ${APP_USER} -u ${APP_UID} -G ${APP_USER} \
    && mkdir -p /var/run/php /var/www/html \
    && chown -R ${APP_USER}:${APP_USER} /var/run/php /var/www/html

WORKDIR /var/www/html

COPY --from=builder --chown=${APP_USER}:${APP_USER} /var/www/html /var/www/html

COPY php-fpm/zz-docker.conf /tmp/zz-docker.conf

COPY nginx/default.conf /tmp/default.conf

RUN if [ "${WEB_ROOT}" = "." ] || [ -z "${WEB_ROOT}" ]; then \
      DOCUMENT_ROOT="/var/www/html"; \
    else \
      DOCUMENT_ROOT="/var/www/html/${WEB_ROOT}"; \
    fi \
    && sed "s/__APP_USER__/${APP_USER}/g" /tmp/zz-docker.conf > /usr/local/etc/php-fpm.d/zz-docker.conf \
    && rm /tmp/zz-docker.conf \
    && chown ${APP_USER}:${APP_USER} /usr/local/etc/php-fpm.d/zz-docker.conf \
    && sed "s/__APP_PORT__/${APP_PORT}/g; s|__DOCUMENT_ROOT__|${DOCUMENT_ROOT}|g" /tmp/default.conf > /etc/nginx/http.d/default.conf \
    && rm /tmp/default.conf \
    && mkdir -p /etc/nginx/conf.d /tmp/nginx /var/lib/nginx/tmp \
    && echo 'pid /tmp/nginx/nginx.pid;' > /etc/nginx/conf.d/00-pid.conf \
    && chown -R ${APP_USER}:${APP_USER} /var/lib/nginx /var/log/nginx /etc/nginx /tmp

USER ${APP_USER}

ENV APP_PORT=${APP_PORT}

EXPOSE ${APP_PORT}

ENTRYPOINT []

CMD ["sh", "-c", "php-fpm -D && exec nginx -g 'daemon off;'"]
