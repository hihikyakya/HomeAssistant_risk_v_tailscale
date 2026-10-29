FROM ubuntu:24.04


RUN apt-get update && \
    apt-get install -y \
        curl \
        ca-certificates \
        && \
    curl -fsSL https://tailscale.com/install.sh | sh && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*


COPY start.sh /start.sh
RUN chmod +x /start.sh


RUN useradd -m -s /bin/bash appuser

WORKDIR /app

COPY --chown=appuser:appuser . /app



CMD ["/start.sh"]
