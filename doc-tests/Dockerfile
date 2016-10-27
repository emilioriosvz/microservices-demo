FROM alpine:3.4
VOLUME /to-test
WORKDIR /to-test

RUN echo "http://dl-cdn.alpinelinux.org/alpine/edge/community" >> /etc/apk/repositories
RUN apk update && apk add --no-cache ruby curl docker py-pip jq ncurses
RUN pip install docker-compose
COPY ./runner/test-documentation /test-documentation
CMD /test-documentation
