# syntax=docker/dockerfile:1

# Builds wcpan.drive.server from the wcpan.drive workspace, which compose passes
# as the named build context "source" (a git URL or a local checkout).

FROM python:3.14-slim-trixie AS build

COPY --from=ghcr.io/astral-sh/uv:0.13.7 /uv /usr/local/bin/
ENV UV_COMPILE_BYTECODE=1 \
    UV_LINK_MODE=copy \
    UV_PYTHON_DOWNLOADS=never \
    UV_NO_BINARY_PACKAGE=pymediainfo \
    UV_PROJECT_ENVIRONMENT=/opt/venv

WORKDIR /src
# Only what the build needs; a local checkout's .venv stays out.
COPY --from=source pyproject.toml uv.lock ./
COPY --from=source packages ./packages

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
