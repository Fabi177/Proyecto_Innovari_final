FROM php:8.1-fpm

# Dependencias del sistema
RUN apt-get update && apt-get install -y \
    git \
    curl \
    libpng-dev \
    libjpeg-dev \
    libfreetype6-dev \
    libonig-dev \
    libzip-dev \
    zip \
    unzip \
 && docker-php-ext-configure gd --with-freetype --with-jpeg \
 && docker-php-ext-install pdo_mysql mbstring exif pcntl bcmath gd zip

# Composer (copiar desde la imagen oficial de composer)
COPY --from=composer:2 /usr/bin/composer /usr/bin/composer

WORKDIR /var/www/html

# Opcional: ajustar UID de www-data para que coincida con tu usuario en Windows host si hace falta
RUN usermod -u 1000 www-data || true

EXPOSE 9000
CMD ["php-fpm"]