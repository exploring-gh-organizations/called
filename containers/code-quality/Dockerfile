FROM python:3.12-slim

# Install git and pre-commit
RUN apt-get update && apt-get install -y git && rm -rf /var/lib/apt/lists/*
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

# Run pre-commit
ENTRYPOINT ["pre-commit"]
CMD ["run", "--all-files"]
