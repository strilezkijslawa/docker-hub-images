FROM php:8.3-fpm-alpine

ENV TAR_OPTIONS="--no-same-owner --no-same-permissions"

RUN apk add --no-cache \
    curl \
    git \
    unzip \
    bash \
    jpegoptim \
    optipng \
    pngquant \
    gifsicle \
    imagemagick \
    imagemagick-webp \
    icu-libs \
    icu-data-full \
    libzip \
    gmp \
    libpng \
    libjpeg-turbo \
    libwebp \
    libwebp-tools \
    freetype \
    oniguruma \
    libxml2

RUN apk add --no-cache --virtual .build-deps \
        $PHPIZE_DEPS \
        curl-dev \
        pkgconf \
        oniguruma-dev \
        icu-dev \
        libxml2-dev \
        libzip-dev \
        libpng-dev \
        libjpeg-turbo-dev \
        libwebp-dev \
        freetype-dev \
        imagemagick-dev \
        gmp-dev \
        linux-headers \
    && docker-php-ext-configure gd \
        --with-freetype \
        --with-jpeg \
        --with-webp \
    && docker-php-ext-install -j"$(nproc)" \
        intl \
        mbstring \
        curl \
        xml \
        zip \
        gd \
        pdo_mysql \
        mysqli \
        ftp \
        sockets \
        exif \
        bcmath \
        gmp \
    && pecl install imagick redis \
    && docker-php-ext-enable imagick redis \
    && apk del .build-deps \
    && rm -rf /var/cache/apk/* /tmp/* /var/tmp/*

COPY --from=composer:latest /usr/bin/composer /usr/local/bin/composer

WORKDIR /www

COPY ./php.ini /usr/local/etc/php/php.ini
COPY ./imagemagick/policy.xml /etc/ImageMagick-7/policy.xml

RUN ! php -m 2>&1 | grep -iE 'warning|error'

EXPOSE 9000

CMD ["php-fpm"]
