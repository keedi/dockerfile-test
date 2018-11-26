#
# docker build --tag keedi/perl-qbit-admin-web .
#
#   docker run -it --rm --name=con-extract keedi/perl-qbit-admin-web:latest /bin/bash
#       tar cvzf /tmp/app-perl.tar.gz -C /app perl
#
#   docker cp con-extract:/tmp/app-perl.tar.gz app-perl.tar.gz
#
FROM perl:5.26.2-slim-threaded
LABEL maintainer="Keedi Kim <keedi@cpan.org>"

ENV \
    DEBIAN_FRONTEND="noninteractive" \
    PATH="/app/perl/bin:$PATH" \
    PERL5LIB="/app/perl/lib/perl5:$PERL5LIB" \
    PERL_CPANM_OPT="-l /app/perl --mirror http://cpan.silex.kr" \
    PERL_LOCAL_LIB_ROOT="/app/perl" \
    PERL_MB_OPT="--install_base /app/perl" \
    PERL_MM_OPT="INSTALL_BASE=/app/perl"

#
# non-root
#
RUN true \
    && mkdir -p /app/perl /app/qbit \
    && useradd -ms /bin/bash qbit \
    && chown -R qbit:qbit /app/perl /app/qbit

#
# for fast build
#
USER qbit
WORKDIR /app/perl
COPY --chown=qbit:qbit app-perl.tar.gz ./
RUN true \
    && tar xvzf app-perl.tar.gz -C /app/perl --strip-components=1 \
    && rm -rf app-perl.tar.gz

#
# preparing to build
#
USER root
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
USER qbit
WORKDIR /app/qbit
COPY static/QBit-Schema-0.001.tar.gz ./
COPY static/QBit-Admin-Web-0.001.tar.gz ./
RUN true \
    && cpanm QBit-Schema-0.001.tar.gz \
    && cpanm --installdeps QBit-Admin-Web-0.001.tar.gz \
    && mkdir -p /app/qbit/qbit-admin-web \
    && tar xvzf QBit-Admin-Web-0.001.tar.gz -C /app/qbit/qbit-admin-web --strip-components=1

#
# cleaning
#
USER root
RUN true \
    && savedPackages="make netbase libmariadbclient18" \
    && apt-mark auto '.*' > /dev/null \
    && apt-mark manual $savedPackages \
    && apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false \
    && rm -fr /var/cache/apt/* /var/lib/apt/lists/* \
    && rm -fr ./cpanm /root/.cpanm /usr/src/perl /tmp/* \
    && rm -fr /app/perl/*.tar.gz /app/qbit/*.tar.gz

#
# ready to run
#
USER qbit
WORKDIR /app/qbit/qbit-admin-web
COPY q_bit-admin-web.conf ./
ENV PERL5LIB=/app/qbit/qbit-admin-web:$PERL5LIB
ENV MOJO_HOME=/app/qbit/qbit-admin-web
ENV MOJO_CONFIG=q_bit-admin-web.conf

#EXPOSE 3000
#CMD [ "morbo", "-l", "http://*:3000", "script/q_bit-admin-web" ]
CMD ["bash"]
