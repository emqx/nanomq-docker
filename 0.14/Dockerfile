FROM alpine:3.16 as builder

RUN apk add gcc cmake make musl-dev g++ curl

ENV VSN=0.14.8
RUN set -eu; \
    curl --silent --show-error -kfL -o nanomq.tar.gz "https://github.com/emqx/nanomq/archive/refs/tags/${VSN}.tar.gz"; \
    mkdir -p nanomq; \
    tar -zxf nanomq.tar.gz -C nanomq --strip-components 1;\
    rm -f nanomq.tar.gz

ENV NNGVSN=0.14.5
RUN set -eu; \
    curl --silent --show-error -kfL -o nanonng.tar.gz "https://github.com/nanomq/NanoNNG/archive/refs/tags/${NNGVSN}.tar.gz"; \
    mkdir -p nanomq/nng; \
    tar -zxf nanonng.tar.gz -C nanomq/nng --strip-components 1;\
    rm -f nanonng.tar.gz

WORKDIR /nanomq/build

RUN cmake .. && make

FROM alpine:3.16

COPY --from=builder /nanomq/build/nanomq/nanomq /usr/local/nanomq/
COPY --from=builder /nanomq/build/nanomq_cli/nanomq_cli /usr/local/nanomq/
COPY --from=builder /nanomq/etc/nanomq.conf /etc/nanomq.conf
COPY --from=builder /usr/lib/libgcc_s.so.1 /usr/lib/
COPY docker-entrypoint.sh /usr/bin/docker-entrypoint.sh

WORKDIR /usr/local/nanomq

RUN ln -s /usr/local/nanomq/nanomq /usr/bin/nanomq && \
    ln -s /usr/local/nanomq/nanomq_cli /usr/bin/nanomq_cli

EXPOSE 1883

ENTRYPOINT ["/usr/bin/docker-entrypoint.sh"]

CMD ["--url", "nmq-tcp://0.0.0.0:1883"]
