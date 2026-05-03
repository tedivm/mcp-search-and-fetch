## ADDED Requirements

### Requirement: Firecrawl core services start and interconnect

The docker-compose file SHALL define all five Firecrawl core services (api, playwright-service, redis, rabbitmq, nuq-postgres) using upstream images (Firecrawl services from `ghcr.io`, Redis and RabbitMQ from Docker Hub).

#### Scenario: All services start on docker compose up

- **WHEN** the user runs `docker compose up`
- **THEN** all five core services start and become healthy

#### Scenario: API service connects to dependencies

- **WHEN** the api service starts
- **THEN** it connects to redis, rabbitmq, nuq-postgres, and playwright-service on the internal Docker network

### Requirement: Firecrawl API is accessible on host port 3002

The Firecrawl API SHALL be exposed on port 3002 bound to `0.0.0.0` so it's reachable from remote clients.

#### Scenario: Health check endpoint responds remotely

- **WHEN** a GET request is made to `http://<server-ip>:3002`
- **THEN** the response returns a healthy status

### Requirement: Database and cache data are persisted

Redis and PostgreSQL data SHALL persist across container restarts using Docker volumes.

#### Scenario: Data survives restart

- **WHEN** the user runs `docker compose down` then `docker compose up`
- **THEN** PostgreSQL and Redis data are preserved

### Requirement: No build step required for core services

All Firecrawl core images SHALL be pulled from their respective registries (ghcr.io for Firecrawl services, Docker Hub for Redis/RabbitMQ) — no local build is required.

#### Scenario: Fresh start without source code

- **WHEN** the user clones the repo and runs `docker compose up` without cloning the Firecrawl source
- **THEN** all core services start by pulling pre-built images
