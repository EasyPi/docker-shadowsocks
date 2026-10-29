#
# Dockerfile for shadowsocks-libev
#

FROM alpine:3.24
MAINTAINER EasyPi Software Foundation

ARG TARGETPLATFORM
ARG SS_LIBEV_VERSION
ARG V2RAY_PLUGIN_VERSION

RUN set -ex \
    && SS_URL=https://github.com/shadowsocks/shadowsocks-c/archive/v${SS_LIBEV_VERSION}.tar.gz \
    && SS_DIR=shadowsocks-c-${SS_LIBEV_VERSION} \
    && apk add --no-cache c-ares \
                          libcrypto3 \
                          libev \
                          libsodium \
                          mbedtls3 \
                          pcre2 \
    && apk add --no-cache \
               --virtual TMP build-base \
                             c-ares-dev \
                             cmake \
                             coreutils \
                             curl \
                             gettext-dev \
                             libev-dev \
                             libsodium-dev \
                             libtool \
                             linux-headers \
                             mbedtls3-dev \
                             openssl-dev \
                             pcre2-dev \
                             pkgconfig \
                             tar \
    && curl -sSL $SS_URL | tar xz \
    && echo "=== shadowsocks-libev ===" \
        && cd $SS_DIR \
        && curl -sSL https://github.com/shadowsocks/ipset/archive/shadowsocks.tar.gz | tar xz --strip 1 -C libipset \
        && curl -sSL https://github.com/shadowsocks/libcork/archive/shadowsocks.tar.gz | tar xz --strip 1 -C libcork \
        && curl -sSL https://github.com/shadowsocks/libbloom/archive/master.tar.gz | tar xz --strip 1 -C libbloom \
        && mkdir -p build && cd build \
        && cmake .. -DCMAKE_BUILD_TYPE=Release \
                    -DPCRE2_CONFIG=/usr/bin/pcre2-config \
                    -DLIBMBEDTLS=/usr/lib/libmbedtls.so \
                    -DLIBMBEDCRYPTO=/usr/lib/libmbedcrypto.so \
                    -DLIBPCRE2=/usr/lib/libpcre2-8.so \
                    -DLIBSODIUM=/usr/lib/libsodium.so \
        && jobs="$(nproc 2>/dev/null || sysctl -n hw.ncpu 2>/dev/null || echo 4)" \
        && make -j"$jobs" \
        && make install \
        && cd ../.. \
        && rm -rf $SS_DIR \
    && echo "=== v2ray-plugin ===" \
        && V2RAY_PLUGIN_ARCH=$(echo $TARGETPLATFORM | sed -e 's@/@-@') \
        && V2RAY_PLUGIN_FILE=v2ray-plugin-${V2RAY_PLUGIN_ARCH}-v${V2RAY_PLUGIN_VERSION}.tar.gz \
        && V2RAY_PLUGIN_URL=https://github.com/shadowsocks/v2ray-plugin/releases/download/v${V2RAY_PLUGIN_VERSION}/${V2RAY_PLUGIN_FILE} \
        && curl -sSL ${V2RAY_PLUGIN_URL} | tar xvz --wildcards 'v2ray-plugin_*' \
        && mv v2ray-plugin_* /usr/bin/v2ray-plugin \
        && v2ray-plugin -version \
    && apk del TMP

ENV SERVER_ADDR=0.0.0.0
ENV SERVER_PORT=8388
ENV METHOD=chacha20-ietf-poly1305
ENV PASSWORD=
ENV TIMEOUT=60
ENV DNS_ADDR=8.8.8.8

EXPOSE $SERVER_PORT/tcp
EXPOSE $SERVER_PORT/udp

CMD ss-server -s "$SERVER_ADDR" \
              -p "$SERVER_PORT" \
              -m "$METHOD"      \
              -k "$PASSWORD"    \
              -t "$TIMEOUT"     \
              -d "$DNS_ADDR"    \
              -u                \
              --fast-open $OPTIONS
