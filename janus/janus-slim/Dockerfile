# Using static version FROM debian:buster-slim AS prebuilder
#FROM debian@sha256:1b8a66f829a3563f98f941f4e0e2ecc619ad12f22331d297d2a9d1e2f18dc257 AS prebuilder
FROM debian:buster-slim AS prebuilder

ARG LIBNICE_VERSION="0.1.17"
ARG LIBSRTP_VERSION="v2.3.0"
ARG LIBWEBSOCKETS_VERSION="v3.2.2"

ARG DEBIAN_FRONTEND=noninteractive

# Prebuilder is just for sake of caching and legibility. It is used to build Janus' dependancies Install Build Dependencies
RUN rm -rf /var/lib/apt/lists/* \
  && apt-get -y update \
  && apt-get install -yq apt-utils \
  && apt-get install -yq \
  libssl-dev \
  pkg-config \
  gengetopt \
  libtool \
  automake \
  autotools-dev \
  build-essential \
  git \
  cmake \
  autoconf \
  gtk-doc-tools \
  libgnutls28-dev \
  libglib2.0-dev \
#Install libnice (ICE candidates)
  && cd / \
  && git clone https://gitlab.freedesktop.org/libnice/libnice \
  && cd libnice \
  && git checkout ${LIBNICE_VERSION} \
  && ./autogen.sh \
  && ./configure --prefix=/opt/deps \
  && make \
  && make install \
# Install libsrtp (Secure Real-time Transport Protocol)  
  && cd / \
  && git clone https://github.com/cisco/libsrtp.git \
  && cd libsrtp \
  && git checkout ${LIBSRTP_VERSION} \
  && ./configure --prefix=/opt/deps --enable-openssl \
  && make shared_library \
  && make install \
# Install libwebsockets (Websockets)
  && cd / \
  && git clone https://github.com/warmcat/libwebsockets.git \
  && cd libwebsockets \
  && git checkout ${LIBWEBSOCKETS_VERSION} \
  && mkdir build \
  && cd build \
  && cmake -DCMAKE_INSTALL_PREFIX:PATH=/opt/deps -DCMAKE_C_FLAGS="-fpic" -DLWS_MAX_SMP=1 -DLWS_IPV6="ON" .. \
  && make \
  && make install \

#FROM debian:buster-slim AS prebuilder
FROM debian@sha256:1b8a66f829a3563f98f941f4e0e2ecc619ad12f22331d297d2a9d1e2f18dc257
LABEL maintainer="Goran Jovanov <goran.jovanov@gmail.com>"
LABEL description="Janus Gateway Lean Docker Image"

# COPY VERSION .

ARG JANUS_VERSION="v0.10.4"
WORKDIR /opt/

COPY --from=prebuilder /opt .

ARG DEBIAN_FRONTEND=noninteractive

# Install Build Dependencies
RUN rm -rf /var/lib/apt/lists/* \
  && apt-get -y update \
  && apt-get install -yq apt-utils \
  && apt-get install -yq \
  libjansson-dev \
  libmicrohttpd-dev \
  libssl-dev \
  libsofia-sip-ua-dev \
  libglib2.0-dev \
  libopus-dev \
  libogg-dev \
  libini-config-dev \
  libcollection-dev \
  libconfig-dev \
  libcurl3-gnutls \
  pkg-config \
  gengetopt \
  libtool \
  automake \
  autotools-dev \
  build-essential \
  git \
  cmake \
  autoconf \
  zlib1g-dev \
  libcurl4-openssl-dev \
# Install Janus Gateway
  && cd /tmp \
  && git clone https://github.com/meetecho/janus-gateway.git \
  && cd janus-gateway \
  && git checkout ${JANUS_VERSION} \
  && sh autogen.sh \
  && export PKG_CONFIG_PATH=/opt/deps/lib/pkgconfig \
  && ./configure \
  --prefix=/opt/janus \
  --disable-boringssl \
  --disable-rabbitmq \
  --disable-mqtt \
  --disable-data-channels \
  --enable-rest \
  --enable-websockets \  
  LDFLAGS="-L/opt/deps/lib -Wl,-rpath=/opt/deps/lib" CFLAGS="-I/opt/deps/include" \
  && make \
  && make install \
  && make configs \
  && ldconfig \
# Cleanup
  && cd / \
  && rm -Rf /tmp/* \
  && apt-get purge -y \
  build-essential \
  git \
  cmake \
  automake \
  pkg-config \
  gengetopt \
  libtool \
  autotools-dev \
  autoconf \
  && apt-get autoremove -y \
  && apt-get install -y libconfig9 libglib2.0-0 \
  && apt-get clean \
  && apt-get autoremove --purge shared-mime-info -y \
  && rm -rf /var/lib/apt/*


CMD /opt/janus/bin/janus
