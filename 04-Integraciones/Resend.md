---
tags: [integracion, resend, email]
created: 2026-05-07
---

# Resend — Email multi-tenant

**Estado**: ✅ LIVE multi-tenant. Sin manejo de bouncing/complaints.

## Setup

- **Per-tenant** vía tabla `email_config`:
  - `client_id` + `account_index` (1, 2, ...)  
  - `api_key`, `from_name`, `from_email`, `reply_to`, `domain`, `verified`
- **Multi-account**: `asesorias-suiza` usa account_index=1 (Portillo) y =2 (Lukas)
- **Default global**: `RESEND_API_KEY` en `enjambre-api/.env` como fallback

## Consumers

### Backend — `enjambre-api/src/workers/sequence-worker.js`

Envía emails de secuencias:
- POST a `https://api.resend.com/emails`
- Retry con backoff (1h, 4h, 24h) ✓
- Fallback graceful si `api_key` falta
- Audit en eventBus

### Backend — `enjambre-api/src/routes/portal.js`

Envía OTPs del [[Portal-Infoproducto]]:
- Lee `email_config` del tenant + `account_index` del `portal_user`
- Manda email con código sha256 hasheado en DB
- Audit en `portal_otp_codes` (no en bus)

### Frontend — `Dashboard-Ops-/api/send-email.js`

Proxy CORS para evitar exponer keys al cliente:
- Action `send-emails` con resendApiKey + email list
- Sin retry, sin backoff
- Acción ad-hoc desde EmailMarketingPage

### Otros

- `Dashboard-Ops-/api/forms/cv-coach-suiza.js` — forwarding CV de landing → email del setter Portillo
- (Detrás de Cámara cuando se configure) — campañas

## Lo que falta (deuda)

- 🔴 **Sin manejo de bouncing/complaints**: hard bounces siguen recibiendo emails (drena reputación de dominio)
- 🟡 **Sin webhook de Resend**: eventos `bounced`, `complained`, `delivered` no entran al sistema
- 🟡 **Sin marca en `crm_contacts`** para "email_status = bounced/unsubscribed"
- 🟡 **Sin templates compartidos**: cada lugar arma su HTML inline

## Riesgos

- **Reputación de dominio**: si Hugo manda a 1000 emails y 30% bouncing, su `from_email` pierde reputación. Sin sistema actual, no se entera.
- **Compliance**: GDPR / CAN-SPAM requieren que respetes unsubscribes. Hoy NO hay unsubscribe link automático en todos los mails.

## Configuración por tenant — pattern

Cuando onboardas un cliente nuevo (ver [[Playbook-Onboarding-Cliente-Nuevo]] Fase 3):

```sql
INSERT INTO email_config (client_id, api_key, from_name, from_email, account_index, account_label) VALUES (
  '<client_uuid>',
  're_xxx...',
  'Hugo de En Forma',
  'hugo@enformaconhugo.com',
  1,
  'principal'
);
```

Para multi-account (asesorias-suiza):

```sql
-- Portillo
INSERT ... VALUES ('<asesorias-suiza-uuid>', 're_portillo...', 'Portillo', 'portillo@...', 1, 'portillo');
-- Lukas
INSERT ... VALUES ('<asesorias-suiza-uuid>', 're_lukas...', 'Lukas', 'lukas@...', 2, 'lukas');
```

## Roadmap propuesto (no en sprint actual)

1. Implementar webhook receiver de Resend (`/api/webhooks/resend`)
2. Marcar bounces/complaints en `crm_contacts.email_status`
3. Filtro automático en sequence-worker: skip si `email_status='hard_bounce'`
4. Templates centralizados en `email_templates` table

## Conexiones

- Bug: keys exposed → [[Bugs-Criticos#C1]]
- Política: [[Politica-de-Secretos]]
- Tenant principal: [[Hugo-EnFormaConHugo]] · [[AsesoriaSuiza-Portillo-Lukas]]
- Pendiente config: [[Detras-de-Camara]]
- Portal: [[Portal-Infoproducto]] (consume Resend para OTP)
