## Why

This project needs a single `docker compose` setup that runs a self-hosted Firecrawl instance with two MCP servers (Firecrawl + Serper) exposed over HTTP/SSE, so any MCP-compatible client (Cursor, Claude Desktop, etc.) can connect and use web scraping and Google search as tools.

## What Changes

- Create a `docker-compose.yaml` pulling upstream images for Firecrawl (api from `ghcr.io`, Redis/RabbitMQ from Docker Hub, nuq-postgres from `ghcr.io`)
- Add a Firecrawl MCP server container (HTTP streamable transport on port 8080) pointing at the self-hosted Firecrawl instance
- Add a Serper MCP server container (SSE transport on port 8894) with a minimal custom Dockerfile for the Go implementation
- Provide an `.env.example` with all required configuration (Serper API key, Firecrawl settings)
- No custom builds of Firecrawl core services — all pulled from upstream registries (`ghcr.io` for Firecrawl, Docker Hub for Redis/RabbitMQ)

## Capabilities

### New Capabilities

- `firecrawl-core`: Self-hosted Firecrawl stack (api, playwright, redis, rabbitmq, postgres) using upstream images
- `firecrawl-mcp`: Firecrawl MCP server container with HTTP streamable transport, connected to the local Firecrawl instance
- `serper-mcp`: Serper MCP server container with SSE transport, wrapping serper.dev's Google Search API
- `docker-compose-orchestration`: Single docker-compose.yaml and .env file orchestrating all services

### Modified Capabilities

## Impact

- New files: `docker-compose.yaml`, `.env.example`, `Dockerfile.serper-mcp`
- External dependencies: `ghcr.io/firecrawl/*` images, Docker Hub for Redis/RabbitMQ
- All services bind to `0.0.0.0` for remote client access via server IP
