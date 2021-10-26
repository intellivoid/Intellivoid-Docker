FROM ubuntu:18.04
EXPOSE 5600 5601 5602 5603 5604 5605 5606
MAINTAINER Diederik Noordhuis (diederikn@intellivoid.net)

ARG DEBIAN_FRONTEND=noninteractive
COPY Docker/bootstrap /usr/local/bin/bootstrap

RUN \
    apt-get update && \
    ln -fs /usr/share/zoneinfo/UTC /etc/localtime && \
    apt-get update && \
    apt-get install git make sudo nano vim wget curl unzip -y && \
    apt-get clean

CMD ["/bin/bash", "/usr/local/bin/bootstrap"]