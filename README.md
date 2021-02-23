# This repositry
Docker settings for MySQL 5.0.96

# Project plan
Do not change the original source code as much as possible

# Original source code links
* [Dockerfile & docker-entrypoint.sh](https://github.com/docker-library/mysql/tree/master/5.6)
* [binary install script](https://dev.mysql.com/doc/refman/5.6/en/binary-installation.html)

# Usage
```
$ bash -c "printf 'UID=%s\nGID=%s' `id -u` `id -g`" > .env
$ docker-compose up --build
```

# diff
## docker-entrypoint.sh
```
100c100,101
<               | awk -v conf="$conf" '$1 == conf && /^[^ \t]/ { sub(/^[^ \t]+[ \t]+/, ""); print; exit }'
---
>               | grep "^$conf" | awk -F' ' '{print $2}'
>               #| awk -v conf="$conf" '$1 == conf && /^[^ \t]/ { sub(/^[^ \t]+[ \t]+/, ""); print; exit }'
106c107
<       if [ "${MYSQL_MAJOR}" = '5.6' ] || [ "${MYSQL_MAJOR}" = '5.7' ]; then
---
>       if [ "${MYSQL_MAJOR}" = '5.0' ] || [ "${MYSQL_MAJOR}" = '5.6' ] || [ "${MYSQL_MAJOR}" = '5.7' ]; then
166c167,169
<       if [ "$MYSQL_MAJOR" = '5.6' ]; then
---
>       if [ "$MYSQL_MAJOR" = '5.0' ]; then
>               (cd /usr/local/mysql && scripts/mysql_install_db --datadir="$DATADIR" "${@:2}")
>       elif [ "$MYSQL_MAJOR" = '5.6' ]; then
248c251
<       if [ "$MYSQL_MAJOR" = '5.6' ]; then
---
>       if [ "$MYSQL_MAJOR" = '5.0' ] || [ "$MYSQL_MAJOR" = '5.6' ]; then
```

## Dockerfile
```
32c32
<       apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false; \
---
>       # apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false; \
52,60c52,53
< RUN set -ex; \
< # gpg: key 5072E1F5: public key "MySQL Release Engineering <mysql-build@oss.oracle.com>" imported
<       key='A4A9406876FCBD3C456770C88C718D3B5072E1F5'; \
<       export GNUPGHOME="$(mktemp -d)"; \
<       gpg --batch --keyserver ha.pool.sks-keyservers.net --recv-keys "$key"; \
<       gpg --batch --export "$key" > /etc/apt/trusted.gpg.d/mysql.gpg; \
<       gpgconf --kill all; \
<       rm -rf "$GNUPGHOME"; \
<       apt-key list > /dev/null
---
> ENV MYSQL_MAJOR 5.0
> ENV MYSQL_VERSION mysql-5.0.96-linux-x86_64-glibc23
62,84c55,69
< ENV MYSQL_MAJOR 5.6
< ENV MYSQL_VERSION 5.6.51-1debian9
<
< RUN echo 'deb http://repo.mysql.com/apt/debian/ stretch mysql-5.6' > /etc/apt/sources.list.d/mysql.list
<
< # the "/var/lib/mysql" stuff here is because the mysql-server postinst doesn't have an explicit way to disable the mysql_install_db codepath besides having a database already "configured" (ie, stuff in /var/lib/mysql/mysql)
< # also, we set debconf keys to make APT a little quieter
< RUN { \
<               echo mysql-community-server mysql-community-server/data-dir select ''; \
<               echo mysql-community-server mysql-community-server/root-pass password ''; \
<               echo mysql-community-server mysql-community-server/re-root-pass password ''; \
<               echo mysql-community-server mysql-community-server/remove-test-db select false; \
<       } | debconf-set-selections \
<       && apt-get update \
<       && apt-get install -y \
<               mysql-server="${MYSQL_VERSION}" \
< # comment out a few problematic configuration values
<       && find /etc/mysql/ -name '*.cnf' -print0 \
<               | xargs -0 grep -lZE '^(bind-address|log)' \
<               | xargs -rt -0 sed -Ei 's/^(bind-address|log)/#&/' \
< # don't reverse lookup hostnames, they are usually another container
<       && echo '[mysqld]\nskip-host-cache\nskip-name-resolve' > /etc/mysql/conf.d/docker.cnf \
<       && rm -rf /var/lib/apt/lists/* \
---
> # https://dev.mysql.com/doc/refman/5.6/en/binary-installation.html
> ENV PATH /usr/local/mysql/bin:$PATH
> WORKDIR /usr/local
> RUN wget https://downloads.mysql.com/archives/get/p/23/file/${MYSQL_VERSION}.tar.gz \
>       # && groupadd mysql \
>       # && useradd -r -g mysql mysql \
>       && tar zxvf ${MYSQL_VERSION}.tar.gz \
>       && ln -s /usr/local/${MYSQL_VERSION} mysql \
>       && rm ${MYSQL_VERSION}.tar.gz \
>       && cd mysql \
>       && chown -R mysql . \
>       && chgrp -R mysql . \
>       # && scripts/mysql_install_db --datadir=/var/lib/mysql --user=mysql \
>       # && chown -R root . \
>       # && chown -R mysql data \
```