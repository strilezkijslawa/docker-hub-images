FROM strilezkijslawa/php8.4-alpine:latest

RUN apk add --no-cache openldap \
    && apk add --no-cache --virtual .ldap-build-deps \
        openldap-dev \
    && docker-php-ext-install ldap \
    && apk del .ldap-build-deps

RUN ! php -m 2>&1 | grep -iE 'warning|error'
