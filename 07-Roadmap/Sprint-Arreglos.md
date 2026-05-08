---
tags: [sprint, arreglos, auditoria]
sprint: arreglos
duration: 6 weeks
status: active
created: 2026-05-07
start_date: 2026-05-07
end_date: 2026-06-15
---

# Sprint Arreglos — Auditoría 2026-05-07

Sprint consolidado con TODOS los hallazgos de [[Resumen-Auditoria]]. Vivo también como sprint en producción dentro del módulo Tareas del tenant `black-wolf`.

## Métricas

- **54 tareas** distribuidas en 9 sub-categorías
- **339 horas** estimadas total
- **Prioridad**: 16 high · 25 medium · 13 low
- **Plazo**: 2026-05-07 → 2026-06-15 (6 semanas)
- **Sprint UUID en DB**: `4816529a-c02a-4e9a-87c6-341b072332a7`
- **Tenant**: black-wolf (`d7d83ca3-7e18-498d-89e1-c6da252675dc`)

## Distribución

| Sub-categoría | Tareas | Horas |
|---|---:|---:|
| `[dx]` | 10 | 129.5h |
| `[security]` | 10 | 82.5h |
| `[db]` | 7 | 13.0h |
| `[integraciones]` | 5 | 20.0h |
| `[observability]` | 5 | 13.5h |
| `[portal]` | 5 | 25.0h |
| `[cleanup]` | 4 | 4.8h |
| `[testing]` | 4 | 26.0h |
| `[whatsapp]` | 4 | 25.0h |
| **TOTAL** | **54** | **339h** |

## Acceso operativo

- **URL del sprint en Dashboard-Ops**: `central.blackwolfsec.io/black-wolf/task-management` (filtrar por sprint "Arreglos Plataforma — Auditoría 2026-05-07")
- **Pipeline**: Principal (`5a81e765-232c-4f5b-a069-4c8b4b80b1da`)
- **Stage inicial**: Por Hacer (todas)

## Convención de tags en title

Cada tarea tiene un emoji de prioridad + un tag de subcategoría:

```
🔴 [security] Rotar las 7 API keys exposed
🟠 [portal] Hito 6 — cablear contenido real en tabs
🟡 [observability] Adoptar pino + structured JSON logging
🟢 [cleanup] Mover scripts check_*.mjs fuera de src
```

- 🔴 high (crítica, hacer en 1-2 sem)
- 🟠 high (alto impacto producto)
- 🟡 medium (importante, hacer en sprint)
- 🟢 low (deuda, parking lot)

## Bloques temáticos

### 🔴 Sprint 0 Higiene — 7 tareas (10h)

Lo de [[Sprint-0-Higiene]]: rotar keys, validar JWT, configurar Stripe Hugo, RLS Fase A, validar firmas webhook, renumerar migrations, parche `bw_superadmin`. **Hacer ESTA semana.**

### 🟠 Portal Infoproducto — 5 tareas (25h)

- Mergear PRs + setear `PORTAL_JWT_SECRET` (1.5h)
- **Hito 6**: cablear contenido real en tabs (16h)
- Rate limiting, logout button, multi-owner config, cleanup OTPs

### 🔴 Seguridad estructural — 5 tareas (74h)

- Auth httpOnly cookies + CSRF (24h)
- RLS Fase B (writes anon → 12h)
- RLS Fase C (JWT claims → 24h)
- audit_logs (6h)
- Política secretos final (8h)

### 🔴 WhatsApp B.A. — 4 tareas (25h)

- RFP a 360dialog/Twilio/Meta (4h)
- Adapter abstrayente (16h)
- Detección banneo (2h)
- Encriptar sessions (3h)

### 🟡 Observability — 5 tareas (13.5h)

pino + correlation IDs + Sentry + error handler + healthcheck.

### 🟡 Testing — 4 tareas (26h)

vitest + Playwright + ESLint + CI.

### 🟡 DX cleanup — 10 tareas (129.5h)

Modularizar data.js, ClientApp, CrmPage, useL hook, CSS variables, ClientType enum, dynamic imports, zod, component library + Storybook.

### 🟡 Cleanup zombie — 4 tareas (4.8h)

ManyChat residual, Discord Node, /api/webhooks/whatsapp, scripts dispersos, tablas DB no usadas.

### 🟡 DB hardening — 7 tareas (13h)

Renumerar migrations, general_orders, documentar tablas no usadas, cleanup OTPs cron, backup pg_dump, PITR, índices.

### 🟡 Integraciones — 5 tareas (20h)

Webhook Stripe Hugo, race condition WA, webhookHandler unificado, Resend bouncing, Meta token refresh.

## Cómo usar este sprint

1. **Cada lunes**: revisar tareas en stage "En Progreso" + drag de TODO si capacidad.
2. **Por sub-categoría**: si haces 1-2 días enfocados en `[security]`, mueve esas en bloque a "En Progreso" y sale más rápido.
3. **Cierre**: cuando complete, mover a "Hecho" + dejar `feedback` en el sprint con lessons learned.
4. **Si una tarea revela trabajo nuevo**: crear tarea hija dentro del mismo sprint con prefijo del padre.

## Mapa con Sprints del Roadmap

Este Sprint Arreglos **agrega tareas operativas concretas** del [[Roadmap-Estrategico]]:

| Sprint roadmap | Tareas en este sprint |
|---|---|
| [[Sprint-0-Higiene]] | 7 (todas las 🔴 con `[security]` y `[db]`) |
| Sprint 1 (Aislamiento) | RLS Fase A (en Sprint 0), audit_logs |
| Sprint 2 (Observability) | 5 tareas `[observability]` |
| Sprint 3 (Testing) | 4 tareas `[testing]` |
| Sprint 4 (Webhooks/Secrets) | webhookHandler, política secretos |
| Sprint 5 (Multi-tenant) | RLS Fase B+C, auth httpOnly |
| Sprint 6 (WhatsApp BA) | 4 tareas `[whatsapp]` |
| Sprint 7 (DX cleanup) | 10 tareas `[dx]` |
| Sprint 8 (Component Lib) | "Component library + Storybook" |

Si un día decides expandir el plan, **cada tarea es una unidad de trabajo concreta** que se puede ejecutar.

## Cerrado 2026-05-08

### Bloque [observability] — 5/5 tareas ✅

PR `aatshadow/Dashboard-Ops-#29` · sha `3709f83` · branch `feat/sprint-arreglos-observability`.

Capa nueva en `api/_lib/`:

- `logger.js` — pino structured JSON con redact de auth/cookie/password/token. Fallback graceful a console-shim si pino no disponible.
- `request-id.js` — `ensureRequestId(req, res)` resuelve `X-Request-Id` entrante o genera UUID. Inyecta header de respuesta.
- `sentry.js` — `@sentry/node` lazy init si `SENTRY_DSN` seteado (tier free Developer, 5k errors/mes). No-op silente sin DSN.
- `error-handler.js` — `withErrorHandler(handler)` wrapper con `HttpError` class. Body uniforme `{error, requestId}`.
- `api/healthcheck.js` — `GET /api/healthcheck` con pings reales a Supabase + Stripe + Resend. Siempre 200 con `{ok, services, version, env, uptimeSec, requestId}`.

Migrado `api/asesoria-suiza-jobs.js` como demo del patrón. Doc en `docs/observability.md`.

Deps añadidas: `pino ^9.14`, `@sentry/node ^9.47`.

**Pendiente operativo:**
- Crear DSN en sentry.io plan free + setear `SENTRY_DSN` en Vercel env vars.
- (Opcional) uptime monitor externo apuntando a `/api/healthcheck`.
- Migración progresiva del resto de handlers en sprints siguientes.
- `@sentry/react` frontend con DSN distinto + ErrorBoundary global (aplazado).

### Bloque [cleanup] — 1/4 aplicada + 3/4 audit closeout

PR `aatshadow/Dashboard-Ops-#30` · sha `31eab67` · branch `feat/sprint-arreglos-cleanup`.

**Aplicada ✅:** `Mover scripts check_*.mjs fuera de src` — adaptada al estado real (no había `check_*` literal, sí 14 one-shots dispersos). Movidos a `scripts/dev/` con README explicativo. Mantenidos en raíz solo `build-tenant.mjs` y `generate-icons.mjs` (ambos cableados en `package.json`).

**Audit closeout (no aplicaban a Dashboard-Ops, marcadas done con justificación):**

- `Limpieza ManyChat residual` — ManyChat es código **activo** aquí (`api/webhook/message.js` recibe inbound, MyIntegrationsPage / IntegrationsPage / AiAgentsInstagram / AiSetterPage lo consumen). Sin residual que limpiar.
- `Eliminar bot Discord Node deprecated` — el bot vive como systemd `agente-discord.service` activo, consumido por `AiAgentsDiscord.jsx` en producción vía `VITE_API_URL`. No es deprecated.
- `Quitar /api/webhooks/whatsapp de PUBLIC_PATHS` — no existe convención `PUBLIC_PATHS` ni endpoint con esa ruta en este repo. Probablemente apunta a `enjambre-api` o auth middleware del backend.

Las 3 últimas se sugieren recategorizar hacia el repo `enjambre-api` donde sí podrían aplicar; aquí en Dashboard-Ops están cerradas como "no aplicable".

### Estado del sprint tras este bloque

| Bloque | Tareas | Estado |
|---|---:|---|
| Sprint 0 Higiene `[security]/[db]` | 7 | 5 done (otra terminal) |
| `[observability]` | 5 | **5 done** (esta sesión) |
| `[cleanup]` | 4 | **4 done** — 1 aplicada + 3 audit closeout (esta sesión) |
| Resto | 38 | todo |

## Conexiones

- Origen: [[Resumen-Auditoria]] · [[Bugs-Criticos]] · [[Deuda-Tecnica]] · [[Codigo-Zombie]]
- Sprint detalle inicial: [[Sprint-0-Higiene]]
- Roadmap completo: [[Roadmap-Estrategico]]
- Cliente owner del sprint: BlackWolf interno (slug `black-wolf`)
