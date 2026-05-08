---
tags: [plataforma, arquitectura]
created: 2026-05-07
---

# Arquitectura General

## Componentes en producción

```
                     ┌─────────────────────────┐
                     │    Alejandro / Equipo   │
                     └──────────┬──────────────┘
                                │ Telegram
                                ▼
                     ┌─────────────────────────┐
                     │       Donna             │ ← Asistente ejecutiva
                     │  /opt/ai-agent-console  │
                     └──────────┬──────────────┘
                                │ HTTP (DONA_PEER_TOKEN)
                                ▼
   ┌────────────────────────────────────────────────────┐
   │  Enjambre-API (Fastify · Node 18 · Anthropic SDK) │
   │  ├─ Routes: /api/* (50+ endpoints)                │
   │  ├─ Connectors: WhatsApp, ManyChat (zombie),      │
   │  │  Resend, DCC, Discord, ElevenLabs              │
   │  ├─ Army: General + CRM commander (operativos),   │
   │  │  OPS/DEV/FORMS (doctrina sin código)           │
   │  └─ Workers: sequence-worker, brain-worker        │
   │                                                    │
   │  Hosted: este server, port 3500                   │
   │  Edge:   Cloudflare Tunnel → enjambre.blackwolf…  │
   └────────────────┬─────────────────┬─────────────────┘
                    │                 │
        Service key │                 │ Webhooks
                    ▼                 ▼
   ┌─────────────────────────────────────────────────────┐
   │  Supabase (Postgres + Storage + Auth)              │
   │  Project: kyxupfowsfkyqklxhtyo                     │
   │  ├─ 166 tablas (103 con client_id)                 │
   │  ├─ RLS habilitada en 127 (PERMISSIVE allow-all)   │
   │  └─ Storage bucket: crm-files                      │
   └────────────────────────┬────────────────────────────┘
                            │ anon key (frontend) +
                            │ service key (backend)
                            ▼
   ┌─────────────────────────────────────────────────────┐
   │  Dashboard-Ops (React 19 + Vite 7)                 │
   │  ├─ ClientApp.jsx (1573 LOC, 336 rutas)            │
   │  ├─ Multi-tenant via /:clientSlug/*                │
   │  └─ Auth custom (tabla team) + Supabase anon       │
   │                                                     │
   │  Hosted: Vercel auto-deploy desde main             │
   │  URL:    central.blackwolfsec.io                   │
   └────────────────────────┬────────────────────────────┘
                            │
                            ▼
   ┌─────────────────────────────────────────────────────┐
   │  Usuarios finales                                   │
   │  ├─ Team (CEO, directors, setters) ← /<slug>/*     │
   │  ├─ Store clients (FBA Academy)                    │
   │  └─ Alumnos infoproducto ← /portal/<slug>          │
   └─────────────────────────────────────────────────────┘
```

## Rutas principales (frontend)

```
/                           ApexLanding (público)
/solutions, /pricing        Marketing (público)
/landing*                   Landings específicas
/admin/*                    AdminApp (super-admin BlackWolf)
/erp/:companySlug/*         ErpApp (separate sub-router)
/:clientSlug/agendas[/...]  BookPublic (booking branded)
/portal/:clientSlug[/hub]   PortalLogin/PortalHub (alumnos)
/onboarding                 Onboarding nuevos team users
/signup                     Self-service signup BlackWolfSec
/:clientSlug/*              ClientApp (catch-all multi-tenant)
```

## Flujos críticos

### Flujo: lead nuevo desde landing
```
Landing pública del cliente
  → POST /api/forms/<slug>-submit (enjambre-api)
  → INSERT crm_contacts (client_id del slug)
  → eventBus 'lead.created'
  → Worker enrolls en sequence si aplica
```

### Flujo: venta Stripe
```
Alumno paga en Stripe
  → Webhook POST /api/webhook/stripe (Dashboard-Ops API route)
  → Valida HMAC + anti-replay
  → INSERT/UPDATE crm_contacts + sales
  → writeAudit
```

### Flujo: WhatsApp inbound
```
Alumno escribe WhatsApp
  → whatsapp-web.js (sesión del tenant)
  → findOrCreateCrmContact (race condition pendiente fix)
  → INSERT crm_messages
  → (si setter AI activo) → orchestrator.js → respuesta automática
```

### Flujo: Donna decide algo
```
Alex escribe Telegram
  → Donna recibe → llama POST /order de Dona General
  → Dona planifica con Claude Sonnet
  → Asigna a CRM commander
  → CRM commander tira tools, escribe brain_decisions
  → Reporta back vía eventBus
```

## Lo que NO hay (gaps estructurales)

- **API gateway** entre frontend y enjambre-api → cada call es directa
- **Observability layer** → logs efímeros, sin tracing
- **Secret manager** → todo en `.env` que circula
- **Multi-tenant boundary** → solo voluntario en frontend
- **Test runner** → cada deploy es ruleta rusa

Ver [[Flaquezas-Estructurales]] y [[Roadmap-Estrategico]].

## Conexiones

- Backend detalle: [[Enjambre-API]]
- Frontend detalle: [[Dashboard-Ops]]
- Capa cognitiva: [[Donna-y-Army]]
- DB: [[Database-Supabase]]
