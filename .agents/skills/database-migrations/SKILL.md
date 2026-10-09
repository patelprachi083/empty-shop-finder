---
name: database-migrations
description: >-
  Use this skill when creating or modifying database schema, running Alembic migrations,
  writing PostGIS geo queries, or working with pgvector embeddings in the
  Supabase PostgreSQL database for Empty Shop Finder.
---

# Database Migrations Skill

## Tools Used
- **ORM**: SQLAlchemy 2 (async)
- **Migrations**: Alembic
- **Extensions**: PostGIS (geo), pgvector (AI embeddings)
- **Database**: Supabase PostgreSQL 16

## Migration Commands
```bash
cd apps/api

# Apply all pending migrations
uv run alembic upgrade head

# Rollback one migration
uv run alembic downgrade -1

# Auto-generate migration from model changes
uv run alembic revision --autogenerate -m "short description of change"

# Check current migration version
uv run alembic current

# View migration history
uv run alembic history --verbose
```

## Rules for Writing Migrations
1. **Never edit existing migration files** — always create a new revision.
2. Always add `IF NOT EXISTS` to `CREATE INDEX` statements in migrations.
3. Extensions (`postgis`, `vector`) are enabled once in the initial migration.
4. Test rollback (`downgrade`) before merging a migration PR.
5. Migrations run as a **pre-deploy step** in CI — not at app startup.

## PostGIS Geo Query Pattern
```python
# ST_DWithin for radius search (always use geography type, not geometry)
from sqlalchemy import text

results = await db.execute(text("""
    SELECT id, title, city,
           ST_Distance(location::geography, ST_MakePoint(:lng, :lat)::geography) AS distance_m
    FROM listings
    WHERE ST_DWithin(
        location::geography,
        ST_MakePoint(:lng, :lat)::geography,
        :radius_m
    )
    AND status = 'ACTIVE'
    ORDER BY distance_m
    LIMIT :limit
"""), {"lat": lat, "lng": lng, "radius_m": radius_km * 1000, "limit": 20})
```

## pgvector Pattern
```python
# Store embedding on listing create/update
from pgvector.sqlalchemy import Vector

class Listing(Base):
    embedding: Mapped[list[float]] = mapped_column(Vector(384), nullable=True)

# Cosine similarity query
results = await db.execute(text("""
    SELECT id, 1 - (embedding <=> :query_vec::vector) AS similarity
    FROM listings
    WHERE status = 'ACTIVE'
    ORDER BY embedding <=> :query_vec::vector
    LIMIT 6
"""), {"query_vec": str(query_embedding)})
```

## Adding a New Table (Checklist)
- [ ] Create SQLAlchemy model in `app/models/<name>.py`
- [ ] Add appropriate indexes (GiST for geo, IVFFlat for vector, GIN for full-text)
- [ ] Run `alembic revision --autogenerate`
- [ ] Review generated migration carefully (check for missing indexes)
- [ ] Run `alembic upgrade head` locally
- [ ] Run `alembic downgrade -1` to verify rollback works
- [ ] Commit both model and migration files together
