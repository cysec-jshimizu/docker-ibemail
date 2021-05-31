# docker build -t test-nsd .
# docker run --dns=172.30.0.4 -itd --name nameserver test-nsd

FROM ubuntu:18.04

RUN \
    apt update -qq && \
    apt -y upgrade && \
    apt install -y nsd && \
    apt install -y dnsutils iputils-ping net-tools  && \
    apt install -y curl nano neofetch nmap sudo

RUN mkdir /etc/nsd/zone/
COPY nsd.conf /etc/nsd/nsd.conf
COPY zone/ /etc/nsd/zone/
COPY run.sh /

WORKDIR /
EXPOSE 53/tcp 53/udp
RUN chmod +x /run.sh

CMD ["/run.sh"]
