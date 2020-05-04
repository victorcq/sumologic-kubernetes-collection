FROM golang:1.14.1 as go-builder
RUN mkdir -p /build/cmd
ADD cmd /build/cmd
ADD go.sum go.mod /build/
WORKDIR /build
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -ldflags '-extldflags "-static"' -o k8s-api-test cmd/k8s-api-test/main.go

FROM rust:alpine3.11 as rust-builder
COPY receiver-mock /build
WORKDIR /build
RUN apk update && apk upgrade && apk add g++
RUN RUSTFLAGS="-C target-feature=-crt-static" cargo build --release
RUN find / -name libgcc_s.so.1

FROM alpine:3.9

RUN set -ex \
    && echo "http://dl-cdn.alpinelinux.org/alpine/edge/main" >> /etc/apk/repositories \
    && echo "http://dl-cdn.alpinelinux.org/alpine/edge/testing" >> /etc/apk/repositories \
    && apk update \
    && apk upgrade \
    && apk add --no-cache \
    bash \
    busybox-extras \
    curl \
    libc6-compat \
    openssl \
    net-tools \
    vim

ADD motd /etc/motd
ADD profile  /etc/profile

ADD check /usr/bin/check
ADD tools-usage /usr/bin/tools-usage
COPY --from=go-builder /build/k8s-api-test /usr/bin/
COPY --from=rust-builder /build/target/release/sumock /usr/bin
COPY --from=rust-builder /usr/lib/libgcc_s.so.1 /usr/lib

CMD ["/usr/bin/tools-usage"]
