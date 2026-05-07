---
tags: [auditoria, zombie, cleanup]
created: 2026-05-07
---

# Código Zombie

Lo que sobra y se puede borrar. Cada item viene con la justificación de por qué es seguro eliminarlo.

## ManyChat — retiro incompleto

**Estado**: memoria del proyecto dice "DB limpia + header actualizado tras retiro". Pero quedaron vivos:

- `enjambre-api/src/routes/index.js`: webhook `/api/webhooks/manychat` sigue expuesto y procesa requests
- `Dashboard-Ops-/api/send-email.js`: proxy ManyChat (POST a `https://api.manychat.com`)
- `Dashboard-Ops-/api/my-integrations.js`: método `manychat()` para validar API key
- `Dashboard-Ops-/src/utils/data.js` (god file): `getManychatConfig()`, `saveManychatConfig()`, `syncManychatSubscribers()`
- Tabla `manychat_config` (vacía pero existe)

**Fix**:
1. Borrar el route handler del webhook
2. Quitar el método de send-email.js
3. Limpiar helpers en data.js
4. `DROP TABLE manychat_config;` (con backup)
5. Verificar en sistemas externos (Stripe? algún script?) que ningún webhook sigue apuntando aquí

## Discord — bot dual

**Estado**: hay dos bots
- `enjambre-api/src/connectors/discord.js` (Node, deprecated, fail-safe si dependencia falta)
- `/agente-discord/` (Python, sidecar en puerto 8788, último log 2026-04-22)

**Fix**: decidir cuál mantener. Si se queda Python, quitar `discord.js` de package.json + borrar el connector Node.

## `/api/webhooks/whatsapp` whitelisteado pero no existe

**Estado**: `enjambre-api/src/auth/auth.js:41` tiene `/api/webhooks/whatsapp` en `PUBLIC_PATHS` (sin auth), pero el endpoint **no existe** en `routes/index.js`. Whitelisting muerto.

**Fix**: quitar la entrada de `PUBLIC_PATHS`.

## Tabla `general_orders` referenciada y no creada

**Estado**: `enjambre-api/src/army/general.js` referencia tabla `general_orders` para auditar órdenes de Donna, pero la tabla **nunca se creó**. Si Donna mete `INSERT INTO general_orders`, falla silenciosamente.

**Fix opciones**:
- Crear migration con la tabla (recomendado, es útil)
- O quitar las referencias del código

## 16 tablas DB no usadas

Tablas creadas en migrations sin referencias en `/api/` ni `src/`:

- `bw_client_projects`, `bw_client_deliverables`, `bw_onboarding_steps`, `bw_support_tickets`, `bw_ticket_messages` (5)
- `chat_broadcasts`, `chat_contacts`, `chat_conversations`, `chat_flows`, `chat_messages` (5)
- `feedback_form_config`, `weekly_feedback_responses`, `weekly_feedback_summaries` (3) — aunque sí hay `weekly-feedback.js` activo, ¿qué consume estas?
- `task_pipelines`, `task_stages` (2)

**Fix**:
1. `COMMENT ON TABLE <name> IS 'LEGACY: ...'` documentando estado
2. Para las claramente muertas: backup + DROP en migration nueva
3. Para las "futuras planeadas": dejar pero documentar y poner deadline

## Scripts de diagnóstico en repo de prod

`Dashboard-Ops-/scripts/check_fba_google.mjs`, `check_google_auth.mjs`, `check_google_email.mjs`, `check_google_email2.mjs`, `oauth_drive_consent.mjs`. Mezclados con código de producción en el git tree.

**Fix**: mover a `Dashboard-Ops-/.dev/` o `tools/` y añadir al `.gitignore` si son scratch personal. Si son útiles, documentar en README cuándo usar cada uno.

## TENANT_LOGOS duplicado

Antes de los fixes de hoy, `TENANT_LOGOS = { 'fba-academy': '...' }` vivía hardcoded en 3 archivos: `BookPublic.jsx`, `PortalLogin.jsx`, `PortalHub.jsx`. Hoy se centralizó en `src/pages/portal/portalTheme.js` para PortalLogin/Hub, pero **BookPublic todavía tiene su copia local**.

**Fix**: extraer también desde BookPublic.jsx → import desde portalTheme. O mover portalTheme.js a `src/constants/tenantLogos.js` con scope más genérico.

## Wwebjs cache + auth runtime

`enjambre-api/.wwebjs_cache/` y `.wwebjs_auth/` viven en el filesystem del repo (auth ya está en .gitignore por commit `46f8e02`, cache es legacy de wwebjs).

**Fix**: confirmar que ambos están en `.gitignore`. Limpiar cache periódicamente (cron mensual).

## Conexiones

- Limpieza: [[Sprint-7-DX-Cleanup]]
- Por qué pasó: poco rigor en cleanup post-feature-flag-off
- Política: [[Naming-Conventions]] (commit conventions para deprecation)
