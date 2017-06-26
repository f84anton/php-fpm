FROM php:7.1-fpm

RUN echo "memory_limit=-1" > $PHP_INI_DIR/conf.d/memory-limit.ini \
 && echo "date.timezone=${PHP_TIMEZONE:-Europe/Moscow}" > "$PHP_INI_DIR/conf.d/date_timezone.ini"

RUN ln -fs /usr/share/zoneinfo/${PHP_TIMEZONE:-Europe/Moscow} /etc/localtime
ENV DEV_PACKAGES "libfreetype6-dev libjpeg62-turbo-dev libmcrypt-dev libpng12-dev libbz2-dev \
    libxslt-dev libldap2-dev   libz-dev  libmemcached-dev libtidy-dev libcurl4-openssl-dev libc-client2007e-dev \
    libkrb5-dev libmagickwand-6.q16-dev libxml2-dev libedit-dev libicu-dev librecode-dev libxslt1-dev"
ENV DEBIAN_FRONTEND noninteractive

RUN apt-get update -q && \
   apt-get install -qy --no-install-recommends \
    curl \
    git \
    subversion \
    unzip \
    wget \
    openssh-client \
    libimage-exiftool-perl \
    libtidy-0.99 \
    libc-client2007e \
    libmagickwand-6.q16-2 \
    libmcrypt4 \
    libmemcached11 \
    librecode0 \
    libmemcachedutil2 \
    libxslt1.1 \
    && rm -r /var/lib/apt/lists/*


RUN apt-get update -q \
  && apt-get install -qy --no-install-recommends ${DEV_PACKAGES} \
  && CFLAGS="-I/usr/src/php" docker-php-ext-install bcmath mcrypt zip bz2 mbstring pcntl xsl tidy curl \
    pdo pdo_mysql dom xml xmlreader xmlrpc xmlwriter simplexml readline soap tokenizer \
    ctype calendar  sysvmsg sysvsem sysvshm shmop gettext  fileinfo  sockets opcache \
  && docker-php-ext-configure recode --with-recode && docker-php-ext-install recode \
  && docker-php-ext-configure exif --enable-exif && docker-php-ext-install exif \
  && docker-php-ext-configure intl --enable-intl &&  docker-php-ext-install intl \  
  && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
  && docker-php-ext-install gd \
  && docker-php-ext-configure imap --with-kerberos --with-imap-ssl && docker-php-ext-install imap \
  && echo 'autodetect' | pecl install -o -f imagick && docker-php-ext-enable imagick \
  && echo 'no' | pecl install -o -f memcached && docker-php-ext-enable memcached \
  && pecl install -o -f igbinary msgpack && docker-php-ext-enable igbinary msgpack \
  && pecl install -o -f redis && docker-php-ext-enable redis \
  && wget -q http://pecl.php.net/get/translit -O /tmp/translit && pecl install -o -f /tmp/translit && docker-php-ext-enable translit && rm -f /tmp/translit \
  && apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false ${DEV_PACKAGES} \
  && rm -r /var/lib/apt/lists/*


ENV PATH "/composer/vendor/bin:$PATH"
ENV COMPOSER_ALLOW_SUPERUSER 1
ENV COMPOSER_HOME /composer
ENV COMPOSER_VERSION 1.4.1

RUN curl -s -f -L -o /tmp/installer.php https://raw.githubusercontent.com/composer/getcomposer.org/da290238de6d63faace0343efbdd5aa9354332c5/web/installer \
 && php -r " \
    \$signature = '669656bab3166a7aff8a7506b8cb2d1c292f042046c5a994c43155c0be6190fa0355160742ab2e1c88d40d5be660b410'; \
    \$hash = hash('SHA384', file_get_contents('/tmp/installer.php')); \
    if (!hash_equals(\$signature, \$hash)) { \
        unlink('/tmp/installer.php'); \
        echo 'Integrity check failed, installer is either corrupt or worse.' . PHP_EOL; \
        exit(1); \
    }" \
    && php /tmp/installer.php --no-ansi --install-dir=/usr/bin --filename=composer --version=${COMPOSER_VERSION} \
    && rm /tmp/installer.php \
    && composer --ansi --version --no-interaction

RUN composer require -d /composer --ansi --no-interaction phpunit/php-invoker phpunit/dbunit

RUN git config --global user.email "yourci@gmail.com" && git config --global user.name "YourCI"

WORKDIR /app

ENV NEWRELIC_VER="php5-7.2.0.191-linux" \
    NR_INSTALL_SILENT=1 \
    NR_INSTALL_ARCH=x64 \
    NR_INSTALL_USE_CP_NOT_LN=1 \
    NEWRELICLICENSE="" \
    NEWRELICAPPNAME="Default App Name" \
    NEWRELICDAEMONDONTLAUHCH=3

RUN wget -q https://download.newrelic.com/php_agent/release/newrelic-${NEWRELIC_VER}.tar.gz -O /tmp/newrelic-agent.tar.gz \
    && tar xf /tmp/newrelic-agent.tar.gz -C /tmp/ \
    && /tmp/newrelic-${NEWRELIC_VER}/newrelic-install install \
    && sed -ri -e 's|;?newrelic.daemon.dont_launch =.+$|newrelic.daemon.dont_launch = ${NEWRELICDAEMONDONTLAUHCH}|g' \
               -e 's|;?newrelic.license =.+$|newrelic.license = ${NEWRELICLICENSE}|g' \
               -e 's|;?newrelic.logfile =.+$|newrelic.logfile = /proc/self/fd/2|g' \
               -e 's|;?newrelic.appname =.+$|newrelic.appname = ${NEWRELICAPPNAME}|g' \
               -e 's|;?newrelic.daemon.logfile =.+$|newrelic.daemon.logfile = /proc/self/fd/2|g' \
               -e 's|;?newrelic.capture_params =.+$|newrelic.capture_params = true|g' \
               -e 's|;?newrelic.error_collector.record_database_errors =.+$|newrelic.error_collector.record_database_errors = true|g' \
       /usr/local/etc/php/conf.d/newrelic.ini \
    && rm -rf /tmp/newrelic-* /tmp/nrinstall-*

ENV ERROR_LOG=/proc/self/fd/2 \
    LOG_LEVEL=notice \
    PROCESS_CONTROL_TIMEOUT=1 \
    FPM_LISTEN="0.0.0.0:9000" \
    LISTEN_BACKLOG=128 \
    PM_MAX_CHILDREN=50 \
    PM_MIN_SPARE_SERVERS=10 \
    PM_MAX_SPARE_SERVERS=25 \
    PM_MAX_REQUESTS=1000 \
    REQUEST_TERMINATE_TIMEOUT=0 \
    SLOW_LOG=/proc/self/fd/2 \
    RLIMIT_FILES=8192 \
    RLIMIT_CORE=0 \
    SESSION_SAVE_HANDLER=files \
    REQUEST_SLOWLOG_TIMEOUT=0 \
    EXPOSE_PHP=0 \
    PHP_ERROR_LOG=/tmp/php-error.log \
    PHP_FCGI_ERROR_HEADER="HTTP/1.0 550 Overload" \
    MAGIC_QUOTES=0 \
    DISPLAY_ERRORS=Off \
    DATE_TIMEZONE="Europe\Moscow" \
    SESSION_CACHE_EXPIRE=86400 \
    SESSION_SERIALIZE_HANDLER="php_binary" \
    MEMORY_LIMIT="-1" \
    UPLOAD_MAX_FILESIZE="500m" \
    MAX_EXECUTION_TIME=30 \
    MAX_INPUT_TIME="-1" \
    GC_PROBABILITY="1" \
    SESSION_SAVE_PATH="" \
    SESSION_SAVE_HANDLER="files" \
    MEMCACHED_SESS_LOCKING="Off"



COPY www.conf /usr/local/etc/php-fpm.d/
COPY php.ini /usr/local/etc/php/
