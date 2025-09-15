FROM mcr.microsoft.com/devcontainers/ruby:1-3.3-bullseye

# System deps for Rails + PG + JS bundling
RUN apt-get update && \
    apt-get install -y build-essential libpq-dev curl git && \
    curl -fsSL https://deb.nodesource.com/setup_20.x | bash - && \
    apt-get install -y nodejs && npm install -g yarn && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /workspace
