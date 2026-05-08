---
tags: [cliente, infoproducto, growth]
slug: enformaconhugo
client_type: growth
status: live
created: 2026-05-07
---

# Hugo — En Forma con Hugo

Cliente growth (infoproducto fitness). Stripe live integrado, Hotmart pendiente.

## Identidad

- **Slug**: `enformaconhugo`
- **clientType**: `growth`
- **Director**: Hugo
- **Vertical**: fitness / health infoproducto

## Estado integraciones

| Integración | Estado | Detalle |
|---|---|---|
| Stripe | 🟡 keys configuradas, **webhook NO** | Cuenta `acct_1Qie25...` — secret está en `.env` pero el webhook NO está configurado en el dashboard Stripe. Ver [[Bugs-Criticos#C2]] |
| Hotmart | 🔴 pendiente | Scaffolding sin construir. Memoria proyecto lo marca como "para Hugo" |
| Resend | ✅ live | Email transaccional + campañas |
| WhatsApp | ✅ live | CRM 2-way |
| GCal | ❓ verificar | |

## Pipelines CRM

4 pipelines configurados. **47% failure rate** según KPIs históricos — un alumno de cada dos no completa el flujo. Esto es la métrica más importante a mover.

## Riesgos / Gotchas

1. **Webhook Stripe Hugo no configurado** → ventas no sincronizan al CRM. Crítico, ver [[Sprint-0-Higiene]] T0.2.
2. **Hotmart pendiente** → si Hugo vende también por Hotmart, esos leads no entran al CRM.
3. Tasa de fallo alta (47%) — ¿es problema de producto, de pipeline CRM o de seguimiento?

## Conversaciones / decisiones notables

- Memoria del proyecto registra a Hugo como brain vivo del cliente con identidad, accesos, infra (4 pipelines), Stripe live, KPIs.

## Conexiones

- Plataforma: [[Modulo-CRM]] · [[Modulo-Formacion-Infoproducto]]
- Integraciones: [[Stripe]] · [[Hotmart]]
- Tipo: [[Tipos-de-Cliente]]
