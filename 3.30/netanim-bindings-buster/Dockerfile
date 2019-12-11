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

# Additional dependencies for generating NS3 bindings
RUN apt-get update \
 && apt-get install -y --no-install-recommends \
        build-essential \
        make cmake git mercurial unzip llvm-dev libclang-dev \
        python3-setuptools python3-wheel python3-pip \
 && rm -rf /var/lib/apt/lists/*

RUN pip3 install git+https://github.com/felix-gohla/pygccxml@v1.9.2 cxxfilt

RUN git clone https://github.com/CastXML/CastXML.git /tmp/CastXML \
 && cd /tmp/CastXML \
 && cmake -DCMAKE_INSTALL_PREFIX=/usr/local/ . \
 && make && make install

ARG VERSION=3.30.1

RUN mkdir /ns3 \
 && curl -sL https://www.nsnam.org/release/ns-allinone-$VERSION.tar.bz2 | tar xvj -C /ns3

# Update the netanim python bindings
ADD netanim_python_$VERSION.patch .
RUN patch /ns3/ns-allinone-$VERSION/ns-$VERSION/src/netanim/wscript netanim_python_${VERSION}.patch

RUN cd /ns3/ns-allinone-$VERSION/ns-$VERSION \
 && ./waf configure \
 && ./waf --apiscan=netanim

RUN cd /ns3/ns-allinone-$VERSION && ./build.py --disable-netanim -- --prefix=/ns3/install
RUN cd /ns3/ns-allinone-$VERSION/ns-$VERSION && ./waf install

FROM base

COPY --from=build /ns3/install /usr/local
RUN ldconfig

ENV PYTHONPATH=/usr/local/lib/python3.7/site-packages

ENV DEBIAN_FRONTEND=
