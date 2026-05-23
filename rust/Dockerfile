FROM rust:1.85-alpine AS builder

ARG APP_UID=1000

WORKDIR /app

RUN apk add --no-cache musl-dev

COPY Cargo.toml Cargo.lock* ./

RUN mkdir src \
    && echo "fn main() {}" > src/main.rs \
    && cargo build --release \
    && rm -rf src target/release/deps target/release/build target/release/.fingerprint

COPY . .

ARG APP_BIN=

RUN cargo build --release \
    && if [ -n "${APP_BIN}" ]; then \
         cp "target/release/${APP_BIN}" /app/bin; \
       else \
         cp "target/release/$(ls target/release/ | grep -v '\.' | head -n 1)" /app/bin; \
       fi

FROM alpine:3.21 AS runtime

ARG APP_PORT=8000
ARG APP_UID=1000
ARG APP_USER=app

RUN apk add --no-cache ca-certificates libgcc \
    && addgroup -g ${APP_UID} -S ${APP_USER} \
    && adduser -S ${APP_USER} -u ${APP_UID} -G ${APP_USER}

WORKDIR /app

COPY --from=builder --chown=${APP_USER}:${APP_USER} /app/bin /app/bin

USER ${APP_USER}

ENV APP_PORT=${APP_PORT}

EXPOSE ${APP_PORT}

CMD ["/app/bin"]
