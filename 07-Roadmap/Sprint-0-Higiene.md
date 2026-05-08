---
tags: [sprint, critical]
sprint: 0
duration: 1 week
status: ready
created: 2026-05-07
---

# Sprint 0 — Higiene (1 semana, prioridad MÁXIMA)

Lo que hace TODA la plataforma más segura sin tocar arquitectura. Si no haces estos 5 antes que nada, los demás sprints construyen sobre arena.

## Tareas

### T0.1 — Rotar las 7 API keys exposed

**Esfuerzo**: 30-60 min  
**Bloquea**: nada

Por servicio:
- Stripe Hugo (acct_1Qie25…) — revoke + new restricted key (`rk_live_...`)
- Stripe Portillo (acct_1S6I7L…) — idem
- Resend enjambre-api — revoke + new
- Resend Dashboard-Ops — revoke + new (la del CV forwarding asesoria-suiza)
- Anthropic — revoke + new (frontend y backend usan la misma)
- Supabase service_role — reset
- Supabase management token — reset
- Cloudflare tunnel token — rotate

Procedimiento por servicio en [[Playbook-Rotar-Secret]].

**Validación**:
```bash
# Confirmar que old keys dan 401
curl -H "Authorization: Bearer <old_key>" https://api.stripe.com/v1/customers
# → 401
```

### T0.2 — Configurar webhook Stripe Hugo

**Esfuerzo**: 15 min  
**Bloquea**: revenue tracking de Hugo ya está roto

1. Stripe Dashboard de Hugo → Developers → Webhooks → Add endpoint
2. URL: `https://enjambre.blackwolfsec.io/api/webhook/stripe`
3. Eventos: `checkout.session.completed`, `customer.subscription.*`, `invoice.payment_succeeded`
4. Copiar signing secret a `STRIPE_WEBHOOK_SECRET_HUGO=whsec_…` en `enjambre-api/.env`
5. Backfill últimos 30 días con Stripe CLI:
   ```bash
   stripe events resend --account=acct_1Qie25... --since=2026-04-07
   ```

**Validación**: hacer una venta de prueba en Hugo, verificar que aparece en `crm_contacts` + `sales` del CRM.

### T0.3 — Validar `JWT_SECRET` en boot

**Esfuerzo**: 5 min  
**Bloquea**: nada

`enjambre-api/src/auth/auth.js`:
```javascript
// ANTES
const JWT_SECRET = process.env.JWT_SECRET || 'default-secret';

// DESPUÉS
const JWT_SECRET = process.env.JWT_SECRET;
if (!JWT_SECRET || JWT_SECRET.length < 32) {
  console.error('[FATAL] JWT_SECRET missing or too short. Generate with: openssl rand -hex 64');
  process.exit(1);
}
```

Aplica el mismo patrón a `PORTAL_JWT_SECRET` (hoy solo loggea warning, debería crashear).

### T0.4 — Validación de firma en `/api/webhooks/central`

**Esfuerzo**: 30 min  
**Bloquea**: nada

Patrón existe ya para `/api/webhooks/manychat`. Replicar:
```javascript
function requireWebhookSecret(name) {
  return async (req, reply) => {
    const expected = process.env[`WEBHOOK_SECRET_${name.toUpperCase()}`];
    const got = req.headers['x-webhook-secret'];
    if (!expected || !got || got !== expected) {
      return reply.code(401).send({ error: 'invalid_webhook_secret' });
    }
  };
}

app.post('/api/webhooks/central', { preHandler: requireWebhookSecret('central') }, async (req, reply) => {
  // ...
});
```

Coordinar con quien envía al webhook para que añada el header.

### T0.5 — Renumerar migrations 045-048 duplicadas

**Esfuerzo**: 1 hora  
**Bloquea**: deploys frescos

```
045_client_operators_table.sql       → mantener
045_yc_logistics_client.sql          → renombrar a 049_yc_logistics_client.sql

046_seed_operators_asesorias_suiza   → mantener
046_task_pipelines.sql                → renombrar a 050_task_pipelines.sql

047_fix_form_trabajo_portillo_scope  → mantener
047_support_tickets.sql              → renombrar a 051_support_tickets.sql

048_operator_id_fks_and_backfill     → mantener
048_tenant_invitations.sql           → renombrar a 052_tenant_invitations.sql
```

Probar en staging Supabase: spin fresh DB, aplicar migrations en orden alfabético, confirmar FKs OK.

Documentar política en [[Politica-de-Migrations]].

### T0.6 — Aplicar Migration 018 Fase A

**Esfuerzo**: 1 hora  
**Bloquea**: Sprint 1

Migration `Dashboard-Ops-/supabase/migrations/018_rls_policies.sql` ya existe en draft. Fase A es no-disruptive: enable RLS + service_role bypass. NADIE lo nota porque el backend usa service_key.

```sql
-- Fase A — pseudocódigo
ALTER TABLE crm_contacts ENABLE ROW LEVEL SECURITY;
CREATE POLICY "service_role_all" ON crm_contacts FOR ALL TO service_role USING (true);
-- (mantener policies anon "allow all" temporalmente; se quitan en Fase B)
```

Aplica a las 30 tablas que aún no tienen RLS habilitada.

**Validación**: backend + frontend siguen funcionando idéntico que antes.

### T0.7 — Quitar `localStorage('bw_superadmin')` parche

**Esfuerzo**: 4 horas  
**Bloquea**: Sprint 5

Parche de seguridad sin migrar a httpOnly aún. Backend endpoint `/api/admin/verify` que valida un token y devuelve role real. Frontend nunca confía en localStorage.

Esto es transicional — la migración real a httpOnly cookies es parte de [[Sprint-4-Webhooks-Secrets]] o un sprint propio de auth.

## Definition of Done

- [ ] Las 7 API keys rotadas. Old keys dan 401 contra el servicio. *(BLOQUEA Alejandro — requiere acceso dashboards)*
- [ ] Webhook Stripe Hugo configurado y backfilled. Test E2E pasa. *(BLOQUEA Alejandro — requiere acceso Stripe Hugo)*
- [x] Boot del backend crashea si `JWT_SECRET` falta. *(commit 4397db7, ya en main)*
- [x] `/api/webhooks/central` rechaza requests sin signature. *(commit 4397db7 + PR #28 fix de callers Dashboard-Ops 2026-05-08)*
- [x] Migrations 045-048 renumeradas. Fresh deploy en staging OK. *(commit 0229c35)*
- [x] Migration 018 Fase A aplicada en producción. Sin regresiones. *(2026-05-07 noche)*
- [x] `bw_superadmin` ya no es válido desde DevTools. *(PR #31 — server-side verify on boot, transicional)*

## Cierre 2026-05-08 (sesión paralela 2 terminales)

**Hecho (5 de 7 DoD items):**
- 4 crm_tasks del Sprint Arreglos en BD que estaban como `todo` pero ya estaban hechas en código → marcadas `done` con nota del commit.
- **PR #28 aatshadow/Dashboard-Ops-** — bug post-Sprint 0: `bridge-enjambre.js` y `agent.js` no enviaban `x-webhook-secret` en sus llamadas a `/api/webhooks/central`. Desde el deploy del Sprint 0 todo `Dashboard-Ops → enjambre-api` devolvía 401 silencioso. Fix + Vercel env `CENTRAL_WEBHOOK_SECRET` seteada (id `mPwsym12vTBaO4fm`).
- **PR #31 aatshadow/Dashboard-Ops-** — T0.7 cerrado: nuevo `GET /api/admin?action=verify` + `src/lib/superadmin.js` helper + `useEffect` de boot en `ClientApp.jsx`. Los 15 callsites legacy siguen leyendo `localStorage.bw_superadmin` directo, pero el value solo está si el JWT verifica server-side al montar.
- **PR #3 SASbot01/ejambre** — race condition `findOrCreateCrmContact`. Defensive re-find recovery tras insert failed. Cura definitiva (UNIQUE INDEX) bloqueada por 151 pares duplicados preexistentes.

**Pendiente para Alejandro:**
- Mergear PRs #28, #31 (aatshadow/Dashboard-Ops-) y #3 (SASbot01/ejambre).
- Rotar las 7 API keys (T0.1) — playbook listo.
- Configurar webhook Stripe Hugo (T0.2) — 15 min en dashboard Stripe.

## Conexiones

- Bugs que cierra: [[Bugs-Criticos]] C1, C2, C3, C4, C5 (parche), C6 (parcial), C7
- Siguiente: [[Sprint-1-Aislamiento]]
