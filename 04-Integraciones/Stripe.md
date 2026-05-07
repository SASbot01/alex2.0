---
tags: [integracion, stripe, critical]
created: 2026-05-07
---

# Stripe — Billing multi-cuenta

**Estado**: 🟡 LIVE en 2 cuentas. **Webhook Hugo NO configurado** = revenue gap activo.

## Setup actual

| Cuenta | account_id | Estado | Webhook |
|---|---|---|---|
| **Hugo (En Forma con Hugo)** | `acct_1Qie25...` | live | 🔴 NO configurado |
| **Portillo (Asesoría Suiza)** | `acct_1S6I7L...` | live (CHF, 36 charges histórico) | ✅ `whsec_4ojzPB1t6MRqWQUV3CefMBbJAGqKmWEH` |

## Webhook handler

`Dashboard-Ops-/api/webhook/stripe.js` — bien escrito:

- ✅ Valida firma HMAC-SHA256
- ✅ Anti-replay: rechaza eventos > 5 min
- ✅ Crea/vincula contacto CRM por email
- ✅ Registra venta con dedup por `session_id`
- ✅ `writeAudit()` en 3 rutas críticas

Eventos manejados:
- `checkout.session.completed`
- `customer.subscription.*`
- `invoice.payment_succeeded`

URL endpoint: `https://enjambre.blackwolfsec.io/api/webhook/stripe` (o `https://api-enjambre.blackwolfsec.io/api/webhook/stripe`)

## Bug crítico activo

[[Bugs-Criticos#C2]]: webhook Hugo NO configurado en `enjambre-api/.env` (`STRIPE_WEBHOOK_SECRET=` vacío). **Las ventas live de Hugo no se sincronizan al CRM ahora mismo**.

## Configuración multi-tenant

Pattern (cuando funcione bien):

```env
# enjambre-api/.env
STRIPE_SECRET_KEY_HUGO=rk_live_51Qie25...
STRIPE_WEBHOOK_SECRET_HUGO=whsec_...   ← VACÍO HOY

STRIPE_SECRET_KEY_PORTILLO=rk_live_51S6I7L...
STRIPE_WEBHOOK_SECRET_PORTILLO=whsec_4ojzPB1t6MRqWQUV3CefMBbJAGqKmWEH
```

El webhook handler decodea la firma para detectar a qué cuenta pertenece y enrutar al `client_id` correcto.

## Pendientes

- 🔴 **Configurar webhook Hugo + backfill últimos 30 días** (ver [[Sprint-0-Higiene]] T0.2)
- 🔴 **Rotar AMBAS secret keys** (circularon en chat)
- 🟡 Sin retry automático si DB falla durante webhook handling
- 🟡 Sin alertas si webhook eventos rechazados
- 🟡 Sincronización de subscriptions a `billing_subscriptions` (sólo se sincronizan products/prices via `stripe-sync.js`)

## Sincronización products → CRM

`Dashboard-Ops-/api/stripe-sync.js` y `sync-products.js`:
- Sincroniza `products` y `prices` desde Stripe → tabla `products` del CRM
- Manual trigger (no auto-cron)
- Cubre Hugo y Portillo

## Conexiones

- Bug: [[Bugs-Criticos#C2]]
- Sprint: [[Sprint-0-Higiene]] T0.2
- Tenants: [[Hugo-EnFormaConHugo]] · [[AsesoriaSuiza-Portillo-Lukas]]
- Webhook code: `Dashboard-Ops-/api/webhook/stripe.js`
- Política de keys: [[Politica-de-Secretos]]
