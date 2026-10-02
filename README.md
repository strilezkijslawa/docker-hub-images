# PHP-FPM Alpine images

Dockerfiles для образів PHP-FPM на Alpine, що публікуються на Docker Hub у `strilezkijslawa/`.

## Образи

| PHP | Базовий | + LDAP | + LDAP + Oracle (oci8) |
|-----|---------|--------|------------------------|
| 8.2 | `strilezkijslawa/php8.2-alpine` | `strilezkijslawa/php8.2-ldap-alpine` | `strilezkijslawa/php8.2-oci-alpine` |
| 8.3 | `strilezkijslawa/php8.3-alpine` | `strilezkijslawa/php8.3-ldap-alpine` | `strilezkijslawa/php8.3-oci-alpine` |
| 8.4 | `strilezkijslawa/php8.4-alpine` | `strilezkijslawa/php8.4-ldap-alpine` | `strilezkijslawa/php8.4-oci-alpine` |
| 8.5 | `strilezkijslawa/php8.5-alpine` | `strilezkijslawa/php8.5-ldap-alpine` | `strilezkijslawa/php8.5-oci-alpine` |

Усі образи публікуються з тегом `latest`.

Папки: `strilezkijslawa-php8X-alpine-<suffix>`, `...-alpine-ldap-<suffix>`, `...-alpine-oci-<suffix>`, де суфікс `stable` для 8.4 і `latest` для решти версій.

### Що всередині

- **Базовий** (`php:8.X-fpm-alpine`): розширення intl, mbstring, curl, xml, zip, gd (freetype, jpeg, webp), pdo_mysql, mysqli, ftp, sockets, exif, bcmath, gmp, imagick, redis; Composer; утиліти оптимізації зображень (jpegoptim, optipng, pngquant, gifsicle, cwebp); ImageMagick з кодеком WebP (`imagemagick-webp`: в Alpine кодеки — окремі пакети); git, unzip, bash. Власний `php.ini` і політика ImageMagick (`imagemagick/policy.xml`). `WORKDIR /www`, php-fpm на порту 9000.
- **LDAP**: базовий + розширення `ldap`.
- **OCI**: LDAP + Oracle Instant Client 21.9 (завантажується з oracle.com під час збірки) + розширення `oci8`.

### Політика ImageMagick

У базових образах діють обмеження з `imagemagick/policy.xml`:
- максимальна ширина/висота зображення — 16000px;
- memory 256MiB, map 512MiB, disk 1GiB;
- вимкнені формати PS/PS2/PS3/EPS/XPS, делегати URL/HTTP/HTTPS та непрямі читання (`@file`).

## Перезбірка образів

Образи залежать один від одного через теги Docker Hub:

```
php:8.X-fpm-alpine → php8.X-alpine → php8.X-ldap-alpine → php8.X-oci-alpine
```

LDAP- та OCI-образи збираються `FROM ...:latest` батьківського образу, тому:

1. Збирати строго по порядку: базовий → ldap → oci.
2. Базовий образ збирати з `--pull`, щоб підтягнути свіжий `php:8.X-fpm-alpine`.
3. **Не** використовувати `--pull` для ldap та oci: інакше Docker завантажить старий батьківський образ із Docker Hub замість щойно зібраного локально.

### PHP 8.2

```bash
docker build --pull -t strilezkijslawa/php8.2-alpine:latest      strilezkijslawa-php82-alpine-latest
docker build        -t strilezkijslawa/php8.2-ldap-alpine:latest strilezkijslawa-php82-alpine-ldap-latest
docker build        -t strilezkijslawa/php8.2-oci-alpine:latest  strilezkijslawa-php82-alpine-oci-latest
```

```bash
docker push strilezkijslawa/php8.2-alpine:latest
docker push strilezkijslawa/php8.2-ldap-alpine:latest
docker push strilezkijslawa/php8.2-oci-alpine:latest
```

### PHP 8.3

```bash
docker build --pull -t strilezkijslawa/php8.3-alpine:latest      strilezkijslawa-php83-alpine-latest
docker build        -t strilezkijslawa/php8.3-ldap-alpine:latest strilezkijslawa-php83-alpine-ldap-latest
docker build        -t strilezkijslawa/php8.3-oci-alpine:latest  strilezkijslawa-php83-alpine-oci-latest
```

```bash
docker push strilezkijslawa/php8.3-alpine:latest
docker push strilezkijslawa/php8.3-ldap-alpine:latest
docker push strilezkijslawa/php8.3-oci-alpine:latest
```

### PHP 8.4

```bash
docker build --pull -t strilezkijslawa/php8.4-alpine:latest      strilezkijslawa-php84-alpine-stable
docker build        -t strilezkijslawa/php8.4-ldap-alpine:latest strilezkijslawa-php84-alpine-ldap-stable
docker build        -t strilezkijslawa/php8.4-oci-alpine:latest  strilezkijslawa-php84-alpine-oci-stable
```

```bash
docker push strilezkijslawa/php8.4-alpine:latest
docker push strilezkijslawa/php8.4-ldap-alpine:latest
docker push strilezkijslawa/php8.4-oci-alpine:latest
```

### PHP 8.5

```bash
docker build --pull -t strilezkijslawa/php8.5-alpine:latest      strilezkijslawa-php85-alpine-latest
docker build        -t strilezkijslawa/php8.5-ldap-alpine:latest strilezkijslawa-php85-alpine-ldap-latest
docker build        -t strilezkijslawa/php8.5-oci-alpine:latest  strilezkijslawa-php85-alpine-oci-latest
```

```bash
docker push strilezkijslawa/php8.5-alpine:latest
docker push strilezkijslawa/php8.5-ldap-alpine:latest
docker push strilezkijslawa/php8.5-oci-alpine:latest
```

Щоб також оновити пакети Alpine, а не брати їх із кешу шарів, додайте `--no-cache` до команди збірки базового образу. Для пушу потрібен `docker login`.

### Перевірка

Кожен Dockerfile завершується `RUN ! php -m 2>&1 | grep -iE 'warning|error'`: збірка падає, якщо будь-яке розширення видає warning або error під час старту. Після збірки можна вручну перевірити версію та розширення:

```bash
docker run --rm strilezkijslawa/php8.4-oci-alpine:latest php -v
docker run --rm strilezkijslawa/php8.4-oci-alpine:latest php -m
```

## Додавання нової версії PHP

1. Скопіювати три папки попередньої версії.
2. Змінити `FROM php:8.X-fpm-alpine` у базовому Dockerfile і теги батьківських образів у ldap/oci.
3. За потреби оновити `ARG OCI8_VERSION` в oci-образі (версії oci8 прив'язані до версій PHP, див. https://pecl.php.net/package/oci8).
4. Зібрати та перевірити, як описано вище.
