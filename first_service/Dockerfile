# syntax=docker/dockerfile:1

# Specify the Ruby version (make sure it matches your .ruby-version)
ARG RUBY_VERSION=3.3.1
FROM docker.io/library/ruby:$RUBY_VERSION-slim AS base

# Set working directory
WORKDIR /rails

# Install base packages
RUN apt-get update -qq && \
    apt-get install --no-install-recommends -y \
      curl \
      libjemalloc2 \
      libvips \
      sqlite3 && \
    rm -rf /var/lib/apt/lists /var/cache/apt/archives

# Set development environment
ENV RAILS_ENV="development" \
    BUNDLE_PATH="/usr/local/bundle"

# Build stage: install development gems
FROM base AS build

# Install packages needed to build gems
RUN apt-get update -qq && \
    apt-get install --no-install-recommends -y \
      build-essential \
      git \
      pkg-config && \
    rm -rf /var/lib/apt/lists /var/cache/apt/archives

# Install application gems (including development group)
COPY Gemfile Gemfile.lock ./
RUN bundle install

# Copy application code
COPY . .

# Final stage: use source code as-is for development
FROM base

# Copy gems and application code from build stage
COPY --from=build "${BUNDLE_PATH}" "${BUNDLE_PATH}"
COPY --from=build /rails /rails

# Create a non-root user for a safer dev experience (optional)
RUN groupadd --system --gid 1000 rails && \
    useradd rails --uid 1000 --gid 1000 --create-home --shell /bin/bash && \
    chown -R rails:rails /rails

USER 1000:1000

# Expose Rails default port
EXPOSE 3000

# Start Rails server in development mode; override if needed
CMD ["bundle", "exec", "rails", "server", "-b", "0.0.0.0"]
