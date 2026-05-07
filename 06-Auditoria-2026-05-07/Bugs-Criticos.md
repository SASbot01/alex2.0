---
tags: [auditoria, critical, bugs]
created: 2026-05-07
status: open
---

# Bugs Críticos (sangrando ahora)

9 issues identificados en [[Resumen-Auditoria]] que deberías cerrar esta semana o la siguiente. Ordenados por severidad y velocidad de fix.

---

## C1 — API keys exposed sin rotar

**Severidad**: 🔴 CRÍTICA · **Esfuerzo**: 30 min

Las API keys que circularon en chat (Stripe live x2, Resend x2, Anthropic, Supabase service+management, Cloudflare tunnel) **siguen presentes** en `/home/blackwolfsec/ejambre/.env` y `Dashboard-Ops-/.env`. Los comments del propio `.env` admiten exposure y dicen "rotar al terminar sprint", sprint que nunca terminó.

**Riesgo**: cualquier ex-colaborador con copia de `.env` puede usar Stripe live, mandar mails desde tus dominios, gastar tu Anthropic.

**Fix**: ver [[Playbook-Rotar-Secret]]. Por servicio:

- **Stripe**: dashboard de cada cuenta → API keys → revoke + new restricted key (rk_live_...)
- **Resend**: dashboard → API keys → revoke + new
- **Anthropic**: console.anthropic.com → revoke + new
- **Supabase**: project settings → reset service_role + management token

**Validación**: tras rotar, `curl` smoke test contra cada API y confirmar que old keys dan 401.

---

## C2 — Webhook Stripe Hugo NO configurado

**Severidad**: 🔴 CRÍTICA · **Esfuerzo**: 15 min

`enjambre-api/.env` tiene `STRIPE_WEBHOOK_SECRET=` vacío para la cuenta de Hugo (En Forma con Hugo). Solo está configurado el secret para Portillo (asesoria-suiza).

**Riesgo**: ventas live de Hugo en Stripe **no se sincronizan al CRM**. Probable revenue tracking gap activo.

**Fix**:
1. Stripe Dashboard de Hugo → Developers → Webhooks → endpoint `https://enjambre.blackwolfsec.io/api/webhook/stripe` (o `https://api-enjambre.blackwolfsec.io/api/webhook/stripe`, validar)
2. Eventos: `checkout.session.completed`, `customer.subscription.*`, `invoice.payment_succeeded`
3. Copiar signing secret a `STRIPE_WEBHOOK_SECRET_HUGO=whsec_…` en `.env`
4. Backfill: usar Stripe CLI para reenviar events últimos 30 días

**Validación**: hacer venta de prueba, verificar que aparece en CRM `crm_contacts` + `sales`.

---

## C3 — `/api/webhooks/central` sin validación de firma

**Severidad**: 🔴 CRÍTICA · **Esfuerzo**: 30 min

`enjambre-api/src/routes/index.js:386` recibe webhooks "del central" sin validar firma. Cualquier IP que conozca la URL puede inyectar leads, ventas, o lo que quiera.

**Riesgo**: bot puede contaminar pipelines de cualquier cliente con datos arbitrarios.

**Fix**: añadir middleware `requireWebhookSecret('central')` que valide header `X-Webhook-Secret` contra env var `WEBHOOK_SECRET_CENTRAL`. Patrón existe ya para ManyChat.

---

## C4 — `JWT_SECRET` fallback `'default-secret'`

**Severidad**: 🔴 CRÍTICA · **Esfuerzo**: 5 min

`enjambre-api/src/auth/auth.js:4`:
```javascript
const JWT_SECRET = process.env.JWT_SECRET || 'default-secret';
```

Si la env var falta en boot (deploy fallido, reinicio sin .env, container fresh), todos los tokens son trivialmente falsificables (cualquiera puede firmar JWTs con `'default-secret'`).

**Fix**:
```javascript
const JWT_SECRET = process.env.JWT_SECRET;
if (!JWT_SECRET || JWT_SECRET.length < 32) {
  console.error('[FATAL] JWT_SECRET missing or too short');
  process.exit(1);
}
```

Mismo tratamiento para `PORTAL_JWT_SECRET` (ya tiene check, pero solo loguea warning).

---

## C5 — `localStorage('bw_superadmin', 'true')` = privilege escalation

**Severidad**: 🔴 CRÍTICA · **Esfuerzo**: 1-2 días (real fix)

`Dashboard-Ops-/src/ClientApp.jsx:178, 371`:
```javascript
const isSuperAdmin = !!localStorage.getItem('bw_superadmin')
```

Cualquier usuario con DevTools del navegador puede `localStorage.setItem('bw_superadmin', 'true')` y entra a `/admin/*`. Lo mismo para `bw_employee`.

**Riesgo**: privilege escalation trivial. Si un ex-empleado o usuario malicioso accede al frontend, controla la plataforma.

**Fix corto plazo** (parche): backend endpoint `/api/admin/verify` que valida un token httpOnly y devuelve role real. Frontend nunca confía en localStorage para super-admin checks.

**Fix correcto** (parte de [[Sprint-1-Aislamiento]]): migrar TODA la auth a httpOnly cookies firmadas + CSRF tokens.

---

## C6 — RLS = "ALLOW ALL" en 127 tablas

**Severidad**: 🔴 CRÍTICA · **Esfuerzo**: Sprint completo (ver [[Sprint-1-Aislamiento]])

127 de 157 tablas tienen RLS habilitado **pero todas las policies son `USING (true) WITH CHECK (true)`**. Sin filtro de `client_id`, un usuario de Tenant A con el anon key puede leer/escribir tablas de Tenant B (frontend filtra, pero ese filter no es un boundary de seguridad).

**Riesgo**: leak cross-tenant. Riesgo legal grave (datos personales de cliente A en cliente B).

**Fix**: ya existe el plan en `Dashboard-Ops-/supabase/migrations/018_rls_policies.sql` (DRAFT, sin aplicar). Tres fases:

- **Fase A**: enable RLS + service_role bypass (no-disruptive — aplicar YA)
- **Fase B**: bloquear writes desde anon (forzar paso por API)
- **Fase C**: tenant isolation con JWT claims

Ver [[RLS-y-Seguridad]] y [[Sprint-1-Aislamiento]] para detalle.

---

## C7 — Migrations 045-048 con números duplicados

**Severidad**: 🔴 CRÍTICA · **Esfuerzo**: 1 hora

```
045_client_operators_table.sql    ← conflicto
045_yc_logistics_client.sql       ← conflicto

046_seed_operators_asesorias_suiza.sql  ← conflicto
046_task_pipelines.sql                  ← conflicto

047_fix_form_trabajo_portillo_scope.sql ← conflicto
047_support_tickets.sql                 ← conflicto

048_operator_id_fks_and_backfill.sql ← conflicto
048_tenant_invitations.sql           ← conflicto
```

**Riesgo**: si haces fresh deploy o staging, el orden de aplicación es indeterminado (depende de filesystem). FKs pueden romperse.

**Fix**: renumerar a 049-056 (los que vienen después son 060+). Documentar en [[Politica-de-Migrations]].

---

## C8 — WhatsApp race condition en `findOrCreateCrmContact`

**Severidad**: 🟡 ALTA · **Esfuerzo**: 1 día

`enjambre-api/src/connectors/whatsapp.js:465-499` hace check-then-act sin DB lock. Dos sesiones simultáneas escribiendo al mismo número crean contactos duplicados.

**Evidencia**: ya pasó. Memoria del proyecto menciona "el inbound se pegaba al duplicado" (commit `4df3669`).

**Fix**:
1. Añadir UNIQUE constraint `(client_id, phone_normalized)` en `crm_contacts`
2. Cambiar INSERT a `INSERT ... ON CONFLICT (client_id, phone_normalized) DO UPDATE SET ...`
3. O alternativamente: `SELECT ... FOR UPDATE` en transacción

---

## C9 — WhatsApp web.js sin plan B

**Severidad**: 🟡 ALTA · **Esfuerzo**: Sprint completo (ver [[Sprint-6-WhatsApp-BA]])

`whatsapp-web.js` es no oficial. Simula el cliente web de WhatsApp y es fácilmente detectable. **Ya hubo un banneo el 26-abr en Creator Founder** y la respuesta fue parchear, no migrar.

**Riesgo**: cualquier cliente puede perder su WhatsApp en cualquier momento. Sin recovery, sin abstracción.

**Fix estructural**: migrar a [[Decision-WhatsApp]] — WhatsApp Business API oficial vía 360dialog, Twilio, o Meta directo. 4-6 semanas según proveedor.

**Fix transicional** (mientras llega la migración):
- Detectar "account banned" (timeout de QR persistente) → notificar admin
- Limitar a 3 sesiones simultáneas en producción
- Encriptar `.wwebjs_auth/<sessionKey>` en disco

---

## Conexiones

- Plan de cierre: [[Sprint-0-Higiene]] cubre C1-C4-C7
- Estructural: [[Sprint-1-Aislamiento]] cubre C5-C6
- WhatsApp: [[Sprint-6-WhatsApp-BA]] cubre C9
- CRM dedup: [[Sprint-1-Aislamiento]] o ad-hoc
- Procedimiento: [[Playbook-Rotar-Secret]] · [[Playbook-Aplicar-Migration]]
