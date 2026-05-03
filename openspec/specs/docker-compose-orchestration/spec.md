## Purpose

Define the Docker Compose orchestration that brings all 7 services together with proper networking, configuration, and dependency management.

## Requirements

### Requirement: Single docker-compose.yaml orchestrates all services

The project SHALL have a single `docker-compose.yaml` file that defines all seven services (5 Firecrawl core + 2 MCP servers).

#### Scenario: One command starts everything

- **WHEN** the user runs `docker compose up`
- **THEN** all seven services start in the correct dependency order

### Requirement: .env.example provides configuration template

The project SHALL include an `.env.example` file with all required and optional environment variables documented.

#### Scenario: User can start by copying env file

- **WHEN** the user copies `.env.example` to `.env` and fills in their Serper API key
- **THEN** `docker compose up` works without further configuration

### Requirement: Services share a Docker network

All services SHALL be on the same Docker network so internal service discovery works by container name.

#### Scenario: MCP servers reach Firecrawl by name

- **WHEN** the Firecrawl MCP server resolves `http://firecrawl:3000`
- **THEN** the request reaches the Firecrawl API container

### Requirement: Only necessary ports are exposed to host

The docker-compose file SHALL expose only three ports to the host: 3002 (Firecrawl API), 8080 (Firecrawl MCP), and 8894 (Serper MCP).

#### Scenario: Internal ports are not exposed

- **WHEN** the user runs `docker compose ps`
- **THEN** only ports 3002, 8080, and 8894 are mapped to the host
