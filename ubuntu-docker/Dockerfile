FROM ubuntu:18.04

RUN apt-get update && \
    apt-get install -y curl gcc libpcre3 libpcre3-dev zlib1g zlib1g-dev libssl-dev build-essential && \
    apt-get -q clean -y && \
    rm -rf /var/lib/apt/lists/*

CMD [ "/bin/bash" ]