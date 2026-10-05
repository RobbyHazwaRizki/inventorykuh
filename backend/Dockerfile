FROM php:8.3-cli

# Paket sistem + ekstensi PHP yang dibutuhkan Laravel, PostgreSQL, dan Redis
RUN apt-get update && apt-get install -y --no-install-recommends \
        git unzip libpq-dev libzip-dev \
    && docker-php-ext-install pdo_pgsql pgsql zip pcntl bcmath \
    && pecl install redis \
    && docker-php-ext-enable redis \
    && rm -rf /var/lib/apt/lists/*

# Composer (pengelola paket PHP)
COPY --from=composer:2 /usr/bin/composer /usr/bin/composer

WORKDIR /var/www/html