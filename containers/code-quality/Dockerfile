FROM ubuntu:24.04

# Install system dependencies
RUN apt-get update && apt-get install -y \
    git \
    curl \
    wget \
    python3 \
    python3-pip \
    python3-venv \
    nodejs \
    npm \
    shellcheck \
    build-essential \
    pkg-config \
    libssl-dev \
    && rm -rf /var/lib/apt/lists/*

# Install Rust
RUN curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y
ENV PATH="/root/.cargo/bin:$PATH"

# Install hadolint
RUN wget -O /usr/local/bin/hadolint \
    https://github.com/hadolint/hadolint/releases/download/v2.12.0/hadolint-Linux-x86_64 && \
    chmod +x /usr/local/bin/hadolint

# Install Python tools
RUN pip3 install --break-system-packages \
    pre-commit \
    ruff \
    black \
    bandit

# Install JS tools
RUN npm install -g \
    prettier \
    @biomejs/biome

# Set git config
RUN git config --global --add safe.directory '*' && \
    git config --global user.email "ci@localhost" && \
    git config --global user.name "CI Bot"

# Pre-install hooks
COPY .pre-commit-config.yaml /.pre-commit-config.yaml
WORKDIR /tmp
RUN cp /.pre-commit-config.yaml . && \
    git init && \
    pre-commit install-hooks

WORKDIR /workspace
