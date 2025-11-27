# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [0.1.0] - 2025-11-26

### Added
- Initial release of Traefik to Pi-hole DNS Sync tool.
- Automatic discovery of HTTP routers from Traefik API.
- Support for Pi-hole v6 API with session-based authentication.
- DNS A record synchronization (Traefik -> Pi-hole).
- Configurable sync interval using cron syntax or `@every` duration.
- `RUN_MODE` environment variable support (`dry-run`, `once`, `scheduled-dry-run`).
- Multi-architecture Docker image support (`amd64`, `arm64`).
- Distroless base image for enhanced security and minimal footprint.
- Comprehensive logging and error handling.
- `docker-compose.yml` for easy deployment.
