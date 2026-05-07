---
tags: [moc, plataforma]
created: 2026-05-07
---

# MOC — Plataforma

Map of Content de la plataforma técnica BlackWolf. Tres componentes vivos: Dashboard-Ops (frontend), enjambre-api (backend), Donna + Army (capa cognitiva).

## Visión global

- [[Arquitectura-General]] — diagrama y flujos entre componentes
- [[ClientTypes]] — qué tipos de clientes existen y qué módulos ven
- [[Multi-Tenant]] — cómo se aísla cada tenant (y cómo NO se aísla)

## Frontend — Dashboard-Ops

- [[Dashboard-Ops]] — repo, deploy Vercel, sidebar, app-shell
- [[Sistema-Auth]] — auth custom + Supabase anon
- [[Sistema-Permisos]] — `permissions.js`, matriz `can()`
- [[Sistema-Theming]] — light/dark, FORCE_LIGHT_SLUGS, paletas per-tenant
- [[Sistema-i18n]] — patrón ad-hoc `L(es, en, zh)`

## Backend — enjambre-api

- [[Enjambre-API]] — Fastify, routes, connectors, workers
- [[Webhooks]] — patrón actual y carencias
- [[Brain-Decisions]] — tabla central donde cae todo el reasoning

## Capa cognitiva

- [[Donna-y-Army]] — orquestador + commanders por dominio
- [[Doctrines]] — los markdowns que le dicen a cada commander qué hacer

## Módulos

- [[Modulo-CRM]] — el más usado, mega-componente CrmPage 4524 LOC
- [[Modulo-Tickets]] — soporte interno (BlackWolf) y por-tenant
- [[Modulo-Formacion-Infoproducto]] — cursos, comunidad, mentorías, marketplace, portal público
- [[Modulo-Marketing]] — Meta Ads, campaigns, creatives
- [[Modulo-AI-Agents]] — Setter AI, Outreach, Email AI, Brain
- [[Modulo-Stores]] — FBA Academy y similares (gestión tiendas)
- [[Modulo-Manufacturing]] — ERP manufacturing
- [[Modulo-Logistics]] — IC Logistics
- [[Modulo-Mentorias]] — bookings 1:1
- [[Modulo-Comunidad]] — feed/posts internos
- [[Modulo-Marketplace]] — solo growth, jobs↔alumnos
- [[Modulo-Tareas]] — task management + sprints
- [[Modulo-Reports]] — dashboards y tablas de venta
- [[Modulo-Finanzas]] — comisiones, pagos, contabilidad

## Conexiones

- Estado de salud: [[_moc/MOC-Auditoria]]
- Hacia dónde va: [[_moc/MOC-Roadmap]]
