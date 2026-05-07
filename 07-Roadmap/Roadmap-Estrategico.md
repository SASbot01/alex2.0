---
tags: [roadmap, estrategia]
created: 2026-05-07
---

# Roadmap Estratégico

De "MVP avanzado funcional" a "SaaS empresarial vendible con compliance". 5-6 meses de trabajo de un dev senior dedicado, o 8-10 meses con rotación parcial.

## Visión

Hoy tienes una plataforma operativa con 16 clientes vivos. Hace lo que tiene que hacer pero **arquitecturalmente está en un punto donde añadir clientes nuevos empieza a doler exponencialmente**. Cada cliente nuevo añade entropy: hardcoded checks por slug, branches en god files, integraciones nuevas sin abstracción.

El roadmap empuja a la plataforma hacia 4 propiedades:

1. **Aislamiento real** — multi-tenant enforced en boundary, no voluntario.
2. **Visibilidad** — saber qué pasa, cuándo, por qué, en cualquier momento.
3. **Reproducibilidad** — testing + tipos + linting hacen que cambios sean predecibles.
4. **Mantenibilidad** — separación de zonas, código zombie limpio, library de UI compartida.

## Sprints

| # | Sprint | Foco | Duración | Bloqueante de |
|---|---|---|---|---|
| 0 | [[Sprint-0-Higiene]] | Rotar keys exposed, validar JWT en boot, configurar webhook Stripe Hugo | 1 semana | nada — esto se hace YA |
| 1 | [[Sprint-1-Aislamiento]] | Migration 018 RLS Fase A, audit_logs, renumerar migrations | 2 semanas | Sprint 5 |
| 2 | [[Sprint-2-Observability]] | pino + correlation IDs + Sentry | 2 semanas | Sprint 3 |
| 3 | [[Sprint-3-Testing]] | vitest + Playwright + 5 flujos críticos + CI | 3 semanas | Sprint 7 |
| 4 | [[Sprint-4-Webhooks-Secrets]] | webhookHandler unificado + migración a Vault | 2 semanas | nada |
| 5 | [[Sprint-5-Multi-Tenant]] | RLS Fase B+C con JWT claims, refactor frontend | 3 semanas | venta a empresa con compliance |
| 6 | [[Sprint-6-WhatsApp-BA]] | RFP + adapter + migración cliente por cliente | 4-6 semanas | escala de clientes con WhatsApp |
| 7 | [[Sprint-7-DX-Cleanup]] | Modularizar god files, useL hook, CSS variables | 2 semanas | nada |
| 8 | [[Sprint-8-Component-Library]] | Lib UI + Storybook + migrar 10 páginas top | 3 semanas | nada |

**Total**: 22-24 semanas de un senior solo.

## Orden recomendado

```
Semana 1:    [Sprint 0]
Semanas 2-3: [Sprint 1] paralelo a [Sprint 4 inicio]
Semanas 4-5: [Sprint 2]
Semanas 6-8: [Sprint 3]
Semanas 9-11: [Sprint 5]
Semanas 12-17: [Sprint 6] (en background, mientras avanzas otros)
Semanas 12-13: [Sprint 7]
Semanas 14-16: [Sprint 8]
```

## Lo que hacer ESTA semana (sin esperar)

1. **Rotar las 7 API keys exposed** ([[Bugs-Criticos#C1]]). 30 minutos. Sin esto, cualquier siguiente sprint es construir sobre arena.
2. **Configurar webhook Stripe Hugo** ([[Bugs-Criticos#C2]]). 15 minutos. Estás perdiendo tracking de revenue ahora mismo.
3. **Validar `JWT_SECRET` en boot** con crash si falta ([[Bugs-Criticos#C4]]). 5 minutos.
4. **Quitar fallback `'default-secret'`**.
5. **Aplicar Fase A de Migration 018** (RLS habilitada con service_role bypass). No-disruptive.

## En curso

- [[Portal-Infoproducto]] — Hitos 1-5 mergeables. Pendiente: merge a main + setear `PORTAL_JWT_SECRET` + smoke test. Hito 6 (cableado contenido real en tabs) queda fuera del scope inicial.

## Después del roadmap (parking lot)

- TypeScript gradual (al menos JSDoc tipos)
- AI gateway propio (Vercel AI Gateway o similar) para multi-provider
- Migrar de `whatsapp-web.js` a un proveedor confiable (parte del Sprint 6)
- Dashboard Grafana para métricas de plataforma
- Storybook publicado como design system docs
- ADRs (Architecture Decision Records) para decisiones grandes

## Conexiones

- De dónde venimos: [[Resumen-Auditoria]]
- Cada sprint: ver MOC [[_moc/MOC-Roadmap]]
- Decisiones que requiere: [[_moc/MOC-Decisiones-CTO]]
