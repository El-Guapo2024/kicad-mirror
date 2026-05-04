# KiCad development image — built from this mirror's master branch.
#
# Provides:
#   - Ubuntu 24.04 with KiCad's full build deps
#   - kicad-python from gitlab main (has the Schematic module
#     PyPI 0.6.0 lacks; pinned via submodule for reproducibility)
#   - Optional KiCad-from-source build (uncomment block below) with
#     KICAD_IPC_API=ON
#
# Build:
#   docker build -t kicad-dev:latest -f docker/Dockerfile .
#
# Run with X11 forwarding (mac via XQuartz, Linux native):
#   docker run --rm -it \
#     -e DISPLAY=$DISPLAY \
#     -v /tmp/.X11-unix:/tmp/.X11-unix \
#     -v /tmp/kicad:/tmp/kicad \
#     -v $(pwd):/workspace \
#     kicad-dev:latest

FROM ubuntu:24.04 AS base

ARG DEBIAN_FRONTEND=noninteractive

RUN apt-get update && apt-get install -y --no-install-recommends \
        build-essential \
        cmake \
        ninja-build \
        git \
        pkg-config \
        software-properties-common \
        ca-certificates \
        curl \
        wget \
    && rm -rf /var/lib/apt/lists/*

# ─── KiCad runtime + build deps ─────────────────────────────────────
RUN apt-get update && apt-get install -y --no-install-recommends \
        libwxgtk3.2-dev libwxgtk-webview3.2-dev \
        libnng-dev \
        libssl-dev \
        libboost-all-dev \
        libcairo2-dev \
        libcurl4-openssl-dev \
        libglew-dev \
        libglm-dev \
        libgtk-3-dev \
        libngspice0-dev \
        libocct-foundation-dev libocct-data-exchange-dev \
        libpython3-dev \
        libsecret-1-dev \
        libudev-dev \
        libunit-test++-dev \
        libwebkit2gtk-4.1-dev \
        protobuf-compiler libprotobuf-dev \
        python3 python3-pip python3-venv \
        swig \
        zlib1g-dev \
    && rm -rf /var/lib/apt/lists/*

# ─── KiCad source build (commented; uncomment when needed) ──────────
# To build KiCad itself inside the image, uncomment the COPY/cmake
# block below and rebuild. Adds ~30-60 min to the image build.
#
# COPY . /src/kicad
# WORKDIR /src/kicad
# RUN mkdir build && cd build && \
#     cmake .. -GNinja \
#         -DCMAKE_BUILD_TYPE=Release \
#         -DKICAD_IPC_API=ON \
#         -DKICAD_USE_NNG=ON \
#         -DCMAKE_INSTALL_PREFIX=/opt/kicad && \
#     ninja && ninja install

# ─── Python deps for kicad-python ───────────────────────────────────
RUN python3 -m pip install --break-system-packages --no-cache-dir \
        'protobuf>=5.29,<6' \
        'pynng>=0.9,<0.10' \
        'mypy-protobuf==3.6.*' \
        'protoletariat==3.3.*'

# kicad-python from gitlab main (PyPI 0.6.0 lacks the Schematic module)
RUN git clone --depth 1 https://gitlab.com/kicad/code/kicad-python.git /opt/kicad-python && \
    cd /opt/kicad-python && \
    git submodule update --init --recursive --depth 1 && \
    python3 -m pip install --break-system-packages --no-cache-dir --no-deps .

# ─── Runtime defaults ───────────────────────────────────────────────
WORKDIR /workspace

# Override KICAD_API_SOCKET if running multiple instances side-by-side
ENV KICAD_API_SOCKET=ipc:///tmp/kicad/api.sock

CMD ["/bin/bash"]
