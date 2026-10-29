#
# Dockerfile for shadowsocks-rust
#

FROM alpine:3.24
MAINTAINER EasyPi Software Foundation

ARG TARGETPLATFORM
ARG SS_RUST_VERSION
ARG V2RAY_PLUGIN_VERSION

RUN set -ex \
    && apk add --no-cache curl tar \
    && echo "=== shadowsocks-rust ===" \
    && SS_RUST_ARCH=$(echo $TARGETPLATFORM | sed -e 's@linux/amd64@x86_64-unknown-linux-musl@' \
                                                 -e 's@linux/arm64@aarch64-unknown-linux-musl@') \
    && SS_RUST_FILE=shadowsocks-v${SS_RUST_VERSION}.${SS_RUST_ARCH}.tar.xz \
    && SS_RUST_URL=https://github.com/shadowsocks/shadowsocks-rust/releases/download/v${SS_RUST_VERSION}/${SS_RUST_FILE} \
    && curl -sSL ${SS_RUST_URL} | tar xvJ ssserver -C /usr/bin/ \
    && ssserver --version \
    && echo "=== v2ray-plugin ===" \
    && V2RAY_PLUGIN_ARCH=$(echo $TARGETPLATFORM | sed -e 's@/@-@') \
    && V2RAY_PLUGIN_FILE=v2ray-plugin-${V2RAY_PLUGIN_ARCH}-v${V2RAY_PLUGIN_VERSION}.tar.gz \
    && V2RAY_PLUGIN_URL=https://github.com/shadowsocks/v2ray-plugin/releases/download/v${V2RAY_PLUGIN_VERSION}/${V2RAY_PLUGIN_FILE} \
    && curl -sSL ${V2RAY_PLUGIN_URL} | tar xvz --wildcards 'v2ray-plugin_*' \
    && mv v2ray-plugin_* /usr/bin/v2ray-plugin \
    && v2ray-plugin -version \
    && apk del curl tar

ENV SERVER_ADDR=0.0.0.0
ENV SERVER_PORT=8388
ENV METHOD=chacha20-ietf-poly1305
ENV PASSWORD=
ENV DNS=8.8.8.8,8.8.4.4
ENV TIMEOUT=300
ENV WORKERS=4

EXPOSE $SERVER_PORT/tcp
EXPOSE $SERVER_PORT/udp

CMD ssservice server -s "$SERVER_ADDR:$SERVER_PORT" \
                     -k "${PASSWORD:-$(hostname)}"  \
                     -m "$METHOD"                   \
                     --dns "$DNS"                   \
                     --timeout "$TIMEOUT"           \
                     --worker-threads "$WORKERS"    \
                     --tcp-fast-open --tcp-no-delay \
                     -U
