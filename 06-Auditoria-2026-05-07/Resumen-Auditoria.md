---
tags: [auditoria, critical]
created: 2026-05-07
audit_date: 2026-05-07
---

# Resumen — Auditoría 2026-05-07

Estudio integral post-Hito 5 del [[Portal-Infoproducto]]. 4 sub-auditorías paralelas: frontend, backend, DB, integraciones.

## Una frase

Tienes un SaaS multi-tenant funcional con 16 clientes en producción, **arquitecturalmente en el punto donde añadir clientes nuevos empieza a doler**. Lo que falta para "vendible a empresa con CISO" es: aislamiento real, observability, política de secretos, testing y separación entre código vivo y zombie.

## Métricas duras

- **Frontend Dashboard-Ops**: 117K LOC · 210 páginas · 48 componentes shared · 1573 LOC en ClientApp.jsx · 3649 LOC en data.js · 4524 LOC en CrmPage.jsx
- **Backend enjambre-api**: ~7K LOC · 50+ endpoints (88% protegidos JWT, 12% públicos) · 1427 LOC en whatsapp.js
- **DB Supabase**: 166 tablas (103 con `client_id`, 63 globales) · 91 FKs a `clients(id)` · 127 con RLS pero **PERMISIVAS (allow all)** · 115 columnas JSONB sin validación
- **Integraciones**: 12 — Stripe, Resend, WhatsApp, Google, Meta, Anthropic, Supabase, Cloudflare, Discord (live), Hotmart/Fathom (pending), ManyChat (zombie)

## 9 bugs críticos sangrando

Ver [[Bugs-Criticos]] para detalle:

1. API keys exposed (Stripe, Resend, Anthropic, Supabase) — todavía no rotadas pese a comments admiten exposure
2. Webhook Stripe Hugo NO configurado → revenue no sincroniza a CRM
3. `/api/webhooks/central` sin firma → inyección arbitraria
4. `JWT_SECRET = 'default-secret'` fallback → tokens falsificables
5. `localStorage('bw_superadmin', 'true')` desde DevTools = admin instantáneo
6. RLS = "ALLOW ALL" en 127 tablas → multi-tenant solo en frontend
7. Migrations 045-048 con números duplicados → orden indeterminado
8. WhatsApp `findOrCreateCrmContact` race condition → contactos duplicados
9. WhatsApp via whatsapp-web.js sin plan B (banneo 26-abr fue aviso)

## Top deuda técnica

Ver [[Deuda-Tecnica]]:

- 3 god files: ClientApp.jsx + data.js + CrmPage.jsx = 9700 LOC mezclados
- Función `L()` para i18n duplicada **41 veces**
- 162 referencias a localStorage (auth débil)
- 2642 colores hex hardcoded en `pages/`
- 0 tests, 0 linter, 0 TypeScript
- Sin structured logging, sin request_id, sin observability

## Top código zombie

Ver [[Codigo-Zombie]]:

- ManyChat: webhook + helpers + tabla siguen vivos pese a "retiro"
- Discord Node bot deprecado pero `discord.js` sigue como dependency
- 16 tablas DB no usadas
- Scripts `check_*.mjs` mezclados con prod

## Fortalezas reales

Ver [[Fortalezas]]:

- Stack moderno (React 19 + Vite 7 + Fastify 5)
- 88% backend protegido con JWT
- Prompt caching consistente en orchestrator + commanders
- AI cost tracking en `brain_decisions`
- Stripe webhook handler robusto (HMAC + anti-replay + dedup)
- Google OAuth bien implementado
- 91 FKs con CASCADE → no orphan data
- Multi-tenant scoping intencional (103/166 tablas)

## Roadmap propuesto

Ver [[Roadmap-Estrategico]] · 9 sprints · 5-6 meses:

0. [[Sprint-0-Higiene]] — rotar keys, validar JWT, Stripe Hugo (1 sem)
1. [[Sprint-1-Aislamiento]] — RLS Fase A + audit_logs (2 sem)
2. [[Sprint-2-Observability]] — pino + Sentry (2 sem)
3. [[Sprint-3-Testing]] — vitest + Playwright (3 sem)
4. [[Sprint-4-Webhooks-Secrets]] — webhookHandler + Vault (2 sem)
5. [[Sprint-5-Multi-Tenant]] — RLS B+C (3 sem)
6. [[Sprint-6-WhatsApp-BA]] — migración API oficial (4-6 sem)
7. [[Sprint-7-DX-Cleanup]] — modularizar god files (2 sem)
8. [[Sprint-8-Component-Library]] — UI lib + Storybook (3 sem)
