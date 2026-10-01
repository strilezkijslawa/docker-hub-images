FROM strilezkijslawa/php8.5-ldap-alpine:latest

# Oracle Instant Client (basic + sdk) is downloaded from oracle.com instead of being stored in git
ARG ORACLE_IC_VERSION=21.9.0.0.0dbru
ARG ORACLE_IC_DIR=219000
ARG OCI8_VERSION=3.4.1

ENV ORACLE_HOME=/opt/oracle/instantclient
ENV LD_LIBRARY_PATH=/opt/oracle/instantclient

# Instant Client is built against glibc, so Alpine (musl) needs gcompat
RUN apk add --no-cache libaio libnsl gcompat \
    && apk add --no-cache --virtual .oci-build-deps \
        $PHPIZE_DEPS \
    && mkdir -p /opt/oracle \
    && cd /tmp \
    && for pkg in basic sdk; do \
         curl -fsSL -o instantclient-$pkg.zip \
           "https://download.oracle.com/otn_software/linux/instantclient/${ORACLE_IC_DIR}/instantclient-$pkg-linux.x64-${ORACLE_IC_VERSION}.zip" \
         && unzip -qo instantclient-$pkg.zip -d /opt/oracle || exit 1; \
       done \
    && mv /opt/oracle/instantclient_* $ORACLE_HOME \
    && echo "instantclient,$ORACLE_HOME" | pecl install oci8-${OCI8_VERSION} \
    && docker-php-ext-enable oci8 \
    && rm -rf $ORACLE_HOME/sdk $ORACLE_HOME/*.jar \
    && apk del .oci-build-deps \
    && rm -rf /var/cache/apk/* /tmp/* /var/tmp/*

RUN ! php -m 2>&1 | grep -iE 'warning|error'
