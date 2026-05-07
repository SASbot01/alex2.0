---
tags: [cliente, growth, evento]
slug: creator-founder
client_type: growth
status: live
created: 2026-05-07
---

# Creator Founder

Cliente growth con evento físico Madrid 2-may-2026. Tuvo banneo de WhatsApp el 26-abr — incidente clave para [[Decision-WhatsApp]].

## Identidad

- **Slug**: `creator-founder`
- **clientType**: `growth`
- **Evento clave**: Madrid 2 mayo 2026

## Particularidades

### Pipeline Evento

Pipeline específico para el evento Madrid. Stages:
- Inscripción
- Segundo Form (estado clave para contactabilidad)
- Confirmado / Asistirá

### Endpoint check-in HMAC

`/api/cf/checkin` con HMAC firmado. Genera QR para los asistentes confirmados.

### Campaña Resend "plaza confirmada + QR"

Campaña automatizada que sale a quien llega a stage "Segundo Form" con:
1. Confirmación de plaza
2. QR único firmado

### Incidente WhatsApp 26-abr

WhatsApp del cliente fue baneado durante outreach. Fue el primer signal claro de que `whatsapp-web.js` es estructuralmente frágil. Disparó la conversación de migración a WhatsApp Business API ([[Decision-WhatsApp]] · [[Sprint-6-WhatsApp-BA]]).

**Lección aprendida**: el banneo no es excepcional, es esperable. Sin plan B, cualquier cliente con outreach masivo está en riesgo.

## Conexiones

- Plataforma: [[Modulo-CRM]] (pipeline evento) · [[Webhooks]] (check-in HMAC)
- Riesgo: [[WhatsApp]] · [[Decision-WhatsApp]]
- Incidente: [[Bugs-Criticos#C9]]
