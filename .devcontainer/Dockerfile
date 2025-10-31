FROM debian:bookworm-slim

ENV HUGO_VERSION=0.145.0

RUN apt-get update && apt-get install -y curl git

RUN curl -L "https://github.com/gohugoio/hugo/releases/download/v0.145.0/hugo_extended_0.145.0_linux-arm64.deb" -o "hugo.deb" && \
    apt-get install -y ./hugo.deb && \
    rm hugo.deb