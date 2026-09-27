# eTribe Railway self-hosting

This fork can publish deployable Firecrawl images to GitHub Container Registry (GHCR) without requiring Railway to have direct GitHub repository access.

## Images

After the **Publish eTribe Firecrawl images** workflow succeeds:

- `ghcr.io/nishantr2/firecrawl:latest`
- `ghcr.io/nishantr2/firecrawl-playwright:latest`
- `ghcr.io/nishantr2/firecrawl-nuq-postgres:latest`

Every build also receives an immutable commit-SHA tag. Prefer SHA tags for production rollbacks.

## Railway stack

Run Firecrawl as separate Railway services:

1. API
2. worker
3. extract worker
4. NuQ worker
5. NuQ prefetch worker
6. NuQ reconciler worker
7. Playwright
8. Redis
9. RabbitMQ
10. NuQ PostgreSQL
11. authenticated gateway

Use the same `firecrawl` image for the API and worker roles, with the role-specific environment/start configuration used by Firecrawl's current Railway template.

Recommended initial limits for eTribe:

- `USE_DB_AUTHENTICATION=false`
- `NUM_WORKERS_PER_QUEUE=4`
- `CRAWL_CONCURRENT_REQUESTS=6`
- `MAX_CONCURRENT_JOBS=3`
- `BROWSER_POOL_SIZE=3`

Keep Redis, RabbitMQ, PostgreSQL, Playwright, API and workers on Railway private networking. Only the authenticated gateway should have a public domain.

## Authentication

Self-hosted Firecrawl with `USE_DB_AUTHENTICATION=false` should not be exposed directly. Put a bearer-token gateway in front of it and keep the API private.

## ChatGPT / MCP routing

For an MCP server that should use this deployment, set `FIRECRAWL_API_URL` to the authenticated Railway gateway URL and supply its bearer key through the client's protected secret mechanism. In eTribe workflows, use the self-hosted route first and retain the hosted/client-provided Firecrawl connector as fallback.

## Updating Railway to use the fork

Once GHCR images are published, replace the official Firecrawl image references in Railway with the corresponding `ghcr.io/nishantr2/*` images. Pin a commit-SHA tag for production after validation.
