# PostgreSQL Docker para Odoo

Infraestructura Docker de PostgreSQL optimizada para despliegues de Odoo ERP.

## Características

- **PostgreSQL** con extensión `unaccent` preconfigurada
- **Template de base de datos** (`unaccent_template`) para creación rápida de bases de datos Odoo
- **Soporte multi-idioma** con configuración de locale (por defecto: es_PE)
- **Optimización de rendimiento** con parámetros PostgreSQL preconfigurados
- **Integración PgAdmin** opcional para administración web
- **Gestión de recursos** Docker con límites de CPU y memoria
- **Multi-usuario** para diferentes versiones de Odoo (odoo_16 a odoo_30)

## Requisitos

- Docker Engine 20.10+
- Docker Compose V2

## Inicio Rápido

```bash
# 1. Clonar el repositorio
git clone https://github.com/focuz-ai/odoo-pg-docker.git -b 18.0 pg_odoo_18
cd pg_odoo_18

# 2. Configurar variables de entorno
cp .env.example .env

# 3. Configurar override para desarrollo local
cp docker-compose.override.local.yml docker-compose.override.yml

# 4. Construir y ejecutar
docker-compose up -d --build postgres
```

## Configuración

### Variables de Entorno Principales

| Variable | Valor por Defecto | Descripción |
|----------|-------------------|-------------|
| `POSTGRES_TAG` | `17` | Versión de PostgreSQL |
| `DB_PORT` | `5454` | Puerto expuesto en el host |
| `DB_USER` | `odoo` | Usuario de base de datos para Odoo |
| `DB_PASSWORD` | `odoo` | Contraseña del usuario |
| `DB_TEMPLATE` | `unaccent_template` | Template para nuevas bases de datos |
| `LOAD_LANGUAGE` | `es_PE` | Locale del sistema |

### Variables de Optimización PostgreSQL

| Variable | Valor | Descripción |
|----------|-------|-------------|
| `POSTGRES_SHARED_BUFFERS` | `1GB` | Memoria compartida para caché |
| `POSTGRES_EFFECTIVE_CACHE_SIZE` | `2GB` | Estimación de caché del SO |
| `POSTGRES_WORK_MEM` | `64MB` | Memoria por operación de ordenamiento |
| `POSTGRES_MAX_CONNECTIONS` | `200` | Conexiones concurrentes máximas |

### PgAdmin (Opcional)

Para habilitar PgAdmin, establecer `USE_PGADMIN=true` en `.env`:

| Variable | Valor por Defecto | Descripción |
|----------|-------------------|-------------|
| `PGADMIN_EMAIL` | `pgadmin@odoocker.test` | Email de acceso |
| `PGADMIN_PASSWORD` | `pgadmin` | Contraseña de acceso |
| `PGADMING_DB_NAME` | `pgadmin` | Base de datos para PgAdmin |

## Uso

### Comandos Docker Compose

```bash
# Construir imagen
docker-compose build postgres

# Iniciar servicio
docker-compose up -d postgres

# Ver logs en tiempo real
docker-compose logs -f postgres

# Detener servicio
docker-compose down

# Reconstruir e iniciar
docker-compose up -d --build postgres

# Ver estado
docker-compose ps
```

### Conexión a PostgreSQL

```bash
# Desde el host
psql -h localhost -p 5454 -U odoo -d postgres

# Desde dentro del contenedor
docker exec -it <nombre_contenedor> psql -U postgres

# Listar bases de datos
docker exec -it <nombre_contenedor> psql -U postgres -c "\l"
```

### Crear Nueva Base de Datos Odoo

```sql
-- Usando el template con unaccent
CREATE DATABASE mi_empresa WITH TEMPLATE unaccent_template OWNER odoo;
```

O desde línea de comandos:

```bash
docker exec -it <nombre_contenedor> createdb -U odoo -T unaccent_template mi_empresa
```

## Arquitectura

```
pg_docker/
├── docker-compose.yml              # Definición principal del servicio
├── docker-compose.override.yml     # Configuración por defecto (puertos, recursos)
├── docker-compose.override.local.yml       # Configuración para desarrollo local
├── docker-compose.override.production.yml  # Configuración para producción
├── .env.example                    # Plantilla de variables de entorno
├── .env                            # Configuración runtime (ignorado en git)
└── postgres/
    ├── Dockerfile                  # Imagen personalizada de PostgreSQL
    └── entrypoint.sh               # Script de inicialización de BD
```

### Flujo de Inicialización

1. **Build**: Dockerfile construye imagen con `postgresql-contrib` y locale configurado
2. **Startup**: Al iniciar el contenedor por primera vez, `entrypoint.sh` se ejecuta automáticamente
3. **Template**: Se crea `unaccent_template` con extensión unaccent marcada como IMMUTABLE
4. **Usuario Principal**: Se crea usuario Odoo con privilegios SUPERUSER y CREATEDB
5. **Usuarios Versionados**: Se crean usuarios odoo_16 hasta odoo_30 para múltiples versiones
6. **PgAdmin**: Si `USE_PGADMIN=true`, se crean usuario y base de datos para PgAdmin

### Recursos Docker

Por defecto, el contenedor está configurado con:

| Recurso | Límite | Reserva |
|---------|--------|---------|
| CPU | 4 cores | 2 cores |
| Memoria | 4 GB | 2 GB |

## Entornos

### Desarrollo Local

```bash
# Usar configuración local
cp docker-compose.override.local.yml docker-compose.override.yml
docker-compose up -d --build postgres
```

### Producción

```bash
# Usar configuración de producción
cp docker-compose.override.production.yml docker-compose.override.yml
docker-compose up -d --build postgres
```

> **Importante**: En producción, cambiar las contraseñas por defecto en `.env`

## Extensión Unaccent

La extensión `unaccent` permite búsquedas de texto sin distinción de acentos, esencial para Odoo en español:

```sql
-- Ejemplo: buscar "García" también encuentra "Garcia"
SELECT * FROM res_partner WHERE unaccent(name) ILIKE unaccent('%garcia%');
```

La función está marcada como `IMMUTABLE` para optimizar índices y consultas.

## Volúmenes

Los datos de PostgreSQL se persisten en el volumen Docker `pg-data`:

```bash
# Ver volúmenes
docker volume ls

# Backup del volumen
docker run --rm -v pg_docker_pg-data:/data -v $(pwd):/backup alpine tar czf /backup/pg-backup.tar.gz -C /data .

# Eliminar volumen (PRECAUCIÓN: elimina todos los datos)
docker-compose down -v
```

## Troubleshooting

### El contenedor no inicia

```bash
# Verificar logs
docker-compose logs postgres

# Verificar que el puerto no esté en uso
netstat -an | grep 5454
```

### Error de permisos en entrypoint.sh

```bash
# Asegurar permisos de ejecución
chmod +x postgres/entrypoint.sh
```

### Reiniciar desde cero

```bash
# Eliminar contenedor y volumen
docker-compose down -v

# Reconstruir
docker-compose up -d --build postgres
```

## Licencia

MIT License
