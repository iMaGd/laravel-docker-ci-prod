# ──────────────────────────────────────────────
# Stage 1: Building frontend assets
# ──────────────────────────────────────────────
FROM node:23-alpine AS node-builder

WORKDIR /app

# Only copy what’s needed for building assets
COPY package.json package-lock.json ./
RUN npm ci --no-audit --no-fund

COPY resources/ resources/
COPY vite.config.* ./
COPY tailwind.config.* ./
COPY postcss.config.* ./

RUN npm run build


# ──────────────────────────────────────────────
# Stage 2: Building PHP Laravel app image
# ──────────────────────────────────────────────

FROM depicter/php:8.3-fpm-alpine

# Set working directory
WORKDIR /var/www
RUN rm -rf html

COPY . .

# Download dependencies as a separate step to take advantage of Docker's caching.
RUN sed 's_@php artisan package:discover_/bin/true_;' -i composer.json \
    && composer install --ignore-platform-req=php --no-dev --optimize-autoloader \
    && composer clear-cache \
    && php artisan package:discover --ansi \
    && chmod -R 775 storage \
    && chown -R www-data:www-data storage \
    && mkdir -p storage/framework/sessions storage/framework/views storage/framework/cache/data

# Exclude temp files
RUN rm -rf storage/app/private/*
RUN rm -rf storage/app/tmp/*
RUN rm -rf storage/app/livewire-tmp/*

# Copy frontend assets from node-builder stage
COPY --from=node-builder /app/public /var/www/public

# Permissions
RUN chmod -R ug+rwx storage public bootstrap/cache

# Copy entrypoint scripts
COPY ./ci/docker/php/entrypoint.sh /usr/local/bin/entrypoint.sh

# Give permisisons to everything in bin/
RUN chmod a+x /usr/local/bin/entrypoint.sh

# Ensure the symlink exists
RUN php artisan storage:link

# optimize performance
RUN php artisan optimize

CMD ["php-fpm"]
