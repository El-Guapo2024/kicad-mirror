# KiCad development image — runnable KiCad + kicad-python.
#
# Provides:
#   - Ubuntu 24.04 with KiCad nightly installed from the official
#     KiCad PPA (KICAD_IPC_API=ON enabled in upstream nightly builds)
#   - Full C++ build deps so this image is also usable for building
#     KiCad from source when working on C++ patches (A.6.3)
#   - kicad-python from gitlab main (has the Schematic module
#     PyPI 0.6.0 lacks; pinned via submodule for reproducibility)
#
# Build:
#   docker build -t kicad-dev:latest -f docker/Dockerfile .
#
# Run with X11 forwarding (mac via XQuartz, Linux native):
#   docker run --rm -it \
#     -e DISPLAY=host.docker.internal:0 \
#     -v /tmp/.X11-unix:/tmp/.X11-unix \
#     -v /tmp/kicad:/tmp/kicad \
#     -v $(pwd):/workspace \
#     kicad-dev:latest
#
# Inside the container:
#   kicad-nightly &                    # launch KiCad GUI (X11-forwarded)
#   python3 -c 'import kipy; print(kipy.KiCad().get_version())'

FROM ubuntu:24.04 AS base

ARG DEBIAN_FRONTEND=noninteractive

RUN apt-get update && apt-get install -y --no-install-recommends \
        ca-certificates \
        curl \
        git \
        gnupg \
        protobuf-compiler libprotobuf-dev \
        python3 python3-pip python3-venv \
        software-properties-common \
        wget \
    && rm -rf /var/lib/apt/lists/*

# ─── KiCad nightly from official PPA (KICAD_IPC_API=ON by default) ──
# The kicad-nightly metapackage pulls all of KiCad's runtime
# dependencies (wxWidgets, ngspice, OCC, etc.) automatically.
RUN add-apt-repository -y ppa:kicad/kicad-dev-nightly && \
    apt-get update && \
    apt-get install -y --no-install-recommends kicad-nightly && \
    rm -rf /var/lib/apt/lists/*

# ─── (Optional) KiCad source build deps + build step ────────────────
# Uncomment the block below when you need to build KiCad from this
# mirror's master branch (e.g. to test our own C++ patches before
# opening MRs upstream — Track A.6.3). Adds ~30-60 min build time
# and ~5 GB disk for build artifacts.
#
# RUN apt-get update && apt-get install -y --no-install-recommends \
#         build-essential cmake ninja-build pkg-config swig \
#         libboost-all-dev libcairo2-dev libcurl4-openssl-dev \
#         libglew-dev libglm-dev libgtk-3-dev \
#         libngspice0-dev libnng-dev \
#         libocct-foundation-dev libocct-data-exchange-dev \
#         libpython3-dev libsecret-1-dev libssl-dev libudev-dev \
#         libwebkit2gtk-4.1-dev libwxgtk3.2-dev libwxgtk-webview3.2-dev \
#         unittest++ libunittest++-dev \
#         zlib1g-dev \
#     && rm -rf /var/lib/apt/lists/*
#
# COPY . /src/kicad
# WORKDIR /src/kicad
# RUN mkdir build && cd build && \
#     cmake .. -GNinja \
#         -DCMAKE_BUILD_TYPE=Release \
#         -DKICAD_IPC_API=ON \
#         -DKICAD_USE_NNG=ON \
#         -DCMAKE_INSTALL_PREFIX=/opt/kicad-fork && \
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
