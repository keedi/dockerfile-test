#
# docker build --tag keedi/perl-5.26.2 .
# docker build --network net-cache --tag keedi/perl-5.26.2 .
#
#   docker run -it --rm --name=con-extract keedi/perl-5.26.2:latest /bin/bash
#       tar cvzf /tmp/app-perl.tar.gz -C /app perl
#
#   docker cp con-extract:/tmp/app-perl.tar.gz app-perl.tar.gz
#
FROM perl:5.26.2-slim-threaded
LABEL maintainer="Keedi Kim <keedi@cpan.org>"

ENV DEBIAN_FRONTEND="noninteractive"

#
# apt caching
#
COPY sources.list /etc/apt/sources.list
RUN true \
    && apt-get update \
    && apt-get install -y squid-deb-proxy-client
COPY apt-proxy.conf /etc/apt/apt.conf.d/30autoproxy
RUN apt-get update

#
# for perl
#
RUN mkdir -p /app/perl
ENV \
    PATH="/app/perl/bin:$PATH" \
    PERL5LIB="/app/perl/lib/perl5:$PERL5LIB" \
    PERL_CPANM_OPT="-l /app/perl --mirror http://cpan.silex.kr" \
    PERL_LOCAL_LIB_ROOT="/app/perl" \
    PERL_MB_OPT="--install_base /app/perl" \
    PERL_MM_OPT="INSTALL_BASE=/app/perl"

#
# for fast build
#
COPY app-perl.tar.gz /tmp
RUN true \
    && tar xvzf /tmp/app-perl.tar.gz -C /app/perl --strip-components=1 \
    && rm -rf app-perl.tar.gz

#
# preparing to build
#
RUN true \
    && apt-get update \
    && apt-get upgrade -y \
    && apt-get install -y --no-install-recommends \
        apt-utils

#
# build tool & lib
#
RUN true \
    && apt-get install -y --no-install-recommends \
        gcc \
        make \
        netbase \
        libc6-dev

#
# Term::ReadLine
#
RUN true \
    && apt-get install -y --no-install-recommends \
        libncursesw5-dev \
        libreadline-dev \
    && cpanm \
        Term::ReadLine \
        Term::ReadLine::Gnu \
    && cpanm --notest Term::ReadLine::Perl

#
# TLS/SSL
#
RUN true \
    && apt-get install -y --no-install-recommends \
        ca-certificates \
        libssl-dev \
        openssl \
        zlib1g-dev \
    && cpanm \
        Mozilla::CA \
        Net::SSLeay \
        IO::Socket::SSL \
    && NO_NETWORK_TESTING=1 cpanm \
        Net::HTTP

#
# DBI & DBD::*
#
RUN true \
    && apt-get install -y --no-install-recommends \
        libmariadbclient-dev \
        libpq-dev \
    && cpanm \
        DBI \
        DBD::SQLite \
        DBD::mysql \
        DBD::Pg

#
# Minion & Mojolicious
#
RUN true \
    && cpanm \
        Mojolicious \
        Minion \
        Minion::Backend::mysql

#
# Mandatory Module: light-weight
#
RUN true \
    && cpanm \
        Capture::Tiny \
        Class::Tiny \
        DateTime::Tiny \
        HTTP::Tiny \
        Log::Log4perl::Tiny \
        Path::Tiny \
        Try::Tiny

#
# Mandatory Module: heavy-weight
#
RUN cpanm Moo
RUN cpanm Moose
RUN cpanm DBIx::Class
RUN cpanm DBIx::Class::Schema::Loader
RUN cpanm DateTime
RUN cpanm Log::Log4perl
RUN cpanm Plack
RUN cpanm Ubic
RUN cpanm Ubic::Service::Hypnotoad
RUN cpanm Dist::Zilla
RUN true \
    && apt-get install -y \
        git \
        rename \
    && cpanm Dist::Zilla::PluginBundle::DAGOLDEN

#
# cleaning
#
RUN true \
    && savedPackages="ca-certificates libc6 libmariadbclient18 libncursesw5 libpq5 libreadline7 make netbase openssl zlib1g" \
    && apt-mark auto '.*' > /dev/null \
    && apt-mark manual $savedPackages \
    && apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false \
    && rm -fr /var/cache/apt/* /var/lib/apt/lists/* \
    && rm -fr ./cpanm /root/.cpanm /usr/src/perl /tmp/*

CMD [ "bash" ]
