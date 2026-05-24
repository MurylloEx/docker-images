FROM node:lts-alpine AS builder

WORKDIR /app

COPY package.json package-lock.json* ./

RUN npm ci

COPY . .

RUN npm run build

FROM node:lts-alpine AS runtime

ARG APP_PORT=8000
ARG APP_UID=1000
ARG APP_USER=nestjs

RUN deluser node 2>/dev/null || true \
    && delgroup node 2>/dev/null || true \
    && addgroup -g ${APP_UID} -S nodejs \
    && adduser -S ${APP_USER} -u ${APP_UID} -G nodejs

WORKDIR /app

COPY package.json package-lock.json* ./

RUN npm ci --omit=dev && npm cache clean --force

COPY --from=builder --chown=${APP_USER}:nodejs /app/dist ./dist

USER ${APP_USER}

ENV APP_PORT=${APP_PORT}
ENV NODE_ENV=production

EXPOSE ${APP_PORT}

CMD ["sh", "-c", "node dist/main.js"]
