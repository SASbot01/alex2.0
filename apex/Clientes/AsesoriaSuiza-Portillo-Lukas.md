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

## Conexiones

- Tipo: [[Tipos-de-Cliente]] · [[Multi-Owner]]
- Integraciones: [[Stripe]] · [[Resend]] · [[Google]] (GCal)
- Plataforma: [[Modulo-Formacion-Infoproducto]] · [[Portal-Infoproducto]]
- Decisiones: [[Decision-Multi-Owner-Generic]] (pendiente)
