---
tags: [sesion, conversacion, autonoma]
date: 2026-05-08
duration: ~1.5 horas (sesión nocturna autónoma)
created: 2026-05-08
---

# Sesión 2026-05-08 noche — Trabajo autónomo desktop + sprint 0

Sesión de trabajo autónomo mientras Alejandro dormía. Allowlist en `.claude/settings.local.json` permitía git/curl/edits sin pedir aprobación.

## Lo que se hizo

### Branch `feat/desktop-app` en Dashboard-Ops- (1 commit, `6c98683`)

**Bootstrap completo de Tauri sobre Dashboard-Ops** desde rama main limpia.

Estructura nueva:
- `src-tauri/Cargo.toml` — deps Tauri 2 + plugins (store, shell, updater, notification, dialog)
- `src-tauri/tauri.conf.json` — config base (window 1440×900, identifier base)
- `src-tauri/src/main.rs` + `src/lib.rs` — entry + comando `get_tenant_info`
- `src-tauri/build.rs`
- `src-tauri/.gitignore`
- `src-tauri/tenants.json` — metadata de 7 tenants (enformaconhugo, asesorias-suiza, fba-academy, yc-logistics, creator-founder, detras-de-camara, black-wolf) con productName, identifier, icon source, primaryColor, loginUrl, binaryName

Scripts de automatización:
- `scripts/build-tenant.mjs` — patcha `tauri.conf.json` per-tenant + cargo tauri build con env vars `TENANT_SLUG`, `TENANT_NAME`, `TENANT_LOGIN_URL`
- `scripts/generate-icons.mjs` — genera `.icns` (Mac), `.ico` (Win), PNGs varios tamaños desde `_logoSource` de cada tenant via ImageMagick + iconutil

Frontend nuevo:
- `src/utils/desktopBridge.js` — abstracción web↔Tauri:
  - `bridgeSet/Get/Remove/Clear` (wraps localStorage o tauri-plugin-store según entorno)
  - `getTenantInfo()` — lee tenant baked-in via comando Rust
  - `openExternal/notify` — cross-platform helpers
  - `checkForUpdate` — wrapper plugin-updater
- `src/pages/Download.jsx` — página `/download` y `/<slug>/download`:
  - Detecta OS del visitante
  - Botón principal a binario correcto del tenant
  - Banner UX con guía paso a paso para abrir sin firma (Mac click derecho > Abrir, Win More info > Run anyway)
  - Pulla desde GitHub Releases públicos (RELEASE_TAG configurable)

CI/CD:
- `.github/workflows/desktop-release.yml` — matrix multi-tenant
  - 7 tenants × 2 OS = 14 binarios por release
  - Cache cargo + pnpm
  - Steps SIGNING comentadas (activar cuando lleguen Apple Dev + Win cert)
  - Trigger: tag `v*` o workflow_dispatch manual

Cambios mínimos a archivos existentes (sin afectar deploy Vercel):
- `package.json`: scripts `tauri:*` + deps Tauri
- `vite.config.js`: detección isTauri + clearScreen + HMR via WS dedicado
- `src/main.jsx`: rutas `/download` y `/<slug>/download` (antes del catch-all)

Documentación:
- `docs/DESKTOP.md` — manual completo para devs

**Sigue pendiente** (no toqué para evitar romper):
- Iconos reales per-tenant (hoy default a blackwolf.png)
- Migrar tokens localStorage → desktopBridge en `portalAuth.js` y `utils/auth.js` (162 referencias en 49 archivos — trabajo grande)
- Endpoint `/desktop/latest.json` para auto-updater
- Apple Developer + Win cert (decisión: diferir hasta validar uso interno)
- Generar private key del updater con `tauri signer generate`

PR ready: https://github.com/aatshadow/Dashboard-Ops-/pull/new/feat/desktop-app

### Branch `feat/sprint0-internal` en monorepo ejambre (1 commit, `4397db7`)

3 fixes críticos del Sprint Arreglos que NO requerían acceso externo:

1. **C4 — `JWT_SECRET` validation en boot** (`enjambre-api/src/auth/auth.js`)
   - Antes: `const JWT_SECRET = process.env.JWT_SECRET || 'default-secret'` → tokens trivialmente falsificables si la env var faltaba.
   - Ahora: crash en boot si falta o `len < 32`. Mensaje claro con `openssl rand -hex 64`.
   - Bonus: `ADMIN_PASSWORD` ahora puede venir de env (antes hardcoded).

2. **Cleanup PUBLIC_PATHS** (`enjambre-api/src/auth/auth.js`)
   - Quitado `/api/webhooks/whatsapp` — endpoint no existe en `routes/index.js`, era whitelist muerto.
   - Añadido skip de `/api/portal/*` para que cuando se mergee feat/portal-infoproducto, su `authPortal` middleware propio no choque con el global.

3. **C3 — `/api/webhooks/central` firma obligatoria** (`enjambre-api/src/routes/index.js`)
   - Antes: cualquier IP podía inyectar leads/ventas arbitrarios.
   - Ahora: exige `x-webhook-secret` matching `CENTRAL_WEBHOOK_SECRET` env var. Sin env var → 503.
   - Pendiente acción humana: setear `CENTRAL_WEBHOOK_SECRET` y coordinar con quien envía al webhook.

4. **CORS Tauri origins** (`enjambre-api/src/server.js`)
   - Whitelist permanente de `tauri://localhost` y `https://tauri.localhost` para que la app desktop pueda llamar a la API independientemente de `CORS_ORIGINS`.

PR ready: https://github.com/SASbot01/ejambre/pull/new/feat/sprint0-internal

### Lo que NO se hizo (intencional)

Estas 4 tareas del Sprint 0 quedan pendientes de Alejandro porque requieren:

1. **Rotar las 7 API keys exposed** — necesita TUS dashboards externos (Stripe, Resend, Anthropic, Supabase, Cloudflare).
2. **Aplicar Migration 018 RLS Fase A** — toca prod DB, requiere verificación pre-deploy.
3. **Renumerar migrations 045-048** — Supabase migrations CLI puede tratar archivos renamed como nuevas migrations. Necesita verificación de la tabla de migrations aplicadas antes de rename.
4. **Quitar `localStorage('bw_superadmin')`** — requiere endpoint nuevo `/api/admin/verify` + refactor frontend, trabajo más grande.

## Acciones humanas requeridas antes del próximo deploy de enjambre-api

Si vas a deployar el branch `feat/sprint0-internal` (o cuando se mergee a main), **el server crashea en boot si**:

```env
# enjambre-api/.env — añadir si no existen
JWT_SECRET=<openssl rand -hex 64>          # CRÍTICO — antes default-secret
CENTRAL_WEBHOOK_SECRET=<openssl rand -hex 32>  # NUEVO
# ADMIN_PASSWORD=...                       # Opcional, fallback a hardcoded
```

Y coordinar con quien envía al `/api/webhooks/central` para que añada `x-webhook-secret: <CENTRAL_WEBHOOK_SECRET>` en sus requests, o el endpoint devolverá 401.

## Riesgos / decisiones tomadas

- **No toqué los archivos `migrations/04*` duplicados** — el rename es delicado y no encajaba en autonomía nocturna. Tarea sigue viva en sprint Arreglos.
- **No instalé Rust localmente** — el server no lo tenía. Setup manual de `src-tauri/` sin Tauri CLI. Cuando Alejandro tenga su Mac, instala `rustup` + `pnpm` y arranca con `pnpm tauri:dev` (build local) o deja que GitHub Actions builde en CI.
- **Iconos genéricos por defecto** — varios tenants apuntan a `blackwolf.png` en su `_logoSource` porque no encontré logos dedicados. Tenants con logo real (fba-academy, creator-founder, yc-logistics) sí tienen su path.
- **Allowlist más restrictivo que `--dangerously-skip-permissions`** — bloqueé `rm -rf`, `git push --force`, `git reset --hard`, `DROP TABLE`, `DELETE FROM clients/team/crm_contacts`. Sesión segura.

## Siguiente paso

Cuando Alejandro despierte:

1. Revisar PR `feat/desktop-app` en Dashboard-Ops- — el más sustancial (~1400 LOC nuevos).
2. Revisar PR `feat/sprint0-internal` en ejambre — corto pero crítico para seguridad.
3. **Decidir orden de merge**:
   - Recomendación: primero `feat/sprint0-internal` (cierra bugs críticos), después `feat/desktop-app` (feature nueva).
   - Antes de mergear `feat/sprint0-internal`: setear `CENTRAL_WEBHOOK_SECRET` en `.env` o el webhook se rompe en producción.
4. Actualizar tareas Sprint 0 en `central.blackwolfsec.io/black-wolf/task-management` a `done` para las 3 que se cerraron.

## Conexiones

- [[Sprint-Desktop-App]]
- [[Sprint-0-Higiene]]
- [[Sprint-Arreglos]]
- [[Bugs-Criticos#C3]] · [[Bugs-Criticos#C4]]
- [[Sesion-2026-05-07]] (sesión anterior, contexto Portal Infoproducto)
- [[HOME]]
