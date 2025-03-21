### Build stage for the website frontend
FROM node:10 AS website
RUN echo "deb http://archive.debian.org/debian stretch main" > /etc/apt/sources.list && \
    apt-get update -o Acquire::Check-Valid-Until=false
RUN apt-get install -y protobuf-compiler libprotobuf-dev
WORKDIR /code
COPY ./website/package.json ./
COPY ./website/package-lock.json ./
RUN npm ci --no-audit --prefer-offline
COPY ./proto/ ../proto/
COPY ./website/ ./
RUN npm run codegen
RUN npm run build

### Build stage for the website backend server
FROM golang:1.13.8-alpine AS server
RUN apk add gcc musl-dev
RUN apk add protobuf
RUN apk add protobuf-dev
WORKDIR /code/wg-access-server
ENV GOOS=linux
ENV GARCH=amd64
ENV CGO_ENABLED=1
ENV GO111MODULE=on
RUN go get github.com/golang/protobuf/protoc-gen-go@v1.3.5
COPY ./go.mod ./
COPY ./go.sum ./
RUN go mod download
COPY ./proto/ ./proto/
COPY ./codegen.sh ./
RUN ./codegen.sh
COPY ./main.go ./main.go
COPY ./cmd/ ./cmd/
COPY ./pkg/ ./pkg/
COPY ./internal/ ./internal/
RUN go build -o wg-access-server

### Server
FROM alpine:latest
RUN apk add iptables
RUN apk add wireguard-tools
RUN apk add curl
ENV WG_CONFIG="/config.yaml"
ENV WG_STORAGE="sqlite3:///data/db.sqlite3"
COPY --from=server /code/wg-access-server /usr/local/bin/wg-access-server
COPY --from=website /code/build /website/build
RUN chmod +x /usr/local/bin/wg-access-server
CMD ["/usr/local/bin/wg-access-server/wg-access-server", "serve"]
