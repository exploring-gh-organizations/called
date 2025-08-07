# Use multi-stage build for smaller final image
FROM python:3.12-alpine AS builder

# Install build dependencies in one layer
RUN apk add --no-cache --virtual .build-deps \
    git \
    curl \
    wget \
    bash \
    build-base \
    rust \
    cargo \
    nodejs \
    npm

# Install Python tools
RUN pip install --no-cache-dir --user \
    pre-commit \
    ruff \
    black \
    bandit

# Add user bin to PATH for this stage
ENV PATH="/root/.local/bin:$PATH"

# Install JS tools
RUN npm install -g \
    prettier \
    @biomejs/biome

# Install hadolint
RUN wget -O /usr/local/bin/hadolint \
    https://github.com/hadolint/hadolint/releases/download/v2.12.0/hadolint-Linux-x86_64 && \
    chmod +x /usr/local/bin/hadolint

# Pre-install hooks with your config
COPY .pre-commit-config.yaml /tmp/.pre-commit-config.yaml
WORKDIR /tmp
RUN git init && \
    git config user.email "ci@localhost" && \
    git config user.name "CI Bot" && \
    PRE_COMMIT_HOME=/opt/pre-commit pre-commit install-hooks

# Final stage - runtime only
FROM python:3.12-alpine

# Install only runtime dependencies
RUN apk add --no-cache \
    git \
    shellcheck \
    nodejs \
    npm \
    bash

# Copy pre-built tools and cache from builder
COPY --from=builder /root/.local /root/.local
COPY --from=builder /usr/local/bin/hadolint /usr/local/bin/hadolint
COPY --from=builder /usr/local/lib/node_modules /usr/local/lib/node_modules
COPY --from=builder /usr/local/bin/prettier /usr/local/bin/prettier
COPY --from=builder /usr/local/bin/biome /usr/local/bin/biome
COPY --from=builder /opt/pre-commit /opt/pre-commit
COPY --from=builder /tmp/.pre-commit-config.yaml /.pre-commit-config.yaml

# Set environment variables
ENV PATH="/root/.local/bin:$PATH" \
    PRE_COMMIT_HOME="/opt/pre-commit"

# Set git config
RUN git config --global --add safe.directory '*' && \
    git config --global user.email "ci@localhost" && \
    git config --global user.name "CI Bot"

WORKDIR /workspace

# Simple entry point
CMD ["pre-commit", "run", "--all-files"]
