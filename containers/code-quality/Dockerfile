FROM python:3.12-alpine

# Build arg for config path
ARG PRECOMMIT_CONFIG_PATH=".pre-commit-config.yaml"

# Install all tools + Rust for pre-commit hooks
RUN apk add --no-cache --upgrade \
    git \
    curl \
    shellcheck \
    nodejs \
    npm \
    wget \
    bash \
    sqlite \
    rust \
    cargo && \
    wget --progress=dot:giga -O /usr/local/bin/hadolint \
    https://github.com/hadolint/hadolint/releases/download/v2.12.0/hadolint-Linux-x86_64 && \
    chmod +x /usr/local/bin/hadolint && \
    pip install --no-cache-dir --upgrade pip setuptools && \
    pip install --no-cache-dir \
        pre-commit \
        ruff \
        black \
        bandit && \
    npm install -g \
        prettier \
        @biomejs/biome

# Set git config
RUN git config --global --add safe.directory '*' && \
    git config --global user.email "ci@localhost" && \
    git config --global user.name "CI Bot"

# Set PRE_COMMIT_HOME to use global cache
ENV PRE_COMMIT_HOME=/root/.cache/pre-commit

# Create a temp directory to install hooks globally
WORKDIR /tmp
COPY ${PRECOMMIT_CONFIG_PATH} .pre-commit-config.yaml
RUN git init && \
    pre-commit install-hooks

# Copy config to root for easy access
RUN cp .pre-commit-config.yaml /.pre-commit-config.yaml

# Set final working directory
WORKDIR /workspace
