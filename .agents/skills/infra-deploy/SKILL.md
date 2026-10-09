---
name: infra-deploy
description: >-
  Use this skill when deploying to GCP Cloud Run, managing Terraform infrastructure,
  running Docker Compose for local development, setting up GitHub Actions CI/CD,
  or configuring Cloudflare R2, Upstash Redis, or Meilisearch for Empty Shop Finder.
---

# Infrastructure & Deployment Skill

## Environments
| Env | API URL | Frontend URL |
|---|---|---|
| Development | http://localhost:8000 | http://localhost:3000 |
| Staging | https://api-staging.emptyshopfinder.in | https://staging.emptyshopfinder.in |
| Production | https://api.emptyshopfinder.in | https://emptyshopfinder.in |

## Local Dev — Docker Compose
```bash
# Start all services (Postgres, Redis, Meilisearch, API, AI service)
docker compose -f infra/docker/docker-compose.yml up -d

# View logs
docker compose -f infra/docker/docker-compose.yml logs -f api

# Stop all
docker compose -f infra/docker/docker-compose.yml down

# Rebuild after code change
docker compose -f infra/docker/docker-compose.yml up -d --build api
```

## GCP Cloud Run Deploy (Manual)
```bash
# Authenticate
gcloud auth login
gcloud config set project <GCP_PROJECT_ID>

# Build and push API image
gcloud builds submit apps/api \
  --tag asia-south1-docker.pkg.dev/<PROJECT>/esf/api:latest

# Deploy API to Cloud Run (Mumbai region)
gcloud run deploy esf-api \
  --image asia-south1-docker.pkg.dev/<PROJECT>/esf/api:latest \
  --region asia-south1 \
  --platform managed \
  --memory 1Gi \
  --cpu 1 \
  --min-instances 0 \
  --max-instances 10 \
  --set-env-vars "ENV=production"

# Deploy AI service (scale-to-zero)
gcloud run deploy esf-ai \
  --image asia-south1-docker.pkg.dev/<PROJECT>/esf/ai:latest \
  --region asia-south1 \
  --platform managed \
  --memory 2Gi \
  --cpu 2 \
  --min-instances 0 \
  --max-instances 5
```

## Terraform Commands
```bash
cd infra/terraform

# Initialise
terraform init

# Preview changes
terraform plan -var-file="production.tfvars"

# Apply (always review plan first)
terraform apply -var-file="production.tfvars"

# Destroy (CAUTION: deletes all resources)
terraform destroy -var-file="production.tfvars"
```

## GitHub Actions Triggers
| Workflow | Trigger | What it does |
|---|---|---|
| `ci.yml` | Every PR | Lint + test all apps |
| `deploy-api.yml` | Push to `main`, `apps/api/**` changed | Build → push to Artifact Registry → run migrations → deploy to Cloud Run |
| `deploy-ai.yml` | Push to `main`, `apps/ai-service/**` changed | Build → push → deploy AI service |
| `deploy-web.yml` | Push to `main`, `apps/web/**` changed | Vercel auto-deploy (via Vercel GitHub integration) |

## Cloudflare R2 Setup
- Bucket name: `esf-media`
- Public access URL: `https://media.emptyshopfinder.in`
- CORS: allow `emptyshopfinder.in` and `localhost:3000`
- All uploads go through the API — never directly from the browser (no signed URLs exposed)

## Meilisearch Re-indexing
```bash
# Full re-index from DB (run after major DB changes)
cd scripts && uv run python index_meilisearch.py --full

# Index a single listing
cd scripts && uv run python index_meilisearch.py --listing-id <uuid>
```

## Upstash Redis
- Connection: use `REDIS_URL` from environment (TLS-enabled Upstash URL)
- Rate limiting: sliding window, 10 OTP requests per phone per hour
- Cache TTLs: listings = 5 min, search results = 30 sec, sessions = 7 days

## Deployment Checklist (Production Release)
- [ ] All tests pass in CI
- [ ] `alembic upgrade head` runs successfully in staging
- [ ] No secrets committed to git
- [ ] Cloud Run health check (`/health`) returns 200
- [ ] Razorpay webhook URL updated to production endpoint
- [ ] Meilisearch index settings confirmed
- [ ] Cloudflare R2 CORS policy set for production domain
