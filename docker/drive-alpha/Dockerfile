# Builds wcpan.drive.server from the wcpan.drive workspace at DRIVE_REF.
# Works with the classic builder; no BuildKit features are used.

FROM python:3.14-slim-trixie AS build

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    ca-certificates git && \
    rm -rf /var/lib/apt/lists/*

COPY --from=ghcr.io/astral-sh/uv:0.13.7 /uv /usr/local/bin/
ENV UV_COMPILE_BYTECODE=1 \
    UV_LINK_MODE=copy \
    UV_PYTHON_DOWNLOADS=never \
    UV_NO_BINARY_PACKAGE=pymediainfo \
    UV_PROJECT_ENVIRONMENT=/opt/venv

ARG DRIVE_REPO=https://github.com/legnaleurc/wcpan.drive.git
# A branch, tag or full commit hash.
ARG DRIVE_REF=master

WORKDIR /src
RUN git init --quiet . && \
    git fetch --quiet --depth 1 "$DRIVE_REPO" "$DRIVE_REF" && \
    git checkout --quiet FETCH_HEAD && \
    git log -1 --format='building %H %s'

# Locked server dependencies only: no dev tools, no client packages.
RUN uv sync --frozen --no-dev --no-editable \
    --package wcpan-drive-server-synology


FROM python:3.14-slim-trixie

RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    libmediainfo0v5 && \
    rm -rf /var/lib/apt/lists/*

COPY --from=build /opt/venv /opt/venv
ENV PATH=/opt/venv/bin:$PATH
