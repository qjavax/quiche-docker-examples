# 
# curl + quiche build image
#
FROM rust:latest AS builder

# Dependencies
RUN apt-get update && apt-get install -y cmake golang curl automake autoconf libtool pkg-config

# Build curl + quiche
# https://github.com/curl/curl/blob/master/docs/HTTP3.md
WORKDIR /build
RUN git clone --recursive https://github.com/cloudflare/quiche.git

# Build quiche
RUN cd quiche && \
    cargo build --release --features ffi,pkg-config-meta,qlog && \
    mkdir deps/boringssl/src/lib && \
    ln -vnf $(find target/release -name libcrypto.a -o -name libssl.a) deps/boringssl/src/lib/

# Build curl
RUN git clone https://github.com/curl/curl.git && \
    cd curl && \
    ./buildconf && \
    ./configure LDFLAGS="-Wl,-rpath,$PWD/../quiche/target/release" \
        --prefix=/usr/local \
        --with-ssl=$PWD/../quiche/deps/boringssl/src \
        --with-quiche=$PWD/../quiche/target/release \
        --disable-ldap \
        --disable-ldaps \
        && \
    make -j`nproc` && \
    make install

# Setup binary image
FROM debian:latest

RUN apt-get update && apt-get install -y ca-certificates

COPY --from=builder /usr/local /usr/local
COPY --from=builder /build/quiche/target/release/libquiche.a /usr/local/lib/
COPY --from=builder /build/quiche/target/release/libquiche.so /usr/local/lib/

RUN rm /etc/ld.so.cache && ldconfig
