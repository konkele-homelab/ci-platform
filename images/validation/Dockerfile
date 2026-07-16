FROM alpine:3.23

LABEL org.opencontainers.image.title="CI Validation Image" \
      org.opencontainers.image.description="Container validation runtime with crane tooling"

ARG CRANE_VERSION=v0.20.6

RUN apk add --no-cache \
        bash \
        ca-certificates \
        curl \
        tar && \
    curl -fsSL \
      "https://github.com/google/go-containerregistry/releases/download/${CRANE_VERSION}/go-containerregistry_Linux_x86_64.tar.gz" \
      | tar -xz -C /usr/local/bin crane && \
    crane version

WORKDIR /workspace

ENTRYPOINT ["/bin/bash"]