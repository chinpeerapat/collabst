# Deploy and Host Collabst with Railway

Collabst is a self-hosted collaborative workspace for Typst projects. It combines a browser editor, live Typst preview, project sharing, comments, and collaborative editing in one deployable web application.

## About Hosting Collabst

Hosting Collabst requires a web application, PostgreSQL for project and account data, Redis for collaboration and realtime coordination, and S3-compatible object storage for uploaded assets and profile images. This Railway template packages those pieces together so users can deploy the full stack from one place.

Collabst is intended for trusted teams, labs, and personal collaborative workspaces. It is not affiliated with Typst and should not be treated as a managed enterprise document platform without additional operational review.

## Common Use Cases

- Run a private Typst collaboration workspace for a research lab or small team.
- Share Typst projects with collaborators from a browser.
- Edit documents with live preview, comments, assets, and presence.
- Self-host a lightweight alternative workflow for Typst drafts and experiments.

## Dependencies for Collabst Hosting

- Collabst web application, built from `chinpeerapat/collabst`.
- PostgreSQL for persistent application data.
- Redis for realtime collaboration coordination.
- Railway Object Storage bucket for uploaded files and profile pictures.

### Deployment Dependencies

- A Railway account with enough resources for one web service, PostgreSQL, Redis, and one bucket.
- Public networking enabled on the `collabst` web service.
- Registration enabled for first-time setup, or user accounts seeded separately.

### Implementation Details

The web service builds from `backend/Dockerfile.prod`, which bundles the Svelte frontend into the FastAPI backend image. The backend runs database migrations on startup and serves both the API and static frontend from one Railway service.

Recommended app service variables:

```dotenv
ENVIRONMENT=production
API_V1_STR=/api/v1
FRONTEND_DIST_DIR=/app/frontend-dist
PORT=8000
WEB_URL=https://${{RAILWAY_PUBLIC_DOMAIN}}
CORS_ORIGINS=https://${{RAILWAY_PUBLIC_DOMAIN}}
DATABASE_URL=postgresql+asyncpg://${{Postgres.PGUSER}}:${{Postgres.PGPASSWORD}}@${{Postgres.PGHOST}}:${{Postgres.PGPORT}}/${{Postgres.PGDATABASE}}
REDIS_URL=${{Redis.REDIS_URL}}
MINIO_ENDPOINT=${{collabst.ENDPOINT}}
MINIO_PUBLIC_ENDPOINT=${{collabst.ENDPOINT}}
MINIO_ACCESS_KEY=${{collabst.ACCESS_KEY_ID}}
MINIO_SECRET_KEY=${{collabst.SECRET_ACCESS_KEY}}
MINIO_BUCKET_NAME=${{collabst.BUCKET}}
MINIO_SECURE=true
MINIO_PUBLIC_SECURE=true
SECRET_KEY=${{secret(64)}}
ALGORITHM=HS256
REGISTRATION_ENABLED=true
```

Recommended app service settings:

- Source: `chinpeerapat/collabst`, branch `main`
- Builder: Dockerfile
- Dockerfile path: `backend/Dockerfile.prod`
- Public HTTP service: enabled
- Healthcheck path: `/api/v1/health`
- Healthcheck timeout: `300`

## Why Deploy Collabst on Railway?

Railway gives Collabst a complete hosting stack without requiring users to hand-wire servers, databases, Redis, storage credentials, public networking, and deployment health checks. By deploying Collabst on Railway, users can run the full application with managed infrastructure and keep future updates tied to the source repository.

## Publish Checklist

1. Create the template from the working Railway project.
2. In the template composer, confirm the services are named `collabst`, `Postgres`, `Redis`, and `collabst` bucket.
3. Replace any copied literal secrets with template functions or reference variables from `.env.railway.example`.
4. Confirm the `collabst` service has public HTTP networking enabled.
5. Confirm the service healthcheck is `/api/v1/health`.
6. Publish with category `Other`, description `Self-host a collaborative Typst workspace with Postgres, Redis, and object storage.`
