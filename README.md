# kicad-mirror

A GitHub mirror of [`kicad/code/kicad`](https://gitlab.com/kicad/code/kicad)
on GitLab, with a Dockerfile and daily sync workflow.

## Branches

- **`main`** — default branch. Holds:
  - `docker/Dockerfile` for local KiCad dev container builds
  - `.github/workflows/sync-upstream.yml` (daily upstream sync)
  - `.github/workflows/initial-mirror.yml` (one-shot bootstrap)
  - This README
- **`master`** — pure mirror of upstream KiCad master. Force-synced
  daily by the sync workflow. **Never modify directly.**

## How sync works

`.github/workflows/sync-upstream.yml` runs daily at 06:00 UTC:

```bash
git fetch upstream master
git reset --hard upstream/master
git push origin master --force-with-lease
```

To trigger manually: GitHub UI → Actions → "Sync upstream KiCad master"
→ Run workflow.

## Bootstrap (first time only)

`.github/workflows/initial-mirror.yml` runs the initial 1.1 GB transfer
on GitHub's runners (server-side), avoiding any local upload. Trigger
once via the Actions UI; disable afterward.

## Local dev workflow

```bash
git clone https://github.com/El-Guapo2024/kicad-mirror.git
cd kicad-mirror

# Get fresh upstream code:
git fetch origin
git checkout master  # or rebase your work onto it

# Build the dev container:
docker build -t kicad-dev -f docker/Dockerfile .

# Run KiCad inside the container (X11 forwarding required for GUI):
docker run --rm -it \
    -e DISPLAY=$DISPLAY \
    -v /tmp/.X11-unix:/tmp/.X11-unix \
    -v /tmp/kicad:/tmp/kicad \
    -v $(pwd):/workspace \
    kicad-dev:latest
```

## Why a mirror

- Track upstream KiCad master without depending on third-party nightly
  build cadence
- Have a known-good base for building KiCad from source with
  `KICAD_IPC_API=ON`
- Run KiCad in a Docker container so the host stays clean
- Provide a stable URL for any project that needs a fresh KiCad
  source tree

## Dockerfile

The default `docker/Dockerfile` builds an Ubuntu 24.04 image with:
- KiCad's full C++ build dependencies (CMake, Ninja, wxWidgets, NNG,
  Boost, OpenCascade, ngspice, etc.)
- Protocol Buffers compiler matched to KiCad's runtime version
- `kicad-python` from `gitlab.com/kicad/code/kicad-python` (main branch
  has the `Schematic` module that PyPI 0.6.0 lacks)
- The KiCad source build step is commented out by default. Uncomment
  the `RUN cmake .. && ninja install` block when you need an installed
  binary inside the container.

## Cleanup

```bash
gh repo delete El-Guapo2024/kicad-mirror --yes
```
