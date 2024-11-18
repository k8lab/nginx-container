FROM golang:1.22.5-alpine3.20

RUN apk update && \
    apk add git && \
    cd src && \
    git clone https://github.com/FiloSottile/mkcert.git && \
    cd mkcert && \
    go build -ldflags "-X main.Version=$(git describe --tags)"

FROM nginx:1.27.2-alpine3.20
ARG WSL_USER

COPY --from=0 /go/src/mkcert/mkcert /usr/local/bin

ARG TIME_ZONE=Asia/Tokyo

ENV TZ=${TIME_ZONE}

RUN apk update && apk add --no-cache --virtual .build-dependencies tzdata openssl \
    && cp /usr/share/zoneinfo/${TIME_ZONE} /etc/localtime \
    && apk del --no-network .build-dependencies \
    && adduser -D -s /bin/ash $WSL_USER \
    && mkdir -p /cert && chown $WSL_USER:$WSL_USER /cert

RUN sed -i 's/keepalive_timeout  65;/keepalive_timeout 180;/' /etc/nginx/nginx.conf \
    && sed -i '/keepalive_timeout 180;/a     send_timeout 180;\n    fastcgi_connect_timeout 180;\n    fastcgi_send_timeout 180;\n    fastcgi_read_timeout 180;' /etc/nginx/nginx.conf
