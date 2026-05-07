---
tags: [moc, auditoria]
created: 2026-05-07
audit_date: 2026-05-07
---

# MOC — Auditoría 2026-05-07

Estudio integral de la plataforma post-Hito 5 del Portal Infoproducto. 4 sub-auditorías en paralelo: frontend, backend, DB, integraciones.

## Resumen ejecutivo

- [[Resumen-Auditoria]] — visión global del estado real
- [[Fortalezas]] — lo que SÍ está bien
- [[Flaquezas-Estructurales]] — los 10 patrones que duelen

## Bugs (lo que sangra)

- [[Bugs-Criticos]] — 9 issues que deberías cerrar esta semana
- [[Codigo-Zombie]] — ManyChat residual, Discord Node, scripts dispersos
- [[Deuda-Tecnica]] — god files, i18n duplicado, modales inline

## Por área

- [[Auditoria-Frontend]] — Dashboard-Ops · 117K LOC · 210 páginas
- [[Auditoria-Backend]] — enjambre-api · ~7K LOC · 50+ endpoints
- [[Auditoria-DB]] — 166 tablas · RLS permisivas · migrations duplicadas
- [[Auditoria-Integraciones]] — 12 integraciones · keys exposed
- [[Seguridad-Posture]] — RLS, secrets, auth, CORS
- [[Performance-DB]] — índices, queries lentas potenciales

## Conexiones

- Qué hacer ahora: [[_moc/MOC-Roadmap]]
- Decisiones que requiere: [[_moc/MOC-Decisiones-CTO]]
