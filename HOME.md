---
tags: [home, dashboard]
created: 2026-05-07
updated: 2026-05-07
---

# 🧠 HOME — Cerebro Alex 2.0

Punto de entrada del segundo cerebro. Todo lo que necesitas saber sobre BlackWolf y su plataforma vive aquí, cross-linkeado en Obsidian.

---

## 🎯 Quick links — lo que más vas a necesitar

| Si vienes a... | Empieza por |
|---|---|
| Entender la empresa | [[BlackWolf-Identidad]] · [[Modelo-de-Negocio]] |
| Saber cómo funciona la plataforma | [[Arquitectura-General]] · [[Dashboard-Ops]] · [[Enjambre-API]] |
| Onboardear cliente nuevo | [[Playbook-Onboarding-Cliente-Nuevo]] |
| Investigar un cliente concreto | [[_MOC-Clientes]] |
| Saber qué está roto y qué hay que arreglar | [[Bugs-Criticos]] · [[Deuda-Tecnica]] · [[Codigo-Zombie]] |
| Decidir el siguiente paso | [[Roadmap-Estrategico]] · [[_MOC-Roadmap]] |
| Tomar una decisión técnica grande | [[_MOC-Decisiones-CTO]] |
| Resolver un incidente | [[Playbook-Investigar-Incidente]] |

---

## 🗺️ MOCs principales (Maps of Content)

- [[_moc/MOC-Empresa]] — quiénes somos, modelo, equipo
- [[_moc/MOC-Plataforma]] — Dashboard-Ops + enjambre-api + módulos
- [[_moc/MOC-Clientes]] — los 16 tenants vivos
- [[_moc/MOC-Tecnologia]] — stack, convenciones, deployment
- [[_moc/MOC-Integraciones]] — Stripe, Resend, WhatsApp, Google, Meta...
- [[_moc/MOC-Operaciones]] — Donna, Army, AI cost tracking
- [[_moc/MOC-Auditoria]] — auditoría 2026-05-07
- [[_moc/MOC-Roadmap]] — del MVP al SaaS empresarial
- [[_moc/MOC-Decisiones-CTO]] — ADRs y políticas

---

## 🚨 Estado actual (2026-05-07)

**En producción y operativo:**
- Dashboard-Ops sirviendo a 16 tenants en `central.blackwolfsec.io`
- Enjambre-API en `enjambre.blackwolfsec.io` vía Cloudflare Tunnel
- Donna en Telegram, Army-CRM commander activo
- Stripe live (Hugo + Asesoría Suiza), Resend live, WhatsApp web.js multi-sesión

**En curso:**
- [[Portal-Infoproducto]] — Hitos 1-5 mergeables, falta merge a main + setear `PORTAL_JWT_SECRET` + smoke test
- Hito 6 (cableado contenido real en tabs Hub) — pendiente

**Sangrando:**
- Las API keys que circularon en chat (Stripe, Resend, Anthropic, Supabase) **no se han rotado todavía**. Ver [[Bugs-Criticos#C1]].
- Webhook Stripe Hugo NO configurado → revenue de Hugo no se sincroniza al CRM. Ver [[Bugs-Criticos#C2]].
- RLS = "ALLOW ALL" en 127 tablas → multi-tenant solo es enforced en frontend. Ver [[RLS-y-Seguridad]].
- `localStorage('bw_superadmin')` = privilege escalation trivial. Ver [[Bugs-Criticos#C5]].

---

## 🎯 Qué hacer esta semana

1. [[Playbook-Rotar-Secret]] aplicado a las 7 keys exposed.
2. Configurar webhook Stripe Hugo en `enjambre-api/.env`.
3. Validar `JWT_SECRET` en boot (crash si falta).
4. Empezar [[Sprint-0-Higiene]].

---

## 🧩 Sobre Alex (tú)

- [[Alejandro-CTO]] — perfil, foco, modos de trabajo
- [[Diario-Decisiones]] — bitácora de decisiones grandes

---

## 📚 Convenciones del vault

- Wikilinks internos: `[[Archivo]]`
- Tags: `#critical`, `#deuda`, `#zombie`, `#cliente`, `#decision-cto`, `#playbook`
- Fechas absolutas siempre
- Si está roto, se documenta roto
