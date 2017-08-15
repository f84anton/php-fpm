FROM f84anton/php-fpm:7.1

RUN apt-get update -qq && \
  apt-get install -qy python-setuptools && \
  easy_install supervisor && \
  rm -rf /var/lib/apt/lists/*

COPY supervisord.conf /etc/supervisor/supervisord.conf
VOLUME ["/etc/supervisor/conf.d"]
WORKDIR /etc/supervisor/conf.d

ENTRYPOINT ["supervisord", "--nodaemon", "--configuration", "/etc/supervisor/supervisord.conf"]
