---
name: fastapi-backend
description: >-
  Use this skill when working on the Python FastAPI core backend (apps/api/).
  Covers service patterns, Alembic migrations, Pydantic schemas, SQLAlchemy
  async models, Celery tasks, Razorpay integration, Signzy KYC, and Cloudflare R2 uploads.
---

# FastAPI Backend Skill

## Project Location
All backend code lives in `apps/api/`.

## Key Commands
```bash
# Start dev server
cd apps/api && uv run uvicorn app.main:app --reload --port 8000

# Run migrations
cd apps/api && uv run alembic upgrade head

# Create a new migration
cd apps/api && uv run alembic revision --autogenerate -m "describe change"

# Run tests
cd apps/api && uv run pytest tests/ -v --cov=app

# Lint & format
cd apps/api && uv run ruff check . && uv run ruff format .
```

## Adding a New Feature (Standard Pattern)

1. **Add SQLAlchemy model** in `app/models/<feature>.py`
2. **Add Pydantic schema** in `app/schemas/<feature>.py` (Request + Response DTOs)
3. **Add service** in `app/services/<feature>_service.py` (all business logic here)
4. **Add router** in `app/routers/<feature>.py` (HTTP layer only, calls service)
5. **Register router** in `app/main.py`
6. **Generate migration**: `uv run alembic revision --autogenerate -m "add <feature> table"`
7. **Write tests** in `tests/unit/` and `tests/integration/`

## Dependency Injection Pattern
```python
# app/core/dependencies.py
from fastapi import Depends, HTTPException, status
from app.core.security import verify_token

async def get_current_user(token: str = Depends(oauth2_scheme)) -> User:
    payload = verify_token(token)
    user = await UserService.get_by_id(payload["sub"])
    if not user:
        raise HTTPException(status_code=401)
    return user

async def require_owner(user: User = Depends(get_current_user)) -> User:
    if user.role != "OWNER":
        raise HTTPException(status_code=403, detail="Owner access required")
    return user
```

## Booking State Machine (Always use this — never bypass)
```python
from app.services.booking_service import transition, BookingStatus
new_status = transition(booking.status, BookingStatus.APPROVED)
```

## Razorpay Payment Flow
1. `POST /api/v1/payments/create-order` → call `payment_service.create_order()`
2. Client completes payment in UI
3. `POST /api/v1/payments/verify` → call `payment_service.verify_signature()`
4. On success → transition booking to `CONFIRMED`
5. Webhook at `POST /api/v1/payments/webhook` for async confirmation (idempotent)

## Environment Variables Required
```
DATABASE_URL=postgresql+asyncpg://...
REDIS_URL=redis://...
RAZORPAY_KEY_ID=
RAZORPAY_KEY_SECRET=
MSG91_API_KEY=
SIGNZY_API_KEY=
CF_ACCOUNT_ID=
CF_R2_ACCESS_KEY=
CF_R2_SECRET_KEY=
CF_R2_BUCKET=
CF_PUBLIC_URL=
MEILISEARCH_URL=
MEILISEARCH_KEY=
JWT_SECRET=
JWT_ALGORITHM=HS256
```
