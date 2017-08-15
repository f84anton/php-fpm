FROM f84anton/php-fpm:7.1

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
    libjasper-runtime \
    && rm -rf /var/lib/apt/lists/*

RUN  easy_install \
     supervisor \
     youtube-dl

COPY supervisord.conf /etc/supervisor/supervisord.conf
VOLUME ["/etc/supervisor/conf.d"]
WORKDIR /etc/supervisor/conf.d

ENTRYPOINT ["supervisord", "--nodaemon", "--configuration", "/etc/supervisor/supervisord.conf"]
