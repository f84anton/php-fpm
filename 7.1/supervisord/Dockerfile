FROM f84anton/php-fpm:7.1-xdebug

RUN apt-get update -qq && \
    apt-get install -qy \
    python-setuptools \
    libav-tools \
    libjpeg-turbo-progs \
    mediainfo \
    ghostscript \
    gsfonts \
    fonts-dejavu-core \
    poppler-utils \
    git \
    python2.7 \
    && rm -rf /var/lib/apt/lists/*
#    libjasper-runtime \

RUN composer require -d /composer --ansi --no-interaction phpunit/php-invoker phpunit/dbunit

RUN git config --global user.email "yourci@gmail.com" && git config --global user.name "YourCI"

ENV NODE_VERSION=8.2.1 \
  NPM_CONFIG_LOGLEVEL=info 

RUN curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION-linux-x64.tar.xz" \
  && tar -xJf "node-v$NODE_VERSION-linux-x64.tar.xz" -C /usr/local --strip-components=1 \
  && rm "node-v$NODE_VERSION-linux-x64.tar.xz" \
  && ln -s /usr/local/bin/node /usr/local/bin/nodejs \
  && npm install --global gulp-cli \
  && chown -R root. /usr/local/lib

RUN ln -sf /usr/bin/python2.7 /usr/bin/python

RUN git clone --recursive https://github.com/sass/node-sass.git /root/node-sass \
    && cd /root/node-sass && npm install && node scripts/build -f \
    && mkdir -p /usr/local/lib/node-sass/ \
    && cp vendor/linux-x64*/binding.node /usr/local/lib/node-sass/ \
    && rm -rf /root/node-sass

ENV SASS_BINARY_PATH=/usr/local/lib/node-sass/binding.node

RUN  easy_install \
     supervisor \
     youtube-dl

COPY supervisord.conf /etc/supervisor/supervisord.conf
VOLUME ["/etc/supervisor/conf.d"]
WORKDIR /etc/supervisor/conf.d

ENTRYPOINT ["supervisord", "--nodaemon", "--configuration", "/etc/supervisor/supervisord.conf"]
