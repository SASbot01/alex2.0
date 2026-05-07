---
tags: [moc, integraciones]
created: 2026-05-07
---

# MOC — Integraciones externas

12 integraciones detectadas. Estado real en 2026-05-07.

## Live (operativas)

| Servicio | Uso | Tenants | Estado | Riesgo |
|---|---|---|---|---|
| [[Stripe]] | Billing | Hugo + Portillo | live | 🔴 keys exposed + webhook Hugo no configurado |
| [[Resend]] | Email transaccional + campañas | multi-tenant via `email_config` | live | 🟡 sin bouncing/complaints |
| [[WhatsApp]] | CRM 2-way (whatsapp-web.js) | multi-tenant | live | 🔴 banneo 26-abr · plan B inexistente |
| [[Google]] | OAuth Calendar/Drive/Gmail | asesoria-suiza | live | 🟡 Drive token hardcoded a personal |
| [[Meta-Ads]] | Marketing dashboard | tenant marketing-active | live | 🟡 sin token refresh automático |
| [[Discord]] | Bot de agente | empresa interna | live | 🟡 dual bot Node+Python |
| [[Cloudflare]] | Tunnel enjambre-api | infra | live · uptime corto | 🟢 |
| [[Supabase]] | DB + Storage + Auth | todos | live | 🔴 keys exposed + RLS permisivas |
| [[Anthropic-Claude]] | Capa cognitiva | todos | live | 🔴 key exposed |

## Zombie

- [[ManyChat-Zombie]] — DB limpia, pero webhook + helpers + tabla siguen vivos en código

## Pending

- [[Hotmart]] — para Hugo, scaffolding sin construir
- [[Fathom]] — transcripciones, API key faltante

## Conexiones

- Auditoría: [[Resumen-Auditoria]]
- Política de secretos: [[Politica-de-Secretos]]
- Webhooks: [[Webhooks]]
