---
tags: [moc, tecnologia]
created: 2026-05-07
---

# MOC — Tecnología

Map of Content del stack técnico, convenciones y deployment.

## Stack

- [[Stack-Tecnico]] — visión global de qué corre dónde
- [[Frontend-React-Vite]] — React 19 + Vite 7 + react-router 7
- [[Backend-Node-Fastify]] — Fastify 5 + Node 18 + Anthropic SDK
- [[Database-Supabase]] — Postgres + RLS + Storage + Auth
- [[Cloudflare]] — Tunnel `enjambre-v2` para enjambre-api
- [[Anthropic-Claude]] — modelo principal de la capa cognitiva

## Seguridad

- [[RLS-y-Seguridad]] — postura actual (permissive, deuda crítica)
- [[Sistema-Auth]] — auth custom team + anon key + portal JWT
- [[Politica-de-Secretos]] — dónde viven, cómo se rotan

## Convenciones

- [[Convenciones-Codigo]] — naming, commits, autor (SASbot01/aatshadow)
- [[Migrations]] — política propuesta + estado actual (045-048 duplicados)
- [[Naming-Conventions]] — slugs, archivos, ramas

## DevOps

- [[Despliegue]] — Vercel auto-deploy + ejambre-stack.service Docker
- [[Testing]] — estado actual: NULO. Plan: vitest + Playwright
- [[Observability]] — propuesta: pino + correlation IDs + Sentry

## Conexiones

- Roadmap técnico: [[_moc/MOC-Roadmap]]
- Decisiones: [[_moc/MOC-Decisiones-CTO]]
