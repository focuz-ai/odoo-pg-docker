# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

pg_docker is a Docker-based PostgreSQL 17 infrastructure specifically designed for Odoo ERP deployments. It provides:
- Containerized PostgreSQL with the unaccent extension pre-configured
- Database template (`unaccent_template`) for rapid Odoo database creation
- Multi-language/locale support (default: es_PE - Spanish Peru)
- Optional PgAdmin integration
- Multi-user creation for different Odoo versions (odoo_16 to odoo_30)

## Common Commands

```bash
# Build the PostgreSQL image
docker-compose build postgres

# Start the service
docker-compose up -d postgres

# View logs
docker-compose logs -f postgres

# Stop and remove containers
docker-compose down

# Rebuild and start (after Dockerfile changes)
docker-compose up -d --build postgres

# Connect to PostgreSQL from host
psql -h localhost -p 5454 -U odoo -d <database_name>

# Execute psql inside container
docker exec -it <container_name> psql -U postgres
```

## Architecture

```
pg_docker/
├── docker-compose.yml              # Main service definition
├── docker-compose.override.yml     # Default config (ports, resources, tuning)
├── docker-compose.override.*.yml   # Environment-specific overrides
├── .env.example                    # Environment template
├── .env                            # Runtime config (not in git)
└── postgres/
    ├── Dockerfile                  # Custom PostgreSQL image
    └── entrypoint.sh               # Database initialization script
```

**Initialization Flow:**
1. Dockerfile builds custom PostgreSQL image with `postgresql-contrib` and locale
2. On first container start, `entrypoint.sh` runs automatically via Docker's init mechanism
3. Script creates `unaccent_template` database with the unaccent extension (marked IMMUTABLE)
4. Creates main Odoo user with SUPERUSER and CREATEDB privileges
5. Creates versioned users (odoo_16 through odoo_30) for multi-version support
6. Optionally creates PgAdmin database/user if `USE_PGADMIN=true`

## Key Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `POSTGRES_TAG` | 17 | PostgreSQL version |
| `DB_PORT` | 5454 | Exposed port on host |
| `DB_USER` | odoo | Odoo database user |
| `DB_PASSWORD` | odoo | Database password |
| `DB_TEMPLATE` | unaccent_template | Template database for new DBs |
| `LOAD_LANGUAGE` | es_PE | System locale |
| `USE_PGADMIN` | false | Enable PgAdmin user/database creation |

## Development Setup

```bash
# 1. Create environment file
cp .env.example .env

# 2. Copy override for local development
cp docker-compose.override.local.yml docker-compose.override.yml

# 3. Build and start
docker-compose up -d --build postgres
```

## Database Template Pattern

The `unaccent_template` database is created with the unaccent extension to enable accent-insensitive text searches in Odoo. New databases can be created using:

```sql
CREATE DATABASE new_db WITH TEMPLATE unaccent_template;
```
