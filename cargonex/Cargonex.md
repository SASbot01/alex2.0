---
tags: [plataforma, cargonex, saas, multi-tenant]
status: live
created: 2026-05-07
---

# Cargonex

Segunda línea de producto SaaS de BlackWolfSec. **Cargonex** es una plataforma multi-tenant white-label para gestión de flotas de logística. Stack y deploy completamente separados de [[Dashboard-Ops]] — otro repo, otro VPS, otro dominio, otra base de datos. Cliente fundador en producción: [[Eilers-Logistik]].

> Si [[Dashboard-Ops]] es el cerebro de los clientes growth/consultoría/infoproducto de BlackWolf, **Cargonex es el cerebro de los clientes logística europeos**. No comparten código.

## URLs en producción

| URL | Para quién | Backed by |
|---|---|---|
| `https://web.cargonex.co/` | Prospects (público) | `landing/index.html` |
| `https://app.cargonex.co/admin` | Equipo BlackWolf (interno) | `public/cargonex-admin.html` |
| `https://app.cargonex.co/portal` | Cliente — login + activación PI-key | `public/portal.html` |
| `https://app.cargonex.co/<Slug>` | Cliente activado — su SPA aislada | `public/index.html` + tenant ctx |
| `https://app.cargonex.co/landing/` | Alias del landing | `landing/index.html` |

Ejemplo real: `https://app.cargonex.co/EilersLogistik` aterriza en la SPA de Eilers con su `clientId` inyectado en `<head>`.

## Stack

| Capa | Tech |
|---|---|
| Backend | Node 20 + Express ESM |
| Frontend | Vanilla JS PWA (no framework) |
| Storage | Postgres 16 (KV + photos) + JSON files en volumen `/data` |
| Email | SMTP / Resend (transaccional + OTP) |
| AI | Anthropic SDK (`claude-haiku-4-5`) — tour planner, damage triage |
| Edge | Cloudflare Tunnel — sin puertos públicos en el VPS |
| Backups | `pg_dump` cron en contenedor `eilers-backup` (+ MinIO opcional) |
| Auth | Email-OTP (default) o email+password (opcional) |

## Repo + deploy

- **Repo**: <https://github.com/SASbot01/eilerrs-portal> (rama `main`)
- **VPS**: el mismo donde corren `capitalhub` y `blackwolf-soc`. Cuidado con [[Host-Docker-Layout]] (puertos/volúmenes/redes no colisionar).
- **Compose**: 4 containers — `eilers-portal`, `eilers-postgres`, `eilers-tunnel`, `eilers-backup`.
- **Update**: `git pull && docker compose up -d --build eilers-portal`. Volumen `eilers-data` no se toca en rebuild.

## Multi-tenant en Cargonex

Cada tenant se aísla en un fichero `data/tenant-<clientId>.json`. Excepción: Eilers es `legacy:true` y vive en `data/db.json` por compatibilidad histórica (8000+ records).

El registry global está en `data/cargonex-platform.json`:

```json
{
  "tenants": [ { "clientId":"cli_…", "slug":"…", "status":"pending|active|revoked", "legacy":bool, … } ],
  "keys":    [ { "clientId":"cli_…", "key":"CX-XXXXXXXX-XXXX", "issuedAt":"…", "activatedAt":"…" } ],
  "audit":   [ { "ts":"…", "action":"…", "clientId":"…", "ip":"…", "detail":"…" } ]
}
```

Detalle completo en [[Arquitectura]].

## Las dos onboarding

Cargonex tiene DOS flujos de "onboarding" que NO son lo mismo:

1. **Lead capture** (marketing → admin): form en `web.cargonex.co` → `db.json#onboardingRequests` → visible en `/admin → Leads`. No crea tenant.
2. **Tenant activation** (PI-key): admin emite `CX-XXXX-XXXX` → cliente lo activa en `/portal` → tenant pasa de `pending` → `active`. Crea tenant aislado.

Procedimiento operativo en [[Playbook-Crear-Tenant-Cargonex]].

## /admin — qué se hace ahí

Login con `PLATFORM_ADMIN_SECRET`. Una sección por función:

| Section | Action backend | Para qué |
|---|---|---|
| Dashboard | `list-clients-with-stats` | KPIs agregados (Fahrer/Fahrzeuge totales) + tabla per-tenant |
| Clients | `list-clients` · `create-client` · `revoke-client` | CRUD de tenants. Aquí se emite el PI-key |
| Leads | `list-leads` · `update-lead-status` | Triaje de marketing leads |
| License Keys | `list-keys` | Inventario de PI-keys emitidos |
| Health | `list-clients-with-stats` | `lastSeen` per-tenant (live/stale/dead) |
| Audit Log | `audit-log` | Cada acción del admin (timestamp · IP · detail) |
| DB Inspector | `get-db` | Lectura read-only del data file de cualquier tenant |
| API Console | (cualquiera) | Sandbox JSON request/response |

Branding: rediseñado 2026-05-07 para alinear con `web.cargonex.co` y la SPA — Inter / Space Grotesk / JetBrains Mono, paleta `#0b0d12`, mark triángulo+chevron Cargonex. Detalle: commit `50833ec`.

## Estado (2026-05-07)

✅ **Live en producción**:
- Marketing landing (`web.cargonex.co`) sirviendo formulario de leads
- Portal con login email+password + activación PI-key
- Slug routing operativo (`/EilersLogistik` resuelve correctamente)
- Admin con datos reales (Eilers: 231 Fahrer / 162 Fahrzeuge / 8343 records)
- Multi-tenant backend (`/api/tenant-logs`, `/api/platform-admin`, `/api/platform-login`)
- Cloudflare Tunnel + Postgres + backups operativos
- Container healthy: `eilers-portal:latest`

🟡 **Pendiente go-live full**:
- **`PLATFORM_ADMIN_SECRET = "testtoken123"`** en producción — placeholder de dev. Rotar YA. Ver [[Playbook-Rotar-Secret]] · [[Bugs-Criticos]].
- SMTP cutover a Resend en progreso (verificación de dominio `cargonex.co`)
- Mobile PIN flow no probado en device real (iOS/Android safari)
- DSGVO/GDPR docs pendientes (cliente alemán, exigencia legal)

🔴 **Sangrando**:
- El token GitHub PAT que se usó para pushear el repo el 2026-05-07 se compartió en chat plano — **rotar**. Ver [[Politica-de-Secretos]].
- No hay billing del SaaS en sí. Eilers paga por contrato anual offline. Si se quiere escalar, meter Stripe Connect (paralelo al patrón [[Stripe]] de Hugo).

## Conexiones

- Arquitectura técnica: [[Arquitectura]]
- Cliente fundador: [[Eilers-Logistik]]
- Onboarding nuevo tenant: [[Playbook-Crear-Tenant-Cargonex]]
- Por qué stack separado de Dashboard-Ops: [[ADR-Cargonex-Stack-Separado]]
- Riesgo de secret expuesto: [[Politica-de-Secretos]] · [[Playbook-Rotar-Secret]]
- Dónde encaja en el negocio: [[Modelo-de-Negocio]]
- Hardware compartido: [[Host-Docker-Layout]] (no colisionar con capitalhub/blackwolf-soc)
