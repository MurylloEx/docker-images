FROM alpine:3.21 AS builder

WORKDIR /app

RUN apk add --no-cache \
    python3 \
    py3-pip \
    python3-dev \
    gcc \
    musl-dev \
    libffi-dev \
    && ln -sf python3 /usr/bin/python

COPY requirements.txt* ./

RUN python3 -m pip install --no-cache-dir --prefix=/install "uvicorn[standard]" \
    && if [ -f requirements.txt ]; then \
      python3 -m pip install --no-cache-dir --prefix=/install -r requirements.txt; \
    fi

FROM alpine:3.21 AS runtime

ARG APP_PORT=8000
ARG APP_UID=1000
ARG APP_USER=app

RUN apk add --no-cache python3 \
    && ln -sf python3 /usr/bin/python \
    && addgroup -g ${APP_UID} -S ${APP_USER} \
    && adduser -S ${APP_USER} -u ${APP_UID} -G ${APP_USER}

WORKDIR /app

COPY --from=builder /install/bin /usr/local/bin

COPY --from=builder /install/lib/python3.12/site-packages /usr/lib/python3.12/site-packages

COPY --chown=${APP_USER}:${APP_USER} . .

USER ${APP_USER}

ENV APP_PORT=${APP_PORT}
ENV PYTHONUNBUFFERED=1
ENV PYTHONDONTWRITEBYTECODE=1
ENV PATH="/usr/local/bin:${PATH}"

EXPOSE ${APP_PORT}

CMD ["sh", "-c", "uvicorn ${APP_MODULE} --host 0.0.0.0 --port ${APP_PORT}"]
