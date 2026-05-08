---
tags: [cliente, consultoria, multi-owner]
slug: asesorias-suiza
client_type: consultoria
status: live
created: 2026-05-07
updated: 2026-05-07
---

# Asesoría Suiza — Portillo + Lukas

Cliente consultoría **multi-owner** (Portillo + Lukas, dos profesionales bajo el mismo tenant). Es el primer y único caso multi-owner en producción — el patrón que se hardcodea hoy es el que define cómo escalas a otros multi-owner.

## Identidad

- **Slug**: `asesorias-suiza` (con S, plural — atención: en mi memoria vieja estaba "asesoria-suiza" en singular, eso era incorrecto)
- **client_id**: `811278fd-892f-4709-8ae6-2d7faa67bd1c`
- **clientType**: `consultoria`
- **Owners**: Portillo (account_index=1), Lukas (account_index=2)
- **Idiomas**: `es` + `en`

## Productos

### Portillo
- Servicios de consultoría 1:1 (sesiones cobradas en CHF)
- **Stripe**: cuenta `acct_1S6I7L...` LIVE (CHF) — 36 charges all-time histórico
- **Webhook Stripe**: ✅ configurado (`whsec_4ojzPB1t6MRqWQUV3CefMBbJAGqKmWEH`)
- **Bookings**: GCal freebusy live para evitar overlap

### Lukas
- Servicios complementarios bajo el mismo tenant
- Cuenta Resend separada (`account_index=2`) — multi-account email config

### Formación: "Cómo Trabajar en Suiza — Método Portillo"

Programa formativo de Portillo en `central.blackwolfsec.io/asesorias-suiza/formacion`. Estructura creada **2026-05-07**:

- **Ruta**: Cómo Trabajar en Suiza — Método Portillo
- **Formation**: Programa Completo Portillo
- **6 módulos**:
  1. Hoja de Ruta y Primeros Pasos (4 lecciones)
  2. Curso de Alemán A-1 (4 lecciones)
  3. Módulo 1 — La Verdad Sin Filtros: Mentalidad y Estrategia (5 lecciones)
  4. Módulo 2 — El Mapa Legal: Permisos, Papeles y Plan de Salida (6 lecciones)
  5. Módulo 3 — Cómo Conseguir Trabajo Rápido en Suiza: el Método Portillo (6 lecciones)
  6. Módulo 4 — Tu Primer Mes: Alojamiento, Llegada y Lo Que Nadie Te Cuenta (6 lecciones)
- **31 lecciones** total. URLs de vídeo pendientes (Alejandro las añade).

Ver [[Formacion-Portillo-Suiza]] para el detalle del syllabus.

## Patrón multi-owner (hardcoded hoy)

Hoy el código asume que **solo `asesorias-suiza` es multi-owner**. Hardcoded en:

- `src/pages/infoproducto/UsuariosPage.jsx:51` — `const isMultiOwner = clientSlug === 'asesoria-suiza'` (atención: hardcoded con typo de slug singular, **revisar**)
- `enjambre-api/src/routes/portal.js` — owner_scope='portillo' → email_account_index=1, 'lukas' → 2
- `Dashboard-Ops-/src/contexts/ClientContext.js` — `memberOwnerScope` para filtrar pipelines/contactos por owner

Cuando llegue el segundo cliente multi-owner, esto explota. Ver [[Decision-Multi-Owner-Generic]] (pendiente) y [[Sprint-7-DX-Cleanup]].

## Pipelines CRM

- Pipeline Trabajo (cv forwarding desde landing `/suizatrabajo`)
- Pipelines de consultoría (Portillo y Lukas filtrados por `owner_scope`)

## Stripe — config técnica

```env
# enjambre-api/.env (sólo Portillo configurado; Lukas no tiene Stripe propio aún)
STRIPE_SECRET_KEY_PORTILLO=rk_live_51S6I7L...
STRIPE_WEBHOOK_SECRET_PORTILLO=whsec_4ojzPB1t6MRqWQUV3CefMBbJAGqKmWEH
```

## Verificar / pendientes

- [ ] Verificar que slug en `UsuariosPage.jsx:51` use `asesorias-suiza` (plural) y no la versión singular legacy
- [ ] Lukas — añadir Stripe propio si vende directo o usa el de Portillo
- [ ] Confirmar que el portal `/portal/asesorias-suiza` sirve OTPs desde la cuenta Resend correcta (1 Portillo, 2 Lukas)
- [ ] Cuando llegue el segundo multi-owner: refactor a `clients.config.multi_owner = { enabled: true, owners: [...] }`

## Sprint 2026-05-08 — Booking sync recuperado + nueva pipeline cierre

**Contexto:** durante 30+ días se acumularon **49 bookings con `crm_contact_id = NULL`** porque (a) `BookPublic` llamaba al webhook con el `host` correcto, (b) los 6 booking_hosts tenían `target_pipeline_slug` y `target_stage_key` vacíos, y (c) el fallback hardcoded apuntaba a `Europeos a Suiza` (que ya no existe — tras el rename a `Europeos a Suiza Portillo` y `Europeos a Suiza · Lukas`). El endpoint devolvía 500 silenciosamente. Pérdida estimada: ~50 leads/mes sin trackear.

**Acciones aplicadas (Dashboard-Ops, branch main):**

1. **Pipeline nueva** `Cierre Closers · Portillo` (id `a74ecfdf-c2b8-4273-8a55-577d585f51c4`, owner_scope=portillo, operator_id=null compartida). Stages: `llamada_agendada → primer_contacto → en_conversacion → seguimiento_cierre → cliente | ghosting | descartado | descalificado`. Aloja a Adrián (closer principal, 19 leads), Toñi (2) y Jose (0 hoy).
2. **Armonización keys Lukas Seguros** en pipeline `Clientes Potenciales Seguros`: `contacted → llamada_agendada`, `stage_1777463365783 → sin_anmeldung`, `stage_1777462703000 → con_anmeldung`. Pipeline tenía 0 contactos así que no había drift.
3. **Routing por host** seteado en `booking_hosts.target_pipeline_slug` + `target_stage_key`:
   - `aterrizaje` (Mio/Portillo) → `Europeos a Suiza Portillo` / `llamada_agendada`
   - `seguros` (Portillo) → `Clientes Potenciales Seguros - Portillo` / `llamada_agendada`
   - `toni-terron`, `adrian`, `jose-fernandez` → `Cierre Closers · Portillo` / `llamada_agendada`
   - `lukas-seguros` → `Clientes Potenciales Seguros` / `llamada_agendada`
4. **Endpoint refactor** (`api/asesoria-suiza-booked.js`): fallback ya no es un nombre hardcoded inexistente sino una **tabla por operator_slug** (portillo→Europeos a Suiza Portillo, lukas→Clientes Potenciales Seguros, closers→Cierre Closers). Además ahora **linkea `bookings.crm_contact_id`** tras crear/actualizar el contacto (cierra la fuga histórica), y populariza `crm_contacts.assigned_closer` con el slug del operador.
5. **`BookPublic.jsx`** envía `booking_id` al webhook → link directo sin lookup.
6. **Backfill** de los 49 históricos vía `scripts/backfill_asesorias_suiza_bookings.mjs` con alias para hosts legacy `analisissuiza`/`analisisuiza` → `lukas-seguros`. Resultado: **49/49 linkeados**, 21 contactos en Cierre Closers (19 adrian + 2 toni-terron), 27 en Lukas Seguros, 1 en Seguros Portillo.

**Scripts añadidos** (idempotentes, ambos con `--dry-run`):
- `scripts/setup_asesorias_suiza_workflow_routing.mjs`
- `scripts/backfill_asesorias_suiza_bookings.mjs`

**Important gotcha encontrado:** `crm_contacts` **no tiene columna `operator_id`** — la atribución se hace por `assigned_closer` (string slug). El operator_id sí existe en `crm_pipelines`, `booking_hosts`, `team`, `user_integrations`, `sales`, pero no en contacts. Si en el futuro consolidamos a `operator_id` en contacts, hay que migrar `assigned_closer` → FK.

**Pendiente del audit (Sprint 2+):**
- [ ] Workflows lifecycle booking: cancelado / reschedule / no-show automático (vía `bookings.status` change + cron horario)
- [ ] Tabla `tracking_events` + endpoint `/api/track` para CAPI server-side a Meta/GA4/TikTok (pixel cliente + server híbrido)
- [ ] Pixel cliente en webs de seguros (Lukas + Portillo) cuando estén listas
- [ ] Workflow Stripe pago fallido → activity + WA recovery (hoy se ignoran)
- [ ] `crm_contacts` cleanup duplicados por `(client_id, email)` — issue preexistente del CRM
- [ ] Borrar booking_hosts legacy `analisissuiza`/`analisisuiza` (tras 30 días sin tráfico nuevo)

**Métrica futura recomendada:** dashboard "leads → llamadas → asistidas → ventas" por `assigned_closer` para Adrián/Toñi/Jose, alimentado por `crm_contacts.fecha_llamada` + `bookings.status` + `sales`.

Ver [[Workflow-Audit-Asesoria-Suiza-2026-05-08]] (a crear) para el catálogo completo de 30+ workflows propuestos en el audit.

## Conexiones

- Tipo: [[Tipos-de-Cliente]] · [[Multi-Owner]]
- Integraciones: [[Stripe]] · [[Resend]] · [[Google]] (GCal)
- Plataforma: [[Modulo-Formacion-Infoproducto]] · [[Portal-Infoproducto]]
- Decisiones: [[Decision-Multi-Owner-Generic]] (pendiente)
