# syntax=docker/dockerfile:1.4
FROM alpine:latest

# Allow buildkit to pass TARGETARCH automatically (buildx sets this)
ARG TARGETARCH

# Install dependencies
RUN apk add --no-cache \
    docker-cli \ 
    curl \
    bash \
    ca-certificates \
    dcron \
    tar

# Download and extract the cronitor binary for the build architecture.
# We choose the archive name based on TARGETARCH (amd64 / arm64). If the archive layout has a top-level folder,
# we attempt to strip the component when extracting.
RUN set -eux; \
    case "${TARGETARCH:-amd64}" in \
      amd64) file="linux_amd64.tar.gz" ;; \
      arm64) file="linux_arm64.tar.gz" ;; \
      *) file="linux_amd64.tar.gz" ;; \
    esac; \
    url="https://cronitor.io/dl/${file}"; \
    echo "Downloading ${url}"; \
    curl -fsSL "$url" -o /tmp/cronitor.tar.gz; \
    # Try extracting with strip-components=1 (most archives have a top dir); if that fails try without it
    if tar -tzf /tmp/cronitor.tar.gz | sed -n '1p' | grep -q '/'; then \
      tar -xzf /tmp/cronitor.tar.gz -C /usr/local/bin --strip-components=1; \
    else \
      tar -xzf /tmp/cronitor.tar.gz -C /usr/local/bin; \
    fi; \
    chmod +x /usr/local/bin/cronitor; \
    rm -f /tmp/cronitor.tar.gz



EXPOSE 9000

CMD ["/usr/local/bin/cronitor", "dash", "--port", "9000"]