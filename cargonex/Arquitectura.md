---
tags: [plataforma, cargonex, arquitectura]
created: 2026-05-07
---

# Cargonex — Arquitectura

Detalle técnico de la plataforma [[Cargonex]]. Cómo se sirve, cómo se aísla, dónde vive el dato.

## Diagrama de servicios

```
                  ┌────────────────────────────────────────────┐
                  │  Cloudflare Tunnel                         │
                  │   app.cargonex.co  →  eilers-portal:3000   │
                  │   web.cargonex.co  →  eilers-portal:3000   │
                  └─────────────────┬──────────────────────────┘
                                    ▼
            ┌─────────────────────────────────────────────────┐
            │  eilers-portal  (Express, Node 20)              │
            │                                                 │
            │   /landing/   →  landing/index.html  (público)  │
            │   /admin      →  cargonex-admin.html (Blackwolf)│
            │   /portal     →  portal.html         (cliente)  │
            │   /<Slug>     →  public/index.html + ctx        │
            │   /api/*      →  handlers (Express)             │
            └────────────┬────────────────┬───────────────────┘
                         │                │
                         ▼                ▼
            ┌─────────────────────┐   ┌────────────────────────┐
            │  /data volume       │   │  eilers-postgres       │
            │  (JSON files)       │   │  (kv_store + photos)   │
            │                     │   └────────────────────────┘
            │  db.json (Eilers)   │              │
            │  cargonex-          │              ▼
            │   platform.json     │   ┌────────────────────────┐
            │  tenant-<id>.json   │   │  eilers-backup         │
            │  photos/            │   │  pg_dump → /backups    │
            │  push-subs.json     │   │  (+ MinIO opcional)    │
            │  locations.json     │   └────────────────────────┘
            └─────────────────────┘
```

Todo en un VPS. Sin puertos públicos: el único ingress es el túnel Cloudflare. Compose-stack también aloja `capitalhub` y `blackwolf-soc` — ojo a [[Host-Docker-Layout]].

## Slug routing — qué pasa en `/<Slug>`

Implementado en `server/server.js` (`app.get('/:slug', …)`):

```
GET /EilersLogistik
  ↓
1. Lookup en cargonex-platform.json#tenants donde slug=='EilersLogistik'
  ↓
2. Match status:
     • status=='pending'  → res.redirect(302, '/portal')   ← key sin activar
     • status=='revoked'  → 403 con página friendly        ← cliente cancelado
     • status=='active'   → continuar
  ↓
3. Leer public/index.html, inyectar en <head>:
     <meta name="cargonex-client-id" content="cli_…">
     <meta name="cargonex-slug"      content="EilersLogistik">
     <script>window.__CARGONEX_TENANT__ = { clientId, legacy };</script>
  ↓
4. res.send(html)
```

El SPA (`public/index.html`) lee `window.__CARGONEX_TENANT__` y bifurca:

- `legacy:true` (solo Eilers) → calls a `/api/logs` (sin header de cliente).
- `legacy:false` → calls a `/api/tenant-logs` con header `X-Client-ID: cli_…`.

`authHeaders()` arrastra el clientId en cada request autenticado.

## Las dos onboarding (flujo técnico)

### Flow 1 — Lead capture (marketing → admin)

```
Browser (web.cargonex.co)
  ↓ POST /api/onboarding-request { name, email, company, drivers, ... }
handleOnboardingRequest (server/server.js:673)
  ↓ append to db.json#onboardingRequests
  ↓ source: "form" | "smoke-test" | …
visible en /admin → Leads (action=list-leads)
```

No crea tenant. Solo lead. Triaje manual desde `/admin`.

### Flow 2 — PI-key issue + activation

```
1. Admin
   /admin → Clients → + New client
   POST /api/platform-admin {
     action: "create-client",
     companyName, contactEmail, plan, slug
   }
   X-Admin-Secret: $PLATFORM_ADMIN_SECRET
     ↓
   server/cargonex-platform.js handleCreateClient()
     ↓
   { clientId: "cli_xyz", key: "CX-XXXXXXXX-XXXX", slug, status: "pending" }

2. Admin envía key al cliente por email/WhatsApp/lo que sea

3. Cliente
   /portal → click "Erstes Mal? PI-Schlüssel verwenden"
   POST /api/platform-login {
     action: "activate-client",
     key: "CX-XXXXXXXX-XXXX"
   }
     ↓
   handleActivateClient():
     - Match key contra cargonex-platform.json#keys
     - Flip tenant.status: pending → active
     - tenant.activatedAt = now
     - Crear data/tenant-<clientId>.json (vacío inicial)
     - Audit entry
     ↓
   { ok: true, slug, redirectTo: "/<Slug>" }

4. Cliente redirected a /<Slug> → SPA arranca
```

## Data layout

```
data/
├── db.json                    Eilers Logistik (legacy)
│                                drivers[], vehicles[], records[], …
│                                onboardingRequests[]  ← marketing leads
├── cargonex-platform.json     Registry global
│                                tenants[], keys[], audit[]
├── tenant-<clientId>.json     1 fichero por tenant non-legacy
│                                misma forma que db.json pero aislado
├── push-subs.json             VAPID subs
├── locations.json             positions live (TTL 10 min)
└── photos/
    └── {vehicleId}.json       Übergabe photos en base64
```

Backup = volumen `eilers-data` completo. Postgres separado en `eilers-pgdata` (KV + photos).

## API endpoints

### Públicos

| Endpoint | Notas |
|---|---|
| `GET /api/health` | Liveness probe (usado por compose healthcheck) |
| `POST /api/onboarding-request` | Marketing form |
| `GET /api/cargonex-config` | Config pública (VAPID public key, etc.) |
| `GET /api/tenants-public` | Slug → display name (para SEO/portal) |

### Tenant data — requiere `X-Client-ID`

| Endpoint | Notas |
|---|---|
| `/api/logs` (GET, POST) | **Solo Eilers** (legacy). Lee/escribe `db.json` |
| `/api/tenant-logs` (GET, POST) | Multi-tenant. Lee/escribe `tenant-<id>.json` |
| `/api/ub-photos` | Übergabe photos (Postgres-backed) |
| `/api/location` | Live positions |
| `/api/push-subscribe` · `/api/push-send` | VAPID web-push |

### Admin — requiere `X-Admin-Secret`

`POST /api/platform-admin` con body `{ action, ...params }`. Acciones definidas en `server/cargonex-platform.js`:

- Tenants: `list-clients`, `list-clients-with-stats`, `create-client`, `revoke-client`, `get-db`
- Leads: `list-leads`, `update-lead-status`
- Keys: `list-keys`
- Audit: `audit-log`

### Auth — cliente-facing

`POST /api/platform-login` con body `{ action, ...params }`:

- `login` (email + password)
- `activate-client` (PI-key)
- `request-otp` · `verify-otp` (email-OTP)

## Service worker

`public/sw.js` precachea el app shell. Cuando `index.html` / `cargonex-admin.html` / `portal.html` cambian sustancialmente, **bumpear `SW_VERSION`** (`v1.2.0` actual, 2026-05-07). Sin bump, los clientes existentes siguen viendo la versión vieja en cache hasta que hard-refreshen.

## Variables de entorno

Listado completo en el README del repo. Las críticas:

| Var | Notas |
|---|---|
| `POSTGRES_PASSWORD` | required, sin esto compose falla |
| `PLATFORM_ADMIN_SECRET` | gate del `/admin` — **rotar de `testtoken123` ya** |
| `ADMIN_TOKEN` | gate de `/api/import-db` |
| `TUNNEL_TOKEN` | Cloudflare tunnel |
| `VAPID_PUBLIC_KEY` / `VAPID_PRIVATE_KEY` | web-push |
| `SMTP_*` | transaccional + OTP (Resend en prod) |
| `ANTHROPIC_API_KEY` | tour planner |

`docker-compose.yml` debe forwardear cada una bajo `eilers-portal.environment:`. Bug histórico (2026-05-07): `PLATFORM_ADMIN_SECRET` estaba en `.env` pero faltaba en compose → `/admin` daba 503. Fix en commit `48fc42b`.

## Conexiones

- Producto: [[Cargonex]]
- Tenant fundador: [[Eilers-Logistik]]
- Stack separado de [[Dashboard-Ops]] — razón: [[ADR-Cargonex-Stack-Separado]]
- Compartiendo VPS: [[Host-Docker-Layout]]
- Onboarding tenant: [[Playbook-Crear-Tenant-Cargonex]]
