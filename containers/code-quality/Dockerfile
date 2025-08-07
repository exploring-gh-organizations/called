# Stage 1: Build and install all tools
FROM python:3.12-slim AS builder

# Install build dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    git \
    curl \
    build-essential \
    pkg-config \
    libssl-dev \
    ca-certificates \
    && rm -rf /var/lib/apt/lists/*

# Install Rust for tools that need it
RUN curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y --profile minimal
ENV PATH="/root/.cargo/bin:$PATH"

# Create virtual environment for Python tools
RUN python -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

# Install Python tools in venv
COPY containers/code-quality/requirements-precommit.txt /tmp/
RUN pip install --no-cache-dir -r /tmp/requirements-precommit.txt

# Install Rust-based tools
RUN cargo install typos-cli --version 1.30.3
RUN cargo install oxipng --version 9.1.4

# Stage 2: Runtime with minimal dependencies
FROM python:3.12-slim AS runtime

# Install runtime dependencies only
RUN apt-get update && apt-get install -y --no-install-recommends \
    git \
    nodejs \
    npm \
    shellcheck \
    make \
    curl \
    default-jre-headless \
    && rm -rf /var/lib/apt/lists/*

# Copy Python virtual environment
COPY --from=builder /opt/venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

# Copy Rust tools
COPY --from=builder /root/.cargo/bin/typos /usr/local/bin/
COPY --from=builder /root/.cargo/bin/oxipng /usr/local/bin/

# Install hadolint (static binary)
RUN curl -L https://github.com/hadolint/hadolint/releases/download/v2.12.0/hadolint-Linux-x86_64 \
    -o /usr/local/bin/hadolint && \
    chmod +x /usr/local/bin/hadolint

# Install gitleaks (static binary)
RUN curl -L https://github.com/gitleaks/gitleaks/releases/download/v8.24.2/gitleaks_8.24.2_linux_x64.tar.gz | \
    tar xz -C /usr/local/bin gitleaks && \
    chmod +x /usr/local/bin/gitleaks

# Install checkmake
RUN curl -L https://github.com/mrtazz/checkmake/releases/download/0.2.2/checkmake-0.2.2.linux.amd64 \
    -o /usr/local/bin/checkmake && \
    chmod +x /usr/local/bin/checkmake

# Install shellcheck-py (for better shellcheck)
RUN pip install shellcheck-py==0.10.0.1

# Install Node.js tools globally
RUN npm install -g \
    @biomejs/biome@1.9.4 \
    prettier@3.4.2 \
    && npm cache clean --force

# Install pretty-format-java (needs Java)
RUN curl -L https://github.com/google/google-java-format/releases/download/v1.24.0/google-java-format-1.24.0-all-deps.jar \
    -o /usr/local/lib/google-java-format.jar && \
    echo '#!/bin/bash\njava -jar /usr/local/lib/google-java-format.jar "$@"' > /usr/local/bin/google-java-format && \
    chmod +x /usr/local/bin/google-java-format

# Configure git for pre-commit
RUN git config --global --add safe.directory '*' && \
    git config --global user.email "ci@opencloudlab.dev" && \
    git config --global user.name "CI Bot"

# Copy and install pre-commit config with all hooks
COPY .pre-commit-config.yaml /opt/pre-commit-config.yaml

# Pre-install all hooks in the container
RUN mkdir -p /tmp/pre-install && \
    cd /tmp/pre-install && \
    git init && \
    cp /opt/pre-commit-config.yaml .pre-commit-config.yaml && \
    pre-commit install-hooks && \
    cd / && \
    rm -rf /tmp/pre-install

# Set environment for pre-commit cache (hooks are already installed)
ENV PRE_COMMIT_HOME=/root/.cache/pre-commit

WORKDIR /workspace

# Create entrypoint script
RUN echo '#!/bin/bash\ncp /opt/pre-commit-config.yaml .pre-commit-config.yaml 2>/dev/null || true\nexec pre-commit "$@"' > /entrypoint.sh && \
    chmod +x /entrypoint.sh

ENTRYPOINT ["/entrypoint.sh"]
CMD ["run", "--all-files", "--color=always"]
