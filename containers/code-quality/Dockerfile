FROM python:3.12-slim

# Install system dependencies that hooks need
RUN apt-get update && apt-get install -y \
    git \
    curl \
    wget \
    shellcheck \
    nodejs \
    npm \
    default-jre-headless \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

# Install hadolint separately (it's not auto-installed)
RUN wget -qO /usr/local/bin/hadolint \
    https://github.com/hadolint/hadolint/releases/download/v2.12.0/hadolint-Linux-x86_64 && \
    chmod +x /usr/local/bin/hadolint

# Install pre-commit
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
    touch test.py test.js test.java test.sh test.yaml test.json Dockerfile README.md && \
    git add . && \
    # Install all environments
    pre-commit install-hooks && \
    # Run once to ensure everything is cached
    pre-commit run --all-files || true && \
    # Clean up the temp repo but keep the cache
    cd / && \
    rm -rf /tmp/setup

# Configure git for GitHub Actions
RUN git config --global --add safe.directory '*'

# Set working directory
WORKDIR /workspace

# The cache is now in /opt/pre-commit-cache and will be reused
ENTRYPOINT ["pre-commit"]
CMD ["run", "--all-files"]
