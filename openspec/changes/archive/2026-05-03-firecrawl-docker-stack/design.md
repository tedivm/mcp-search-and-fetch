## Context

Blank repo. The goal is a single `docker compose up` command that starts a fully working self-hosted Firecrawl instance with two MCP servers exposed over HTTP, connectable from any MCP client on the host machine.

**Services involved:**

- Firecrawl core (5 upstream containers): api, playwright-service, redis, rabbitmq, nuq-postgres
- Firecrawl MCP server (1 container): wraps the Firecrawl API, exposes MCP over HTTP streamable + SSE
- Serper MCP server (1 container): wraps serper.dev Google Search API, exposes MCP over SSE

**Key constraint:** MCP clients (Cursor, Claude Desktop) connect over HTTP to containers. Stdio transport doesn't work across container boundaries, so both MCP servers must run in HTTP/SSE mode.

## Goals / Non-Goals

**Goals:**

- Single `docker compose up` starts all 7 services
- All upstream images pulled from registries (Firecrawl from `ghcr.io`, Redis/RabbitMQ from Docker Hub) — no building Firecrawl core
- One custom Dockerfile only for Serper MCP (Go binary, no upstream image exists)
- Firecrawl MCP container built from upstream repo's Dockerfile
- `.env` file controls all secrets and configuration
- MCP clients connect via `http://<server-ip>:8080/mcp` (Firecrawl) and `http://<server-ip>:8894/sse` (Serper) — all services bind to `0.0.0.0`

**Non-Goals:**

- Building Firecrawl from source
- Authentication/Supabase setup
- Production hardening (TLS, reverse proxy, etc.) — external ingress is out of scope but services must bind to `0.0.0.0` for remote access
- Custom MCP server implementations

## Decisions

**1. Pull vs. build for Firecrawl core**

- Decision: Pull pre-built images from `ghcr.io/firecrawl/firecrawl`, `ghcr.io/firecrawl/nuq-postgres`, `ghcr.io/firecrawl/playwright-service`
- Rationale: Faster startup, smaller local footprint, upstream maintains the images. No need to clone the Firecrawl repo.

**2. Serper MCP: Go implementation (`hightemp/go_serper_mcp_server`)**

- Decision: Use the Go implementation with a custom multi-stage Dockerfile
- Rationale: SSE transport built-in (default `-t sse`), full Serper API surface (13 tools), actively maintained (Jan 2026), small binary (~10MB). Python alternative (garylab) is stdio-only and would need a custom HTTP bridge.

**3. Firecrawl MCP: Build from upstream Dockerfile**

- Decision: Use the upstream `firecrawl/firecrawl-mcp-server` Dockerfile with nginx, run in HTTP streamable mode
- Rationale: Officially maintained, active development (May 2026), nginx handles both `/mcp` (HTTP streamable) and `/sse` endpoints. Set `FIRECRAWL_API_URL=http://firecrawl:3002` to point at the internal Firecrawl API.

**4. Network topology**

- Decision: Single Docker network. All services bind to `0.0.0.0`. Ports 3002, 8080, 8894 exposed to host. MCP clients connect via the server's IP address.
- Rationale: This is a centralized install — clients on other machines need to reach the MCP endpoints. All services must listen on all interfaces, not just localhost. An external ingress already exists for this project but is out of scope here.

**5. `.env` file for all configuration**

- Decision: Single `.env` at project root with `.env.example` as template
- Rationale: Docker Compose reads `.env` automatically. Keeps secrets out of the repo.

## Architecture

```
Host Machine                    Docker Network
────────────                    ──────────────

Cursor / Claude Desktop
        │
        │  http://<server-ip>:8080/mcp
        ▼
  ┌──────────────────────┐
  │ firecrawl-mcp-server  │  :8080 (exposed, 0.0.0.0)
  │ (HTTP streamable)     │
  └──────────┬───────────┘
             │ FIRECRAWL_API_URL=http://firecrawl:3002
             ▼
  ┌──────────────────────┐     ┌──────────────┐
  │   firecrawl (api)    │◄────│  playwright   │
  │   :3002 (exposed)    │     │  :3000       │
  └──────────┬───────────┘     └──────────────┘
             │
     ┌───────┼───────────┐
     ▼       ▼           ▼
  ┌──────┐ ┌────────┐ ┌──────────┐
  │redis │ │rabbitmq│ │ nuq-pg   │
  │:6379 │ │:5672   │ │ :5432    │
  └──────┘ └────────┘ └──────────┘

        │  http://<server-ip>:8894/sse
        ▼
  ┌──────────────────────┐
  │   serper-mcp-server   │  :8894 (exposed, 0.0.0.0)
  │  (SSE, -t sse)       │
  └──────────┬───────────┘
             │ API calls to serper.dev (external)
             ▼
       serper.dev
```

## Risks / Trade-offs

- [Firecrawl image tags drift] → Pin specific image tags or use `latest` with awareness that updates may break. Mitigation: pin to a known-good tag in `.env.example`.
- [Serper Go server has no official Docker image] → We write a minimal multi-stage Dockerfile. Mitigation: keep it simple, build from `golang:1.23-alpine`.
- [Resource usage] → Full stack needs ~12GB RAM. Mitigation: document requirements, allow resource tuning via `.env`.
- [Self-hosted Firecrawl lacks `/agent` and `/browser` endpoints] → Documented limitation. MCP server tools for these will fail gracefully.

## Open Questions

- None remaining.
