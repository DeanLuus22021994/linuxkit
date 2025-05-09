FROM linuxkit/alpine:35b33c6b03c40e51046c3b053dd131a68a26c37a as build

RUN apk add --no-cache bash go git musl-dev linux-headers make patch
# Hack to work around an issue with go on arm64 requiring gcc
RUN [ $(uname -m) = aarch64 ] && apk add --no-cache gcc || true

ENV GOPATH=/go PATH=$PATH:/go/bin
ENV GITBASE=github.com/google
ENV GITREPO=github.com/google/cadvisor
ENV COMMIT=v0.51.0

ADD /static.patch /tmp/

RUN mkdir -p /go/src/${GITBASE} \
    && cd /go/src/${GITBASE} \
    && git clone https://${GITREPO}.git \
    && cd /go/src/${GITREPO} \
    && git checkout ${COMMIT} \
    && patch -p1 build/build.sh </tmp/static.patch \
    && make build \
    && mv _output/cadvisor /usr/bin/


FROM linuxkit/alpine:35b33c6b03c40e51046c3b053dd131a68a26c37a AS mirror

RUN mkdir -p /out/etc/apk && cp -r /etc/apk/* /out/etc/apk/
RUN apk add --no-cache --initdb -p /out \
    alpine-baselayout \
    busybox \
    curl

# Remove apk residuals
RUN rm -rf /out/etc/apk /out/lib/apk /out/var/cache


FROM scratch
ENTRYPOINT []
CMD []
WORKDIR /
COPY --from=mirror /out/ /
COPY --from=build /usr/bin/cadvisor /usr/bin/cadvisor
COPY /waitfordocker.sh /usr/bin/waitfordocker.sh

ENTRYPOINT ["/usr/bin/waitfordocker.sh", "/usr/bin/cadvisor", "-logtostderr", "--disable_metrics=disk,tcp,udp"]
