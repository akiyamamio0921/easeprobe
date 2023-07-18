FROM golang:1.20.4-alpine as builder
WORKDIR /go/src/github.com/megaease/easeprobe/
COPY ./ /go/src/github.com/megaease/easeprobe/
RUN --mount=type=cache,target=/var/cache/apk \
    apk --no-cache add make git gcc libc-dev

COPY go.mod go.mod
COPY go.sum go.sum
COPY . .

RUN --mount=type=cache,mode=0777,id=gomodcache,target=/go/pkg/mod \
    --mount=type=cache,target=/root/.cache/go-build \
    make

FROM alpine:latest

ENV PATH=/opt/:$PATH \
    PROBE_CONFIG=/opt/config.yaml

WORKDIR /opt/

COPY --from=builder /go/src/github.com/megaease/easeprobe/build/bin/* ./
COPY --from=builder /go/src/github.com/megaease/easeprobe/resources/scripts/entrypoint.sh /

RUN set -x \
    && apk update \
    && apk add  --no-cache tini tzdata busybox-extras curl redis upx \
    && upx ./easeprobe \
    && rm -rf /var/cache/apk/*

ENTRYPOINT [ "/sbin/tini", "--", "/entrypoint.sh" ]
