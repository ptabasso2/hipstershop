FROM golang:1.14-alpine AS mod
RUN apk add git

ENV PROJECT productcatalogservice
WORKDIR $GOPATH/$PROJECT

COPY go.* ./
RUN GO111MODULE=on go mod download

FROM golang:1.14-alpine AS build
COPY --from=mod $GOCACHE $GOCACHE
COPY --from=mod $GOPATH/pkg/mod $GOPATH/pkg/mod
ENV PROJECT productcatalogservice
WORKDIR $GOPATH/$PROJECT

COPY . .

RUN GO111MODULE=on CGO_ENABLED=0 GOOS=linux go build -o /productcatalogservice .

FROM alpine AS release
RUN apk add --no-cache ca-certificates
RUN GRPC_HEALTH_PROBE_VERSION=v0.2.0 && \
    wget -qO/bin/grpc_health_probe https://github.com/grpc-ecosystem/grpc-health-probe/releases/download/${GRPC_HEALTH_PROBE_VERSION}/grpc_health_probe-linux-amd64 && \
    chmod +x /bin/grpc_health_probe
WORKDIR /productcatalogservice
COPY --from=build /productcatalogservice ./server
COPY products.json .
EXPOSE 3550
ENTRYPOINT ["/productcatalogservice/server"]

