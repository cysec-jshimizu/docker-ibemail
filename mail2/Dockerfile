FROM alpine:3.11
RUN apk --update add nginx postfix procmail postfix-pcre rsyslog supervisor && \
    apk add gcc g++ make curl && \
    apk add curl-dev openssl-dev gmp-dev boost-dev && \
    apk add git nano bash

RUN rm -rf /var/cache/apk/*

# build mimetic
RUN git clone -b fix-uint https://github.com/cysec-jshimizu/mimetic/ && \
    cd mimetic && \
    ./configure && \
    make install && \
    cd / && \
    rm -rf mimetic

# build mcl
RUN git clone https://github.com/herumi/mcl && \
    make MCL_USE_GMP=1 -j$(grep cpu.cores  /proc/cpuinfo | sort -u | sed 's/[^0-9]//g') -C /mcl && \
    make install -C /mcl && \
    rm -rf mcl

ENV LD_LIBRARY_PATH=/usr/local/lib:$LD_LIBRARY_PATH

# build ibemail
# COPY ibemail/ /ibemail
RUN git clone https://github.com/cysec-jshimizu/ibemail && \
    mkdir /var/cache/ibemail && \
    make -C /ibemail/idniks && \
    make all -C /ibemail/ibe_mail && \
    make install -C /ibemail/ibe_mail && \
    make -C /ibemail/ibe_mail_prog/kgc

# supervisor の設定ファイルをコピー
RUN touch /etc/supervisor
COPY supervisord.conf /etc/

# COPY postfix config
ADD main.cf /etc/postfix/main.cf
ADD mydestination /etc/postfix/mydestination

# nginx
COPY index.html /var/www/html/
COPY default.conf /etc/nginx/conf.d/

# sample mail
ADD mail.txt /root/mail.txt

# domain
ARG domain="example.fuga"
# generate public parameter, master secret key, user secret key
# publish public parameter at web server
RUN mkdir /root/.ibemail && \
    /ibemail/ibe_mail_prog/kgc/bin/kgc -s && \
    /ibemail/ibe_mail_prog/kgc/bin/kgc -g root@${domain} && \
    mkdir /var/www/html/.well-known &&\
    sed -n 1P ibemail.param > /var/www/html/.well-known/ibemail.txt && \
    cp ibemail.param /root/.ibemail/ && \
    mv ibemail.param  /ibemail/ibe_mail/files/params.txt && \
    mv root\@${domain}.userkey /root/.ibemail/${domain}.decryptkey && \
    echo ${domain} > /root/.ibemail/mydomain




CMD ["sh", "-c", "/usr/sbin/rsyslogd; newaliases; /usr/bin/supervisord", "-c", "/etc/supervisord.conf"]
