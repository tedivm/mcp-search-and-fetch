## ADDED Requirements

### Requirement: Firecrawl MCP server runs in HTTP streamable mode

The Firecrawl MCP server SHALL run in HTTP streamable mode, exposing the `/mcp` endpoint on port 8080.

#### Scenario: MCP endpoint is reachable remotely

- **WHEN** the user sends an MCP handshake to `http://<server-ip>:8080/mcp`
- **THEN** the server responds with a valid MCP initialization

### Requirement: Firecrawl MCP server points to local Firecrawl instance

The Firecrawl MCP server SHALL use `FIRECRAWL_API_URL` set to the internal Firecrawl API service URL (`http://firecrawl:3002`).

#### Scenario: MCP tools use local Firecrawl

- **WHEN** an MCP client calls `firecrawl_scrape` via the MCP server
- **THEN** the request is routed to the local Firecrawl API, not the cloud service

### Requirement: No API key required for self-hosted mode

The Firecrawl MCP server SHALL work without `FIRECRAWL_API_KEY` when pointing to a self-hosted instance with authentication disabled.

#### Scenario: MCP tools work without API key

- **WHEN** `FIRECRAWL_API_KEY` is not set and `FIRECRAWL_API_URL` points to the local instance
- **THEN** MCP tools execute successfully against the local Firecrawl API
