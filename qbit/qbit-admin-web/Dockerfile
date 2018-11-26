FROM perl:5.26.2-slim-threaded
LABEL maintainer="Keedi Kim <keedi@cpan.org>"

#
# for fast build
#
RUN mkdir -p /app/qbit
WORKDIR /app/qbit
COPY static/perl-5.26.2-site_perl-qbit.tar.gz ./
RUN tar xvzf perl-5.26.2-site_perl-qbit.tar.gz -C /usr/local/lib/perl5/site_perl

#
# preparing to build
#
RUN true \
    && apt-get update \
    && apt-get upgrade -y \
    && apt-get install -y --no-install-recommends \
        apt-utils \
    && apt-get install -y --no-install-recommends \
        gcc \
        libmariadbclient-dev \
        make \
        netbase \
    && cpanm \
        Minion \
        Minion::Backend::mysql \
        Mojolicious

#
# app specific
#
COPY static/QBit-Schema-0.001.tar.gz ./
COPY static/QBit-Admin-Web-0.001.tar.gz ./
RUN true \
    && cpanm QBit-Schema-0.001.tar.gz \
    && cpanm --installdeps QBit-Admin-Web-0.001.tar.gz \
    && tar xvzf QBit-Admin-Web-0.001.tar.gz

#
# cleaning
#
RUN true \
    && savedPackages="make netbase libmariadbclient18" \
    && apt-mark auto '.*' > /dev/null \
    && apt-mark manual $savedPackages \
    && apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false \
    && rm -fr /var/cache/apt/* /var/lib/apt/lists/* \
    && rm -fr ./cpanm /root/.cpanm /usr/src/perl /tmp/* \
    && rm -fr /app/qbit/*.tar.gz

#
# non-root
#
RUN true \
    && useradd -ms /bin/bash qbit \
    && chown -R qbit:qbit /app/qbit
USER qbit
WORKDIR /app/qbit/QBit-Admin-Web-0.001

# QBIT_PORT=
# QBIT_DATABASE_OPTS=
# QBIT_DATABASE_DSN=
# QBIT_DATABASE_NAME=
# QBIT_DATABASE_USER=
# QBIT_DATABASE_PASS=
# QBIT_COOKIE_DOMAIN=
# QBIT_COOKIE_NAME=
#ENV \
#    QBIT_PORT=3000 \
#    QBIT_DATABASE_DSN='dbi:mysql:con-qbit-db:localhost' \
#    QBIT_DATABASE_NAME='qbit' \
#    QBIT_DATABASE_USER='qbit' \
#    QBIT_DATABASE_PASS='qbit' \
#    QBIT_COOKIE_DOMAIN='qbit' \
#    QBIT_COOKIE_NAME='qbit'

#CMD ["perl5.26.2", "-de0"]
CMD ["bash"]
