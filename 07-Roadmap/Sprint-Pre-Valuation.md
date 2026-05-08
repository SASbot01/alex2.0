---
tags: [sprint, valuation, hardening, due-diligence]
sprint: pre-valuation
duration: 8 weeks
status: active
created: 2026-05-08
start_date: 2026-05-08
end_date: 2026-07-01
---

# Sprint Pre-Valuation Hardening — Subir múltiplo de mercado

Sprint dedicado a cerrar las **8 deudas técnicas que penalizan la valoración** como startup. Subset de pendientes detectados al hacer un snapshot técnico para proyección de venta / ronda.

## Métricas

- **8 tareas** distribuidas: 5 critical (`security`+`whatsapp`), 2 medium (`dx`+`quality`), 1 low (`valuation`)
- **185 horas** estimadas total (~6-8 semanas con 1 dev FT, 4-5 semanas con 2 devs en paralelo)
- **Sprint UUID en DB**: `8b3ba577-1511-4851-b7cc-903eb1c68fa3`
- **Tenant**: black-wolf (`d7d83ca3-7e18-498d-89e1-c6da252675dc`)
- **Plazo**: 2026-05-08 → 2026-07-01

## Backstory

Un snapshot rápido del producto al cierre del Sprint Arreglos detectó un valor de mercado entre **€500K – €2M** (depende de ARR real). Las 8 tareas siguientes son las que **mueven la aguja del múltiplo** en una negociación con comprador o VC.

Múltiplos cuantificados (orden de impacto):

| Mejora | Impacto en múltiplo |
|---|---|
| Backups + RLS + auth (red flag de DD) | +30-50% |
| WA Business API (desbloquea enterprise) | +20-40% |
| Métricas comerciales documentadas (ARR/churn/CAC/LTV) | +10-25% |
| Sprint Arreglos completo (descuento técnico) | +10-20% |

## Las 8 tareas (orden por prioridad)

### 🔴 1. Backups DB via SOC propio + restore test mensual (8h)

PITR de Supabase queda **OFF** — usamos el **sistema SOC propio** que ya tiene gestión de copias de seguridad.

- pg_dump diario desde SOC → AES-256 + retención 30 días + offsite secondary.
- Script `restore_to_staging.sh` aplica el dump más reciente sobre Supabase staging.
- Cron mensual SOC dispara restore + smoke test + alerta si falla.
- Runbook `/docs/disaster-recovery.md` con RTO 4h, RPO 24h.
- Badge en dashboard SOC del último backup OK/KO.

**Bloqueante de DD**: sin esto, pérdida de DB = pérdida total del negocio.

### 🔴 2. Auth httpOnly cookies + CSRF tokens (24h)

Hoy: frontend usa Supabase anon key + auth custom en localStorage (vulnerable XSS).

- Endpoint `/api/auth/login` server-side emite JWT en cookie httpOnly + sameSite=strict.
- Middleware `api/_lib/auth.js` extrae JWT → inyecta `req.user`.
- CSRF token en POST/PUT/DELETE validado en `error-handler` ya existente.
- Frontend solo llama a `/api/*` autenticado (deja de leer SUPABASE_SERVICE_KEY).
- Deprecar `memberSession.js` localStorage.

Patrón: `user_integrations.js`. **Bloqueante de RLS Fase B+C.**

### 🔴 3. RLS Fase B + Fase C — tenant isolation real (36h)

Hoy las RLS son permisivas — un cliente con anon key puede leer/escribir datos de otros tenants.

**Fase B** (writes anon bloqueados):
- Auditar 40 tablas multi-tenant (`clients`, `crm_*`, `sales`, `training_*`, `ops_*`, etc.).
- Política WRITE: anon bloqueado, authenticated solo si `client_id` matches request claim.
- Cutover por bloques con feature flag.

**Fase C** (tenant isolation con JWT claims):
- Custom claim `tenant_id` en JWT emitido por `/api/auth/login`.
- Políticas `WHERE client_id = (auth.jwt()->>tenant_id)::uuid` en cada tabla.
- Staging completo + cutover semana fin.

Pre-req: completar tarea 2 (auth httpOnly).

### 🔴 4. Rotar 7 API keys expuestas + política secretos (9h)

Auditoría 2026-05-07 detectó **7 keys leakeadas** en commits viejos.

- Listar las 7: Supabase service, Stripe Hugo, Stripe Portillo, Resend, Anthropic, Google OAuth, GitHub PAT.
- Rotar cada una en proveedor + actualizar Vercel env vars + `.env` VPS.
- Pre-commit hook con git-secrets / trufflehog.
- Política: Vercel env vars (frontend) + Doppler $7/dev/mes (backend) si crece.

### 🟠 5. Migración full a WhatsApp Business API (24h)

Hoy WA via `whatsapp-web.js` (QR + Chromium) — escala mal, riesgo banneo (incidente Creator Founder), bloqueante para enterprise.

- RFP comparativa 360dialog vs Twilio vs Meta Cloud API (pricing, latencia, multi-cuenta, plantillas).
- Adapter `src/connectors/whatsapp.js` abstrayendo ambos providers con interfaz común.
- Implementar provider WA Business API.
- Cutover por tenant con feature flag `wa_provider` en `clients` (qr|business).
- Detección banneo automática funciona con ambos.

**Beneficio valuation: +20-40% al múltiplo** (desbloquea contratos enterprise + reduce risk regulatorio Meta).

### 🟡 6. Refactor `data.js` (3649 LOC) + `ClientApp.jsx` (1573 LOC) (40h)

Archivos hipertrófiados penalizan velocidad de desarrollo y due diligence técnica.

**`data.js`**: agrupar por dominio en `src/utils/data/{auth,crm,sales,team,clients,store,pipelines,reports}.js` con barrel export para compat. Mover funciones puras sin tocar callers; tras estabilizar, deprecar barrel.

**`ClientApp.jsx`**: extraer sub-routers por área (`CrmRouter`, `SalesRouter`, `TasksRouter`, `FormacionRouter`, `OpsRouter`) en `src/routers/<area>.jsx`. ClientApp queda como shell de auth + layout.

**Beneficio**: code review +50% velocidad, onboarding devs en horas vs días.

### 🟡 7. ESLint 1006 → 0 + Coverage 30%+ — CI hard-fail (32h)

Estado actual: ~1006 ESLint errors pre-existentes con `continue-on-error: true` temporal; coverage threshold al 1% (solo `permissions.js` + `useAsync.js` testeados).

- Categorizar errors (`no-unused-vars`, `no-empty`, `no-undef`, `no-useless-escape`) y arreglar en bloques de 100. Añadir globals (`process`, `__dirname`, `__IS_TAURI__`).
- Tests unitarios para `src/hooks/{useClientData,useTickets,useOpsTickets,useClientConfig}` y `src/utils/{auth,roles,clientConfig,features}` hasta 30%.
- Subir threshold vitest a 30%.
- Quitar `continue-on-error` del job lint en `ci.yml`.

### 🟢 8. Dashboard ARR/MRR/churn/CAC/LTV + IP ownership (12h)

Lo que pide cualquier comprador o VC en la primera reunión.

**(A) Métricas comerciales:**
- Tabla `bw_metrics_monthly` con MRR, ARR, new MRR, expansion, churn, gross/net retention, customer count por mes.
- Cron `/api/cron/snapshot-metrics.js` mensual (lee Stripe + sales + clients).
- Dashboard `/black-wolf/metrics/valuation` con recharts + CSV export.

**(B) IP ownership:**
- Verificar autores commits (aatshadow, SASbot01, S4sf), consolidar via `.mailmap`.
- Mover repos a organización GitHub legal (BlackWolf SL).
- `CONTRIBUTORS.md` + `LICENSE` explícita.
- Contributor agreements para terceros (Steven, Alex Gutiérrez).

**Sin esto, comprador descuenta 20-40%.**

## Orden de ejecución sugerido

```
Semana 1: tarea 4 (rotar keys) + tarea 1 (backups SOC)        → quita los red flags inmediatos
Semana 2-3: tarea 2 (auth httpOnly) → bloquea las siguientes
Semana 4-6: tarea 3 (RLS B+C) → pre-req auth
Semana 4-6 paralelo: tarea 5 (WA Business API) → independiente
Semana 7: tarea 7 (lint+coverage) + tarea 8 (métricas) → finishing
Semana 7-8: tarea 6 (refactor data.js+ClientApp) → puede empezar antes en paralelo
```

## Conexiones

- Origen: snapshot técnico 2026-05-08 (preguntada valoración como startup)
- Sprint paralelo: [[Sprint-Arreglos]] (54 tareas, 30 done al cierre de hoy)
- Roadmap completo: [[Roadmap-Estrategico]]
- Cliente owner: BlackWolf interno (slug `black-wolf`)
- Pipeline DB: `5a81e765-232c-4f5b-a069-4c8b4b80b1da` (Principal)
- Stage inicial: `e44eb18b-…` (Por Hacer)

## Acceso operativo

URL del sprint en producción:
`central.blackwolfsec.io/black-wolf/task-management` → filtrar sprint "Pre-Valuation Hardening".

## Definition of Done del sprint

- [ ] Backups DB documentados, último restore test < 30 días en verde
- [ ] Auth httpOnly + CSRF en producción para todos los tenants
- [ ] RLS Fase B+C activas, audit trail de un test de pen-test simple
- [ ] 7 API keys rotadas + git-secrets activo en pre-commit
- [ ] Al menos 1 tenant migrado a WA Business API en staging
- [ ] `data.js` < 1500 LOC, `ClientApp.jsx` < 600 LOC
- [ ] ESLint errors = 0, coverage > 30%, CI hard-fail
- [ ] Dashboard `/metrics/valuation` accesible, datos reales últimos 6 meses
- [ ] Repos transferidos a org GitHub legal + LICENSE + CONTRIBUTORS

**Cuando los 9 checkboxes están en verde, el producto está pre-DD-ready y el múltiplo proyectado sube +30-60% sobre el baseline actual.**
