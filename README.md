# invites-blog-deploy

Production deployment for **invites.blog** — a premium animated invitation platform.

Companion repos (clone as siblings next to this one):
- [`invites-blog-backend`](https://github.com/Zouriel/invites-blog-backend) — ASP.NET Core / .NET 10 API (layered controllers/services/repositories, full RBAC)
- [`invites-blog-frontend`](https://github.com/Zouriel/invites-blog-frontend) — Angular 22 workspace: `web-inviter` (invites.blog) + `web-invitee` (me.invites.blog) + the `ui` component library

## Topology

The shared host runs a Caddy reverse-proxy container (`/opt/stacks/proxy`) that fronts many projects
on the external `proxy` Docker network. This stack adds, without touching anything else:

| Container | Role | Network(s) | Host ports |
|---|---|---|---|
| `invites-blog-postgres` | PostgreSQL 17 | `invites-internal` | none (internal) |
| `invites-blog-api` | .NET 10 API + `/assets` | `invites-internal`, `proxy` | none |
| `invites-blog-web-inviter` | Angular SPA (nginx) | `proxy` | none |
| `invites-blog-web-invitee` | Angular SPA (nginx) | `proxy` | none |

Caddy routes (see `Caddyfile.d/invites-blog.caddy`) — each domain proxies `/api/*` and `/assets/*`
to the API (same-origin, no CORS) and everything else to its SPA:
- `invites.blog` → web-inviter
- `me.invites.blog` → web-invitee
- `invites.164.68.102.231.sslip.io` / `me-invites.164.68.102.231.sslip.io` → immediate-access fallback

## Deploy

```bash
cd /opt/apps
git clone git@github.com:Zouriel/invites-blog-backend.git
git clone git@github.com:Zouriel/invites-blog-frontend.git
git clone git@github.com:Zouriel/invites-blog-deploy.git
cd invites-blog-deploy
cp .env.example .env && $EDITOR .env         # set strong secrets

docker compose -f compose.prod.yml up -d --build

# add the Caddy route (additive — does not touch other sites) and reload gracefully
cp Caddyfile.d/invites-blog.caddy /opt/stacks/proxy/Caddyfile.d/
docker exec caddy caddy reload --config /etc/caddy/Caddyfile
```

Update: `git -C ../invites-blog-backend pull && git -C ../invites-blog-frontend pull && docker compose -f compose.prod.yml up -d --build`.

## DNS

Point `invites.blog`, `me.invites.blog` (and optionally `api.invites.blog`, `assets.invites.blog`)
A records at `164.68.102.231`. If proxied through Cloudflare, use **Full** SSL and ensure the origin
is this server so Caddy can serve/validate. The `*.sslip.io` hosts work immediately with no DNS change.

The API applies EF migrations and seeds templates + RBAC (roles/permissions + the admin account from
`.env`) on first start.
