# syntax=docker/dockerfile:1
ARG MAJOR_VERSION=6.4
ARG ZBX_VERSION=${MAJOR_VERSION}.11
ARG BUILD_BASE_IMAGE=zabbix-build-mysql:alpine-${ZBX_VERSION}

FROM ${BUILD_BASE_IMAGE} AS builder

FROM alpine:3.17

ARG MAJOR_VERSION
ARG ZBX_VERSION
ARG ZBX_SOURCES=https://git.zabbix.com/scm/zbx/zabbix.git

ENV TERM=xterm \
    ZBX_VERSION=${ZBX_VERSION} ZBX_SOURCES=${ZBX_SOURCES}

LABEL org.opencontainers.image.authors="Alexey Pustovalov <alexey.pustovalov@zabbix.com>" \
      org.opencontainers.image.description="Zabbix web-interface based on Apache2 web server with MySQL database support" \
      org.opencontainers.image.documentation="https://www.zabbix.com/documentation/${MAJOR_VERSION}/manual/installation/containers" \
      org.opencontainers.image.licenses="GPL v2.0" \
      org.opencontainers.image.source="${ZBX_SOURCES}" \
      org.opencontainers.image.title="Zabbix web-interface (Apache, MySQL)" \
      org.opencontainers.image.url="https://zabbix.com/" \
      org.opencontainers.image.vendor="Zabbix LLC" \
      org.opencontainers.image.version="${ZBX_VERSION}"

STOPSIGNAL SIGTERM

COPY --from=builder ["/tmp/zabbix-${ZBX_VERSION}/ui", "/usr/share/zabbix"]
COPY ["conf/etc/", "/etc/"]

RUN set -eux && \
    INSTALL_PKGS="bash \
            apache2 \
            curl \
            mariadb-client \
            mariadb-connector-c \
            php81-apache2 \
            php81-bcmath \
            php81-ctype \
            php81-gd \
            php81-gettext \
            php81-json \
            php81-ldap \
            php81-mbstring \
            php81-mysqli \
            php81-session \
            php81-simplexml \
            php81-sockets \
            php81-fileinfo \
            php81-xmlreader \
            php81-xmlwriter \
            php81-openssl" && \
    apk add \
            --no-cache \
            --clean-protected \
        ${INSTALL_PKGS} && \
    apk add \
            --clean-protected \
            --no-cache \
            --no-scripts \
        apache2-ssl && \
    addgroup \
            --system \
            --gid 1995 \
        zabbix && \
    adduser \
            --system \
            --gecos "Zabbix monitoring system" \
            --disabled-password \
            --uid 1997 \
            --ingroup zabbix \
            --shell /sbin/nologin \
            --home /var/lib/zabbix/ \
        zabbix && \
    mkdir -p /etc/zabbix && \
    mkdir -p /etc/zabbix/web && \
    mkdir -p /etc/zabbix/web/certs && \
    rm -f "/etc/apache2/conf.d/default.conf" && \
    rm -f "/etc/apache2/conf.d/ssl.conf" && \
    sed -ri \
            -e 's!^(\s*CustomLog)\s+\S+!\1 /proc/self/fd/1!g' \
            -e 's!^(\s*ErrorLog)\s+\S+!\1 /proc/self/fd/2!g' \
        "/etc/apache2/httpd.conf" && \
    sed -ri \
            -e 's!^(\s*PidFile)\s+\S+!\1 "/tmp/httpd.pid"!g' \
        "/etc/apache2/conf.d/mpm.conf" && \
    sed -i 's/Listen 80/Listen 8080/g' /etc/apache2/httpd.conf && \
    rm -rf "/var/run/apache2/" && \
    cd /usr/share/zabbix/ && \
    rm -f conf/zabbix.conf.php conf/maintenance.inc.php conf/zabbix.conf.php.example && \
    rm -rf tests && \
    rm -f locale/add_new_language.sh locale/update_po.sh locale/make_mo.sh && \
    find /usr/share/zabbix/locale -name '*.po' | xargs rm -f && \
    find /usr/share/zabbix/locale -name '*.sh' | xargs rm -f && \
    ln -s "/etc/zabbix/web/zabbix.conf.php" "/usr/share/zabbix/conf/zabbix.conf.php" && \
    ln -s "/etc/zabbix/web/maintenance.inc.php" "/usr/share/zabbix/conf/maintenance.inc.php" && \
    chown --quiet -R zabbix:root /etc/zabbix/ /usr/share/zabbix/include/defines.inc.php /usr/share/zabbix/modules/ && \
    chgrp -R 0 /etc/zabbix/ /usr/share/zabbix/include/defines.inc.php /usr/share/zabbix/modules/ && \
    chmod -R g=u /etc/zabbix/ /usr/share/zabbix/include/defines.inc.php /usr/share/zabbix/modules/ && \
    chown --quiet -R zabbix:root /etc/apache2/ /etc/php81/ && \
    chgrp -R 0 /etc/apache2/ /etc/php81/ && \
    chmod -R g=u /etc/apache2/ /etc/php81/ && \
    rm -rf /var/cache/apk/*

EXPOSE 8080/TCP 8443/TCP

WORKDIR /usr/share/zabbix

COPY ["docker-entrypoint.sh", "/usr/bin/"]

USER 1997

ENTRYPOINT ["docker-entrypoint.sh"]

CMD ["/usr/sbin/httpd", "-D", "FOREGROUND"]
