FROM php:8.3-fpm

WORKDIR /var/www

RUN apt-get update && apt-get install -y \
    git \
    unzip \
    libpq-dev \
    && docker-php-ext-install pdo_pgsql \
    && rm -rf /var/lib/apt/lists/*

# Cria usuário dev com UID/GID 1000
RUN groupadd -g 1000 dev \
    && useradd -u 1000 -g 1000 -m -s /bin/bash dev

COPY --from=composer:2 /usr/bin/composer /usr/bin/composer

COPY . .

RUN composer install

COPY docker/php.ini /usr/local/etc/php/conf.d/99-custom.ini

RUN mkdir -p \
    storage/framework/views \
    storage/framework/cache \
    storage/framework/sessions \
    bootstrap/cache \
    && chown -R dev:dev /var/www \
    && chmod -R 775 storage bootstrap/cache

EXPOSE 9000

USER dev

CMD ["php-fpm", "-F"]
