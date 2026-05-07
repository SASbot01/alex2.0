---
tags: [auditoria, fortalezas]
created: 2026-05-07
---

# Fortalezas Reales

Lo que SÍ está bien. Vale la pena mantenerlo y construir sobre ello.

## Stack moderno y al día

- **React 19** + **Vite 7** + **react-router 7** (frontend)
- **Fastify 5** + **Node 18** + **Anthropic SDK** (backend)
- **Supabase** (DB + Storage + Auth)
- Cero deuda de versiones mayores. Ningún stack obsoleto.

## 88% endpoints backend protegidos con JWT

Solo ~6 endpoints son verdaderamente públicos (health, login, webhooks). El resto (45+) van por `authHook` con Bearer token. La postura defensiva base es correcta — los huecos son específicos y arreglables.

## Lazy loading aplicado en frontend

120 lazy-loads en ClientApp.jsx. El bundle inicial es razonable. Solo XLSX (400KB) y heic2any tienen oportunidad de migrar a dynamic import.

## Permissions matriz bien diseñada

`src/utils/permissions.js` es 83 líneas con matriz central, helpers `can()`, `canAny()`, `canAll()`, `firstDenied()`. 40+ permisos definidos. **Está bien escrito**, solo está infrautilizado (7 usos vs 300+ rutas).

## Prompt caching consistente en capa cognitiva

- `orchestrator.js`: `cache_control: { type: 'ephemeral' }` en system + tools
- `BaseCommander`: `_cachedSystem()`, `_cachedTools()`
- AI tracker reconoce cache_read y cache_create tokens en pricing

~70% de las llamadas a Claude están cacheadas. Esto te ahorra mucho dinero.

## AI cost tracking propio

`brain_decisions.decision_type = 'ai_usage'` registra cada llamada con agent, model, tokens, cost_usd. Endpoint `/api/ai-usage` agrega por agent/model/día. **Sabes cuánto gasta cada agente** sin tener que mirar el dashboard de Anthropic.

## Stripe webhook handler robusto

`Dashboard-Ops-/api/webhook/stripe.js` es probablemente el endpoint mejor escrito del repo:

- Valida firma HMAC-SHA256 ✓
- Anti-replay: rechaza eventos > 5 min ✓
- Crea/vincula contacto CRM ✓
- Registra venta con dedup por `session_id` ✓
- Auditoría: `writeAudit()` en 3 rutas críticas ✓

## Google OAuth bien implementado

`Dashboard-Ops-/api/google-calendar.js`:
- OAuth 2.0 con refresh token ✓
- Auto-refresh si `expires_at < now` ✓
- Scopes mínimos (`calendar`, `gmail.readonly`) ✓
- UX amigable para errores (redirige con `?google=error&reason=...`) ✓
- Multi-account + per-member ✓

## 91 FKs a `clients(id)` con CASCADE

Cuando borras un cliente, **no quedan datos huérfanos**. La integridad referencial está intencionalmente diseñada. 103 de 166 tablas tienen `client_id` con FK CASCADE.

## Multi-tenant scoping intencional

103 de 166 tablas (62%) tienen `client_id`. La intención de aislamiento existe en el schema. Falta solo enforcement (RLS Fase B+C — ver [[Sprint-5-Multi-Tenant]]) para convertir intención en boundary real.

## Army arquitectura sana

`enjambre-api/src/army/`:
- `SOUL.md` documenta la lógica
- Doctrines en markdown (`crm.md`, `ops.md`, `dev.md`, `forms.md`, `general.md`)
- BaseCommander con prompt caching
- General + CRM commander operativos
- Dry-run mode (`ARMY_DRY_RUN=1`) para testing

Solo falta completar handlers OPS/DEV/FORMS y conectar Donna como executor.

## Cloudflare Tunnel + cert managed

Edge layer limpia. Cero gestión manual de TLS. `cloudflared-enjambre.service` activo. Subdomains separados para frontend (`central`), backend (`enjambre`), SOC (`soc`).

## Multi-language no hardcoded en backend

El `clients.config.language` define `es`/`en`/`zh`. Tenants `yc-logistics` (zh+en) y `asesoria-suiza` (es+en) ya operan multi-idioma. El sistema soporta más idiomas fácilmente, solo hay que limpiar el patrón inline en frontend.

## Dashboard-Ops cubre MUCHO terreno

CRM, ventas, agendas, marketing, formación (cursos + comunidad + mentorías + marketplace), tickets, IA agents (setter, outreach, email, brain), manufacturing, logistics, FBA stores, info-productos, comisiones, contabilidad, legal, integraciones... La superficie de producto es enorme y cubre verticales muy distintas. Esto es valor estratégico real.

## Conexiones

- Lo que cuesta: [[Deuda-Tecnica]] · [[Bugs-Criticos]]
- Lo que falta: [[Flaquezas-Estructurales]]
- Cómo construir encima: [[Roadmap-Estrategico]]
