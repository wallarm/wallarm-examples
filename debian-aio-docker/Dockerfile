
# Use the official Debian 12 image from the Docker Hub
FROM debian:12

# WALLARM_AIO_VERSION: version of AIO package to download (e.g. 6.1.0)
ARG WALLARM_AIO_VERSION
ARG NGINX_VERSION

ARG WALLARM_WCLI_MODE=filtering

LABEL org.opencontainers.image.authors=mail@maintainer.com

# Set environment variables to non-interactive to avoid prompts
ENV DEBIAN_FRONTEND=noninteractive
ENV WALLARM_WCLI_MODE=$WALLARM_WCLI_MODE

# Install required packages for adding repositories and fetching keys, install nginx from repo
RUN groupadd -r wallarm && \
    useradd -r -d /opt/wallarm -g wallarm wallarm && \
    apt-get update && apt-get install -y gnupg2 ca-certificates lsb-release wget --no-install-recommends && \
    wget https://nginx.org/keys/nginx_signing.key && \
    apt-key add nginx_signing.key && \
    printf "deb http://nginx.org/packages/debian/ $(lsb_release -cs) nginx\n" | tee /etc/apt/sources.list.d/nginx.list && \
    apt-get update && apt-get install -y libcap2-bin nginx=$NGINX_VERSION* --no-install-recommends && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

# Install AiO in specified mode, node registration skipped with the --install-only argument, clean up, fix chown
RUN wget -O ./aio.sh "https://meganode.wallarm.com/${WALLARM_AIO_VERSION%.*}/wallarm-${WALLARM_AIO_VERSION}.x86_64-glibc.sh" && \
    chmod +x aio.sh && \
    ./aio.sh -- -b --install-only --docker ${WALLARM_WCLI_MODE} && \
    chown -R wallarm:wallarm /run /etc/environment /etc/nginx /var/log/nginx /var/cache/nginx /usr/share/nginx /opt/wallarm && \
    rm aio.sh

# Optional - redirect nginx logs to stdout, and setcap on binaries
RUN ln -sf /dev/stdout /var/log/nginx/access.log && \
    ln -sf /dev/stderr /var/log/nginx/error.log && \
    setcap    cap_net_bind_service=+ep /usr/sbin/nginx && \
    setcap -v cap_net_bind_service=+ep /usr/sbin/nginx

# Copy your entry point script
COPY entrypoint.sh /entrypoint.sh

# Copy your custom config(s) for nginx
COPY nginx.conf /etc/nginx/nginx.conf

# Expose port 80
EXPOSE 80

# Run as unprivileged user wallarm (used in AIO)
USER wallarm

# Use entrypoint
ENTRYPOINT ["/entrypoint.sh"]
