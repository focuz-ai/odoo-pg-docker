# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

pg_odoo_19 is a Docker-based PostgreSQL 17 infrastructure for Odoo 19 ERP deployments. Key features:
- PostgreSQL with `unaccent` and `pgvector` extensions pre-configured
- Database template (`unaccent_template`) with extensions for rapid Odoo database creation
- pgvector 0.8.1 for AI/vector similarity search (required by Odoo 19 ai_app addon)
- Multi-language/locale support (default: es_PE)
- Optional PgAdmin integration

## Common Commands

```bash
# Build and start
docker-compose up -d --build postgres

# View logs
docker-compose logs -f postgres

# Stop and remove containers
docker-compose down

# Connect to PostgreSQL from host (default port 5432, override sets 5454)
psql -h localhost -p 5432 -U odoo -d postgres

# Execute psql inside container
docker exec -it <container_name> psql -U postgres

# Create new Odoo database using template
docker exec -it <container_name> createdb -U odoo -T unaccent_template my_database

# Reset everything (WARNING: deletes all data)
docker-compose down -v && docker-compose up -d --build postgres
```

## Architecture

**Key Files:**
- `docker-compose.yml` - Base service definition (no ports exposed)
- `docker-compose.override.yml` - Active config (copy from local or production)
- `docker-compose.override.local.yml` - Local dev: 1 CPU / 1GB reserved, port 5454
- `docker-compose.override.production.yml` - Production: 2 CPU / 2GB reserved, port 5454
- `postgres/Dockerfile` - Custom image with pgvector built from source, locale configured
- `postgres/entrypoint.sh` - Creates template DB, extensions, and Odoo user

**Initialization Flow (on first container start):**
1. Dockerfile builds image with `postgresql-contrib` and pgvector (compiled from source)
2. `entrypoint.sh` runs via Docker's initdb mechanism
3. Creates `unaccent_template` database with `unaccent` (IMMUTABLE) and `vector` extensions
4. Creates Odoo user (`$DB_USER`) with SUPERUSER and CREATEDB privileges
5. If `USE_PGADMIN=true`, creates PgAdmin database and user

## Key Environment Variables

See `.env.example` for all variables. Critical ones:

| Variable | Default | Description |
|----------|---------|-------------|
| `POSTGRES_TAG` | 17 | PostgreSQL version |
| `PGVECTOR_VERSION` | 0.8.1 | pgvector extension version |
| `DB_PORT` | 5432 | Internal port (override exposes to host) |
| `DB_USER` | odoo | Odoo database user |
| `DB_PASSWORD` | odoo | Database password |
| `DB_TEMPLATE` | unaccent_template | Template for new Odoo DBs |
| `LOAD_LANGUAGE` | es_PE | System locale for PostgreSQL |

## Development Setup

```bash
cp .env.example .env
cp docker-compose.override.local.yml docker-compose.override.yml
docker-compose up -d --build postgres
```

## Modifying the Image

When changing `postgres/Dockerfile`:
- pgvector is compiled from source using `PG_MAJOR` from the base postgres image
- Build dependencies are removed after compilation to reduce image size
- The `.env` file is copied into the image for `entrypoint.sh` to source

When changing `postgres/entrypoint.sh`:
- Script runs as postgres user during container initialization
- Uses variables from `/.env` (sourced at runtime)
- Ensure script has execute permissions: `chmod +x postgres/entrypoint.sh`
