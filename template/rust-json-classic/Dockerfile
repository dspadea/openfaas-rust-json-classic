FROM openfaas/classic-watchdog:0.20.1 as watchdog

# Build Stage
FROM rust:1.57.0-alpine AS builder

RUN rustup target add x86_64-unknown-linux-musl

RUN apk add musl-dev

#RUN USER=root
WORKDIR /usr/src/openfaas
COPY function ./function
COPY main ./main

RUN cd "function" && cargo build --release

RUN cd main && cargo install --target x86_64-unknown-linux-musl --path .

# Runner stage
FROM alpine:3.15.0 as runner 

RUN apk --no-cache add curl ca-certificates \
    && addgroup -S app && adduser -S -g app app

COPY --from=watchdog /fwatchdog /usr/bin/fwatchdog
RUN chmod +x /usr/bin/fwatchdog

COPY --from=builder /usr/local/cargo/bin/main /usr/bin/main

HEALTHCHECK --interval=5s CMD [ -e /tmp/.lock ] || exit 1

ENV fprocess="main"

RUN ls -la /

CMD ["fwatchdog"]

