---
tags: [playbook, cliente, onboarding]
created: 2026-05-07
---

# Playbook — Onboarding Cliente Nuevo

Patrón Fase 0-5 destilado de los 7 onboardings hechos hasta hoy.

## Fase 0 — Pre-onboarding (con el cliente)

- [ ] Llamada estratégica: qué vende, qué pipelines necesita, qué integraciones
- [ ] Decidir [[Tipos-de-Cliente]]: `growth`, `consultoria`, `software_developing`, `manufactura`, `logistica`, `admin`
- [ ] Definir si es [[Multi-Owner]] (asesoria-suiza es el primer caso)
- [ ] Idiomas operativos (es / en / zh)
- [ ] Slug del tenant (lowercase, kebab-case, ej: `cliente-nuevo`)

## Fase 1 — DB tenant

```sql
INSERT INTO clients (slug, name, client_type, language, config) VALUES (
  'cliente-nuevo',
  'Cliente Nuevo',
  'growth',
  'es',
  '{"features": {"crm": true, "sales": true, "formacion": true, ...}}'::jsonb
);
```

- [ ] Crear `clients` row
- [ ] Crear pipelines iniciales en `crm_pipelines` con stages JSONB
- [ ] Si multi-owner: poblar `client_operators` y/o `clients.config.multi_owner`

## Fase 2 — Auth y team

- [ ] Crear primer usuario en `team` (CEO/director)
- [ ] Generar password inicial + comunicar
- [ ] Confirmar login en `central.blackwolfsec.io/<slug>`
- [ ] Confirmar role correcto y permisos efectivos

## Fase 3 — Integraciones

Por cada integración que aplique:

### Resend (email per-tenant)

- [ ] Crear API key en Resend dashboard del tenant
- [ ] Verificar dominio (`from_email`)
- [ ] Insertar en `email_config` (con `account_index=1`, o más si multi-owner)

### Stripe (billing)

- [ ] Si el cliente cobra a sus alumnos por Stripe → conectar su cuenta Stripe Connect
- [ ] Configurar webhook endpoint en Stripe dashboard del cliente
- [ ] Meter `STRIPE_WEBHOOK_SECRET_<slug>` en `enjambre-api/.env`
- [ ] Test con venta de prueba

### WhatsApp

- [ ] Sesión nueva en `whatsapp-web.js` → escanear QR con número del cliente
- [ ] Validar que mensajes entrantes generan contactos en CRM
- [ ] (futuro) — sustituir por WhatsApp Business API cuando llegue [[Sprint-6-WhatsApp-BA]]

### Google (si bookings)

- [ ] OAuth flow con cuenta del director
- [ ] Validar `freebusy` query funciona
- [ ] Configurar en `user_integrations` (service='google')

### Hotmart (si aplica)

- [ ] Pendiente — scaffolding sin construir. Ver [[Hotmart]].

### Meta Ads (si marketing activo)

- [ ] Token long-lived en `ceo_integrations` (service='meta_ads')
- [ ] Validar `accountInsights` devuelve datos

## Fase 4 — Marketing assets

Memoria recuerda: las URLs de landings, grupos WhatsApp, Zoom, replays viven en **Supabase `info_producto_assets`**, NO en el repo.

- [ ] Crear rows en `info_producto_assets` con landings del cliente
- [ ] Subir logos a Storage (bucket assets)
- [ ] Si es tenant con portal público → poblar `TENANT_LOGOS` en `portalTheme.js` con la URL del logo
- [ ] Si tema light forzado → añadir slug a `FORCE_LIGHT_SLUGS` (ClientApp.jsx + portalTheme.js)

## Fase 5 — Validación E2E

- [ ] Login del director
- [ ] Crear contacto manual en CRM
- [ ] Crear venta de prueba (Stripe test mode)
- [ ] Mandar email transaccional con Resend
- [ ] Si bookings: agendar reunión y confirmar GCal sync
- [ ] Si formación: crear ruta + formation + módulo + lección de prueba
- [ ] Smoke test del portal público si aplica (`/portal/<slug>`)

## Gotchas aprendidos

- **Slug typos**: `yc-logistics` se quedó así por typo histórico. Usa el real, no el "esperado".
- **Multi-owner hardcoded**: hoy solo asesorias-suiza. Si llega otro, hay que refactorizar (ver [[Decision-Multi-Owner-Generic]]).
- **Vercel commit author**: solo `SASbot01` o `aatshadow` o se rechaza el deploy. Ver [[Naming-Conventions]].
- **Stages JSONB**: `crm_pipelines.stages` es jsonb, `crm_contacts.stage_key` apunta a la `key` del stage. Sin schema validation hoy — cuidado con typos.
- **light-only**: `FORCE_LIGHT_SLUGS` en 2 archivos (ClientApp + portalTheme). Sincronizar.

## Conexiones

- Tipos: [[Tipos-de-Cliente]] · [[ClientTypes]]
- DB: [[Database-Supabase]] · [[Migrations]]
- Integraciones: [[_moc/MOC-Integraciones]]
- Casos vivos: [[_moc/MOC-Clientes]]
