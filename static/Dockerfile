FROM alpine:3.21 AS builder

WORKDIR /site

COPY . /site

RUN rm -rf /site/nginx /site/Dockerfile /site/docker-compose.yml /site/.dockerignore /site/.env.example

FROM nginx:1.27-alpine AS runtime

ARG APP_PORT=8000
ARG APP_UID=1000
ARG APP_USER=nginx

RUN apk add --no-cache shadow \
    && deluser nginx 2>/dev/null || true \
    && delgroup nginx 2>/dev/null || true \
    && addgroup -g ${APP_UID} -S ${APP_USER} \
    && adduser -S ${APP_USER} -u ${APP_UID} -G ${APP_USER}

COPY --from=builder --chown=${APP_USER}:${APP_USER} /site /usr/share/nginx/html

COPY nginx/default.conf /tmp/default.conf

RUN sed "s/__APP_PORT__/${APP_PORT}/g" /tmp/default.conf > /etc/nginx/conf.d/default.conf \
    && rm /tmp/default.conf \
    && sed -i 's|pid /run/nginx.pid;|pid /tmp/nginx.pid;|' /etc/nginx/nginx.conf \
    && sed -i "s|user  nginx;|user ${APP_USER};|" /etc/nginx/nginx.conf \
    && chown -R ${APP_USER}:${APP_USER} /var/cache/nginx /var/log/nginx /etc/nginx/conf.d /tmp

USER ${APP_USER}

ENV APP_PORT=${APP_PORT}

EXPOSE ${APP_PORT}

CMD ["nginx", "-g", "daemon off;"]
