---
tags: [moc, roadmap]
created: 2026-05-07
---

# MOC — Roadmap

Del MVP avanzado actual al "SaaS empresarial vendible con compliance".

## Estrategia

- [[Roadmap-Estrategico]] — visión 6-10 meses, qué cambia y por qué

## Sprints

| # | Sprint | Foco | Duración | Estado |
|---|---|---|---|---|
| 0 | [[Sprint-0-Higiene]] | Rotar keys, validar JWT, configurar Stripe Hugo | 1 semana | 🔵 prio MAX |
| 1 | [[Sprint-1-Aislamiento]] | Migration 018 RLS Fase A · audit_logs | 2 semanas | pendiente |
| 2 | [[Sprint-2-Observability]] | pino + correlation IDs + Sentry | 2 semanas | pendiente |
| 3 | [[Sprint-3-Testing]] | vitest + Playwright | 3 semanas | pendiente |
| 4 | [[Sprint-4-Webhooks-Secrets]] | webhookHandler unificado + Vault | 2 semanas | pendiente |
| 5 | [[Sprint-5-Multi-Tenant]] | RLS Fase B+C con JWT claims | 3 semanas | pendiente |
| 6 | [[Sprint-6-WhatsApp-BA]] | Migración a WhatsApp Business API | 4-6 semanas | pendiente |
| 7 | [[Sprint-7-DX-Cleanup]] | Modularizar god files | 2 semanas | pendiente |
| 8 | [[Sprint-8-Component-Library]] | Lib UI + Storybook | 3 semanas | pendiente |

## Sprints operativos vivos

| Sprint | Tareas | Horas | Foco | UUID |
|---|---:|---:|---|---|
| [[Sprint-Setter-WhatsApp]] | 25 | 72.5h | Setter listo lunes 2026-05-11 | `523b8b96-5d96-4200-94ce-639a4a0658c1` |
| [[Sprint-Desktop-App]] | 23 | 74.5h | Dashboard-Ops nativa Mac+Win (Tauri, Opción B per-tenant, signing diferido) | `0e26d64e-dd2d-46d5-8278-e78318fba0a0` |
| [[Sprint-Arreglos]] | 54 | 339h | Hallazgos auditoría plataforma | `4816529a-c02a-4e9a-87c6-341b072332a7` |

Todos en módulo Tareas del tenant `black-wolf`: `central.blackwolfsec.io/black-wolf/task-management`.

## En curso

- [[Portal-Infoproducto]] — Hitos 1-5 mergeables · pendiente Hito 6 (cableado contenido real)

## Ideas / parking lot

- Migrar auth a httpOnly cookies + CSRF
- Centralizar config tenant (4 fuentes hoy)
- Component library + Storybook
- TypeScript gradual (al menos JSDoc tipos)
- AI gateway propio (Vercel AI Gateway o similar)

## Conexiones

- De dónde venimos: [[_moc/MOC-Auditoria]]
- Decisiones por tomar: [[_moc/MOC-Decisiones-CTO]]
