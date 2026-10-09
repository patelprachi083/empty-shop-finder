# API Sub-directory Rules

## Python FastAPI — Core Backend

### Service Layer Rules
- All database queries go through **SQLAlchemy async sessions** — never raw `asyncpg` directly in routers.
- Every router function must call a corresponding service function — no DB logic in routers.
- All service functions must have **type-annotated** parameters and return types.
- Raise `HTTPException` only in routers. Services raise domain-specific `ValueError` or custom exceptions.

### Response Format
All API responses must follow this envelope:
```python
# Success
{"success": True, "data": {...}, "meta": {"cursor": "...", "total": 0}}
# Error
{"success": False, "error": {"code": "LISTING_NOT_FOUND", "message": "..."}}
```

### Async Rules
- All DB calls, Redis calls, and external API calls must be `await`-ed.
- Never use `time.sleep()` — use `asyncio.sleep()`.
- Background tasks use **Celery** — never `BackgroundTasks` for long-running work.

### Testing
```bash
uv run pytest tests/ -v --cov=app --cov-report=term-missing
# Minimum 80% coverage on app/services/ required
```
