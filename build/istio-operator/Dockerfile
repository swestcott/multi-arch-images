ARG VERSION=1.6.5

FROM golang:1.14 as build

ARG VERSION=1.6.5
ARG TARGETPLATFORM

ENV GO111MODULE=on \
  CGO_ENABLED=0

WORKDIR /go/src/github.com/istio/istio

RUN git clone --depth 1 -b ${VERSION} https://github.com/istio/istio.git .

RUN export GOOS=$(echo ${TARGETPLATFORM} | cut -d / -f1) && \
  export GOARCH=$(echo ${TARGETPLATFORM} | cut -d / -f2) && \
  GOARM=$(echo ${TARGETPLATFORM} | cut -d / -f3 | cut -c2-) && \
  GO111MODULE=on go get github.com/jteeuwen/go-bindata/go-bindata@6025e8de665b && \
  ./operator/scripts/create_assets_gen.sh && \
  STATIC=0 LDFLAGS='-extldflags -static -s -w' ./common/scripts/gobuild.sh /go/src/github.com/istio/istio/out/operator ./operator/cmd/operator

FROM istio/operator:${VERSION} as operator

FROM raspbernetes/istio-base:e0817929c81150b95947ed75690021a6c703c4d8

COPY --from=build /go/src/github.com/istio/istio/out/operator /usr/local/bin/operator
COPY --from=operator /var/lib/istio/manifests/ /var/lib/istio/manifests/

ENTRYPOINT ["/usr/local/bin/operator"]

