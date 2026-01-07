FROM golang:alpine AS builder

WORKDIR /src
COPY . .

RUN apk add --no-cache curl git make libcap
RUN make V=1 bin/step-ca
RUN setcap CAP_NET_BIND_SERVICE=+eip bin/step-ca

FROM smallstep/step-kms-plugin:cloud AS kms

FROM smallstep/step-cli:latest

COPY --from=builder /src/bin/step-ca /usr/local/bin/step-ca
COPY --from=kms /usr/local/bin/step-kms-plugin /usr/local/bin/step-kms-plugin

USER step

ENV CONFIGPATH="/home/step/config/ca.json"
ENV PWDPATH="/home/step/secrets/password"

VOLUME ["/home/step"]
STOPSIGNAL SIGTERM
HEALTHCHECK --interval=30s --timeout=10s --start-period=60s --retries=3 \
    CMD step ca health

COPY docker/entrypoint.sh /entrypoint.sh

ENTRYPOINT ["/bin/bash", "/entrypoint.sh"]
CMD exec /usr/local/bin/step-ca --password-file $PWDPATH $CONFIGPATH
