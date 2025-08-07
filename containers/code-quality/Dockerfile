FROM python:3.12-slim

# Install system dependencies that hooks need
RUN apt-get update && apt-get install -y \
    git \
    curl \
    shellcheck \
    nodejs \
    npm \
    default-jre-headless \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

# Install pre-commit
RUN pip install pre-commit

# Copy the config
COPY .pre-commit-config.yaml /.pre-commit-config.yaml

# Pre-install the hooks
WORKDIR /tmp/install
RUN git init && \
    cp /.pre-commit-config.yaml . && \
    pre-commit install-hooks && \
    rm -rf /tmp/install

# Set working directory
WORKDIR /workspace

# Configure git for GitHub Actions
RUN git config --global --add safe.directory '*'

# Run pre-commit
ENTRYPOINT ["pre-commit"]
CMD ["run", "--all-files"]
