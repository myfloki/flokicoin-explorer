FROM node:24 AS builder

ARG commitHash
ENV DOCKER_COMMIT_HASH=${commitHash}
ENV CYPRESS_INSTALL_BINARY=0

RUN apt-get update
RUN apt-get install -y build-essential rsync

WORKDIR /build
COPY . .

RUN cp mempool-frontend-config.sample.json mempool-frontend-config.json
RUN npm install --omit=dev --omit=optional

RUN npm run build

FROM nginx:1.29.0-alpine
LABEL org.opencontainers.image.source="https://github.com/myfloki/operator-plus"

WORKDIR /patch

# Copy runtime scripts from docker/ context and built app from builder
COPY --from=docker frontend/entrypoint.sh .
COPY --from=docker frontend/wait-for .
COPY --from=builder /build/dist/mempool /var/www/mempool

# Copy nginx configs from repo root context
COPY --from=root nginx.conf /etc/nginx/
# nginx.conf expects this file at /etc/nginx/nginx-mempool.conf
COPY --from=root nginx-mempool.conf /etc/nginx/
COPY --from=root nginx-mempool.conf /etc/nginx/conf.d/nginx-mempool.conf

RUN chmod +x /patch/entrypoint.sh
RUN chmod +x /patch/wait-for

RUN chown -R 1000:1000 /patch && chmod -R 755 /patch && \
        chown -R 1000:1000 /var/cache/nginx && \
        chown -R 1000:1000 /var/log/nginx && \
        chown -R 1000:1000 /etc/nginx/nginx.conf && \
        chown -R 1000:1000 /etc/nginx/conf.d && \
        chown -R 1000:1000 /var/www/mempool

RUN touch /var/run/nginx.pid && \
        chown -R 1000:1000 /var/run/nginx.pid

USER 1000

ENTRYPOINT ["/patch/entrypoint.sh"]
CMD ["nginx", "-g", "daemon off;"]
