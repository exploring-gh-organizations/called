FROM python:3.12-slim AS builder

# Install build dependencies
# hadolint ignore=DL3008
RUN apt-get update && apt-get install -y --no-install-recommends \
    git \
    curl \
    wget \
    ca-certificates \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

# Install pre-commit and dependencies
RUN pip install --no-cache-dir pre-commit

# Copy the config
COPY .pre-commit-config.yaml /.pre-commit-config.yaml

# Set up pre-commit cache directory
ENV PRE_COMMIT_HOME=/opt/pre-commit-cache

# Pre-install ALL the hooks with proper caching
WORKDIR /tmp/setup
RUN git init && \
    git config user.email "test@test.com" && \
    git config user.name "Test" && \
    cp /.pre-commit-config.yaml . && \
    # Create dummy files to trigger all hooks
    touch test.py test.js test.java test.sh test.yaml test.json Dockerfile README.md test.png && \
    git add . && \
    # Install all environments
    pre-commit install-hooks && \
    # Run once to ensure everything is cached
    pre-commit run --all-files || true

# Clean up unnecessary cache files
RUN find /opt/pre-commit-cache -type f -name "*.pyc" -delete && \
    find /opt/pre-commit-cache -type d -name "__pycache__" -delete && \
    find /opt/pre-commit-cache -type f -name "*.whl" -delete

# Final stage - smaller runtime
FROM python:3.12-slim

# Install only runtime dependencies
# hadolint ignore=DL3008
RUN apt-get update && apt-get install -y --no-install-recommends \
    git \
    curl \
    shellcheck \
    nodejs \
    npm \
    default-jre-headless \
    && rm -rf /var/lib/apt/lists/* \
    && apt-get clean \
    && rm -rf /var/cache/apt/archives

# Install hadolint separately
RUN curl -sSL https://github.com/hadolint/hadolint/releases/download/v2.12.0/hadolint-Linux-x86_64 \
    -o /usr/local/bin/hadolint && \
    chmod +x /usr/local/bin/hadolint

# Install pre-commit
RUN pip install --no-cache-dir pre-commit

# Copy pre-commit cache from builder
COPY --from=builder /opt/pre-commit-cache /opt/pre-commit-cache
COPY --from=builder /.pre-commit-config.yaml /.pre-commit-config.yaml

# Set environment
ENV PRE_COMMIT_HOME=/opt/pre-commit-cache

# Configure git for GitHub Actions
RUN git config --global --add safe.directory '*'

WORKDIR /workspace

ENTRYPOINT ["pre-commit"]
CMD ["run", "--all-files"]
