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
| Saber cómo funciona la plataforma BlackWolf | [[Arquitectura-General]] · [[Dashboard-Ops]] · [[Enjambre-API]] |
| **Saber qué es Cargonex** (segunda línea de producto) | [[Cargonex]] · [[Arquitectura]] |
| Onboardear cliente nuevo en Dashboard-Ops | [[Playbook-Onboarding-Cliente-Nuevo]] |
| Onboardear tenant Cargonex (logística) | [[Playbook-Crear-Tenant-Cargonex]] |
| Investigar un cliente concreto | [[_MOC-Clientes]] |
| Saber qué está roto y qué hay que arreglar | [[Bugs-Criticos]] · [[Deuda-Tecnica]] · [[Codigo-Zombie]] |
| Decidir el siguiente paso | [[Roadmap-Estrategico]] · [[_MOC-Roadmap]] |
| Tomar una decisión técnica grande | [[_MOC-Decisiones-CTO]] |
| Resolver un incidente | [[Playbook-Investigar-Incidente]] |

---

## 🗺️ MOCs principales (Maps of Content)

- [[_moc/MOC-Empresa]] — quiénes somos, modelo, equipo
- [[_moc/MOC-Plataforma]] — los dos productos: Apex + Cargonex
- [[_moc/MOC-Clientes]] — tenants vivos (Apex 16 + Cargonex 1)
- [[_moc/MOC-Tecnologia]] — stack, convenciones, deployment
- [[_moc/MOC-Integraciones]] — Stripe, Resend, WhatsApp, Google, Meta...
- [[_moc/MOC-Operaciones]] — Donna, Army, AI cost tracking
- [[_moc/MOC-Auditoria]] — auditoría 2026-05-07
- [[_moc/MOC-Roadmap]] — del MVP al SaaS empresarial
- [[_moc/MOC-Decisiones-CTO]] — ADRs y políticas

## 🧪 I+D (research, spikes, prototipos)

- [[I+D-Apex]] — RLS migration, WhatsApp BA, Hotmart, Stripe SaaS billing, Donna v2, AI Agents v2 + tech radar
- [[I+D-Cargonex]] — migración legacy Eilers, tour planner v2, OBD2 telemetry, DSGVO automation, mobile UX, Stripe Connect, multi-language EU + tech radar

## 📁 Estructura del vault (post-reorg 2026-05-08)

```
.                       ← cross-cutting (este HOME, _moc, README)
├── 00-Empresa/         ← BlackWolf, modelo de negocio
├── 04-Integraciones/   ← Stripe, Resend, WhatsApp (compartidos)
├── 06-Auditoria-…/     ← auditoría plataforma
├── 07-Roadmap/         ← roadmap estratégico cross-producto
├── 08-Decisiones-CTO/  ← ADRs cross-producto + política de secretos
├── 09-Playbooks/       ← playbooks cross-producto (rotar secret, etc.)
├── 10-Glosario/        ← términos
├── 99-Personal/        ← Alejandro, decisiones, diario
│
├── apex/               ← producto Dashboard-Ops + Enjambre-API + Donna/Army
│   ├── Plataforma/
│   ├── Clientes/       ← los 16 tenants growth/consultoría/etc
│   ├── Playbooks/      ← onboarding cliente Apex
│   └── I+D/            ← investigación específica Apex
│
└── cargonex/           ← producto SaaS logística europea
    ├── Plataforma/
    ├── Clientes/       ← Eilers Logistik
    ├── Playbooks/      ← onboarding tenant Cargonex
    └── I+D/            ← investigación específica Cargonex
```

---

## 🚨 Estado actual (2026-05-07)

**Dashboard-Ops (BlackWolf platform) — en producción y operativo:**
- Dashboard-Ops sirviendo a 16 tenants en `central.blackwolfsec.io`
- Enjambre-API en `enjambre.blackwolfsec.io` vía Cloudflare Tunnel
- Donna en Telegram, Army-CRM commander activo
- Stripe live (Hugo + Asesoría Suiza), Resend live, WhatsApp web.js multi-sesión

**[[Cargonex]] (segunda línea de producto) — en producción y operativo:**
- Marketing landing live en `web.cargonex.co`
- Portal cliente + activación PI-key live en `app.cargonex.co/portal`
- Slug routing operativo (`app.cargonex.co/EilersLogistik`)
- Admin con datos reales en `app.cargonex.co/admin` (rebrandeado 2026-05-07)
- [[Eilers-Logistik]] live: 231 Fahrer / 162 Fahrzeuge / 8343 records
- Stack: Express + Postgres + Cloudflare Tunnel en VPS compartido — ver [[Host-Docker-Layout]]

**En curso:**
- [[Portal-Infoproducto]] — Hitos 1-5 mergeables, falta merge a main + setear `PORTAL_JWT_SECRET` + smoke test
- Hito 6 (cableado contenido real en tabs Hub) — pendiente
- Cargonex: SMTP cutover a Resend, DSGVO docs Eilers, mobile PIN test

**Sangrando:**
- Las API keys que circularon en chat (Stripe, Resend, Anthropic, Supabase) **no se han rotado todavía**. Ver [[Bugs-Criticos#C1]].
- Webhook Stripe Hugo NO configurado → revenue de Hugo no se sincroniza al CRM. Ver [[Bugs-Criticos#C2]].
- RLS = "ALLOW ALL" en 127 tablas → multi-tenant solo es enforced en frontend. Ver [[RLS-y-Seguridad]].
- `localStorage('bw_superadmin')` = privilege escalation trivial. Ver [[Bugs-Criticos#C5]].
- **Cargonex `PLATFORM_ADMIN_SECRET = "testtoken123"`** en producción — placeholder de dev sin rotar. Anyone con ese string toma `/admin` de Cargonex. Ver [[Cargonex#Estado]].
- Token GitHub PAT compartido en chat el 2026-05-07 (push a `eilerrs-portal`) — sin rotar. Ver [[Politica-de-Secretos]].

---

## 🎯 Qué hacer esta semana

1. [[Playbook-Rotar-Secret]] aplicado a las 7 keys exposed.
2. Configurar webhook Stripe Hugo en `enjambre-api/.env`.
3. Validar `JWT_SECRET` en boot (crash si falta).
4. Empezar [[Sprint-0-Higiene]].

> **Sprints operativos en producción** (`central.blackwolfsec.io/black-wolf/task-management`):
> - [[Sprint-Setter-WhatsApp]] — 25 tareas · 72.5h · **MVP lunes 2026-05-11** (19.5h bloqueantes 🔴). Setter full optimizado para grupos+contactos.
> - [[Sprint-Arreglos]] — 54 tareas · 339h. Hallazgos auditoría 2026-05-07 (security, dx, observability, etc).

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
