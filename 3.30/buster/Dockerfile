FROM debian:buster AS base

ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get update \
 && apt-get install -y --no-install-recommends \
    gcc \
    g++ \
    python \
    python3 \
    python3-dev \
 && rm -rf /var/lib/apt/lists/*

FROM base AS build

RUN apt-get update \
 && apt-get install -y --no-install-recommends \
    curl \
    ca-certificates \
    bzip2 \
 && rm -rf /var/lib/apt/lists/*

ARG VERSION=3.30.1

RUN mkdir /ns3 \
 && curl -sL https://www.nsnam.org/release/ns-allinone-$VERSION.tar.bz2 | tar xvj -C /ns3

RUN cd /ns3/ns-allinone-$VERSION && ./build.py --disable-netanim -- --prefix=/ns3/install
RUN cd /ns3/ns-allinone-$VERSION/ns-$VERSION && ./waf install

FROM base

COPY --from=build /ns3/install /usr/local
RUN ldconfig

ENV PYTHONPATH=/usr/local/lib/python3.7/site-packages

ENV DEBIAN_FRONTEND=
