FROM linuxkit/alpine:35b33c6b03c40e51046c3b053dd131a68a26c37a AS mirror

RUN apk add --no-cache go gcc musl-dev linux-headers
ARG GOPKGVERSION
ENV ldflags="-X main.Version=$GOPKGVERSION"
ENV GOPATH=/go PATH=$PATH:/go/bin
# Hack to work around an issue with go on arm64 requiring gcc
RUN [ $(uname -m) = aarch64 ] && apk add --no-cache gcc || true

# see https://github.com/golang/go/issues/23672
ENV CGO_CFLAGS_ALLOW=(-mrdrnd|-mrdseed)

COPY . /go/src/rngd/
RUN REQUIRE_CGO=1 go-compile.sh /go/src/rngd/cmd/rngd

FROM scratch
ENTRYPOINT []
CMD []
WORKDIR /
COPY --from=mirror /go/bin/rngd /sbin/rngd
CMD ["/sbin/rngd"]
