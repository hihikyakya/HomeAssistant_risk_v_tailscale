
FROM debian:testing-slim

ENV DEBIAN_FRONTEND=noninteractive \
    UV_LINK_MODE=copy \
    UV_CACHE_DIR=/tmp/uv-cache \
    PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1

RUN apt-get update && apt-get install -y --no-install-recommends \
    ca-certificates \
    curl \
    tzdata \
    git \
    build-essential \
    pkg-config \
    cmake \
    autoconf \
    automake \
    rustc \
    cargo \
    gfortran \
    python3.14 \
    python3.14-dev \
    python3.14-venv \
    libffi-dev \
    libssl-dev \
    libxml2-dev \
    libxslt1-dev \
    libjpeg-dev \
    zlib1g-dev \
    libopenjp2-7-dev \
    libtiff-dev \
    libopenblas-dev \
    liblapack-dev \
    libpcap-dev \
    libdbus-1-dev \
    libbluetooth-dev \
    libavformat-dev \
    libavcodec-dev \
    libavdevice-dev \
    libavutil-dev \
    libavfilter-dev \
    libswscale-dev \
    libswresample-dev \
    libturbojpeg0 \
    libturbojpeg0-dev \
    bluez \
    ffmpeg \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

RUN python3.14 -m venv /opt/homeassistant

RUN curl -LsSf https://astral.sh/uv/install.sh \
    | env UV_INSTALL_DIR=/usr/local/bin sh


# Configure uv
ENV UV_INDEX_STRATEGY=unsafe-best-match \
    UV_EXTRA_INDEX_URL=https://pypi.riseproject.dev/simple/

# Install Home Assistant and additional dependencies
RUN uv pip install \
        --python /opt/homeassistant/bin/python \
        homeassistant==2026.10.2 \
    && uv pip install \
        --python /opt/homeassistant/bin/python \
        numpy \
    && uv pip install \
        --python /opt/homeassistant/bin/python \
        hassil gazetteer-matcher \
        PyTurboJPEG \
        mutagen \
        pymicro-vad \
        pyspeex-noise \
    && uv pip install \
        --python /opt/homeassistant/bin/python \
        ha-ffmpeg \
    && rm -rf /tmp/uv-cache


ENV PATH="/opt/homeassistant/bin:${PATH}"

WORKDIR /config

EXPOSE 8123

CMD ["python", "-m", "homeassistant", "--config", "/config"]
