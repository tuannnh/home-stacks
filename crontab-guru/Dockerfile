# syntax=docker/dockerfile:1.4
FROM alpine:latest

# Install dependencies
RUN apk add --no-cache \
    curl \
    bash \
    ca-certificates \
    dcron

# Install cronitor CLI
RUN curl -sL https://cronitor.io/dl/linux_amd64.tar.gz -o /usr/local/bin/cronitor && \
    chmod +x /usr/local/bin/cronitor

# Configure authentication using BuildKit secret (does NOT persist secret in image history)
# The secret file will be mounted at /run/secrets/cronitor_creds during build.
# The secret content should be "username:password".
RUN --mount=type=secret,id=cronitor_creds \
    set -e; \
    CREDS_FILE=/run/secrets/cronitor_creds; \
    if [ -f "$CREDS_FILE" ]; then \
      CRONITOR_USERNAME=$(cut -d: -f1 "$CREDS_FILE"); \
      CRONITOR_PASSWORD=$(cut -d: -f2- "$CREDS_FILE"); \
      cronitor configure --dash-username "$CRONITOR_USERNAME" --dash-password "$CRONITOR_PASSWORD"; \
    else \
      echo "No cronitor secret provided; skipping configure"; \
    fi

EXPOSE 9000

CMD ["cronitor", "dash", "--port", "9000"]