
# Stage 1: Build React application
FROM node:22-alpine AS build

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

# Stage 2: Production Nginx image
FROM nginxinc/nginx-unprivileged:alpine AS production

# Temporarily switch to root to install security updates
USER root
RUN apk update && apk upgrade --no-cache

ENV BACKEND_HOST=backend
ENV BACKEND_PORT=5000

COPY --from=build /app/dist /usr/share/nginx/html
COPY default.conf.template /etc/nginx/templates/default.conf.template

EXPOSE 8080

# Run Nginx as non-root
USER nginx

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD wget -q -O /dev/null http://127.0.0.1:8080/ || exit 1
