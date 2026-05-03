## 1. Project scaffolding

- [x] 1.1 Create `.env.example` with all required environment variables (Firecrawl core, MCP servers, secrets)
- [x] 1.2 Create `.gitignore` excluding `.env`, `tmp/`, and Docker artifacts

## 2. Firecrawl core services

- [x] 2.1 Create `docker-compose.yaml` with the five Firecrawl core services (api, playwright-service, redis, rabbitmq, nuq-postgres) using upstream images
- [x] 2.2 Configure internal Docker network, volumes for data persistence, and service dependencies
- [x] 2.3 Set proper environment variables for Firecrawl core (REDIS_URL, NUQ_DATABASE_URL, PLAYWRIGHT_MICROSERVICE_URL, etc.)

## 3. Serper MCP server

- [x] 3.1 Create `Dockerfile.serper-mcp` multi-stage build for the Go Serper MCP server (`hightemp/go_serper_mcp_server`)
- [x] 3.2 Add `serper-mcp` service to docker-compose.yaml with SSE transport (`-t sse`), port 8894, and `SERPER_API_KEY` env var

## 4. Firecrawl MCP server

- [x] 4.1 Add `firecrawl-mcp` service to docker-compose.yaml using the upstream `firecrawl/firecrawl-mcp-server` Dockerfile with nginx
- [x] 4.2 Configure `FIRECRAWL_API_URL=http://firecrawl:3002` and HTTP streamable mode for the Firecrawl MCP server
- [x] 4.3 Expose port 8080 and add dependency on the Firecrawl api service

## 5. Verification

- [x] 5.1 Validate docker-compose syntax with `docker compose config`
- [x] 5.2 Test full stack startup with `docker compose up` and verify all 7 services are healthy
- [x] 5.3 Verify Firecrawl API responds on `http://<server-ip>:3002`
- [x] 5.4 Verify Firecrawl MCP endpoint responds on `http://<server-ip>:8080/mcp`
- [x] 5.5 Verify Serper MCP endpoint responds on `http://<server-ip>:8894/sse`
