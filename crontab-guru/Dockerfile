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

# Configure authentication
RUN cronitor configure --dash-username admin --dash-password changeme

EXPOSE 9000

CMD ["cronitor", "dash", "--port", "9000"]