## Purpose

Define the Serper MCP server that wraps the serper.dev Google Search API and exposes MCP over SSE transport on port 8894.

## Requirements

### Requirement: Serper MCP server runs in SSE mode

The Serper MCP server SHALL run in SSE transport mode (`-t sse`), exposing the `/sse` endpoint on port 8894.

#### Scenario: SSE endpoint is reachable remotely

- **WHEN** the user connects to `http://<server-ip>:8894/sse`
- **THEN** the server establishes an SSE connection for MCP communication

### Requirement: Serper MCP server requires SERPER_API_KEY

The Serper MCP server SHALL read the `SERPER_API_KEY` environment variable and use it for all serper.dev API calls.

#### Scenario: Search tool works with valid key

- **WHEN** `SERPER_API_KEY` is set and the client calls `google_search`
- **THEN** the server returns search results from serper.dev

#### Scenario: Server warns when key is missing

- **WHEN** `SERPER_API_KEY` is not set
- **THEN** the server logs a warning and tool calls return an error message

### Requirement: Custom Dockerfile builds Go binary

A Dockerfile in the project root SHALL compile the Go Serper MCP server binary using a multi-stage build, producing a small final image.

#### Scenario: Image builds from source

- **WHEN** `docker compose build serper-mcp` is run
- **THEN** the Go binary is compiled and the final image is under 20MB
