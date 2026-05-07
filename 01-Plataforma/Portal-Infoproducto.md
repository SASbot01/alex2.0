---
tags: [plataforma, modulo, portal, infoproducto]
status: pending-merge
created: 2026-05-07
---

# Portal Infoproducto

Sistema de login OTP por email para alumnos de tenants con infoproducto. URL pública: `central.blackwolfsec.io/portal/<slug>`.

## Por qué

Hoy los alumnos de los infoproductos de los clientes (Hugo, Asesoría Suiza, Abel, etc.) **no tienen sitio donde entrar**. Dashboard-Ops es para el team del cliente. El alumno no tiene login. El portal cierra ese gap.

## Flujo

```
Alumno entra a /portal/<slug>
  ↓
Mete email
  ↓
POST /api/portal/otp/request
  ↓ (si email está en portal_users.allowlist activa)
Recibe código 6 dígitos por email (Resend del tenant)
  ↓
Mete código → POST /api/portal/otp/verify
  ↓
JWT en localStorage (slot portal_token_<slug>, TTL 30 días)
  ↓
Navega a /portal/<slug>/hub (autenticado)
```

## Componentes

### DB (migration 060)

- `portal_users` — allowlist (email, name, owner_scope, email_account_index, active)
- `portal_otp_codes` — códigos vivos (sha256, TTL 10 min, max 5 intentos)
- RLS deny-all desde anon — solo enjambre-api con SERVICE_ROLE

### Backend (`enjambre-api/src/routes/portal.js`)

- `POST /api/portal/otp/request` { slug, email }
- `POST /api/portal/otp/verify` { slug, email, code }
- `GET /api/portal/me` (Bearer JWT, revalida en DB)
- `GET /api/portal/admin/users?clientSlug=…`
- `POST /api/portal/admin/users`
- `PATCH /api/portal/admin/users/:id`
- `DELETE /api/portal/admin/users/:id`

Env vars necesarias: `PORTAL_JWT_SECRET` (crash si falta o len < 32 — fix aplicado en sesión).

### Frontend público

- `src/pages/portal/PortalLogin.jsx` — pantalla de login (email → OTP → success)
- `src/pages/portal/PortalHub.jsx` — pantalla autenticada (header + tabs Formación/Marketplace **placeholders**)
- `src/pages/portal/portalTheme.js` — TENANT_LOGOS + getPortalPalette + FORCE_LIGHT_SLUGS
- `src/utils/portalAuth.js` — JWT helpers

### Frontend admin (`/<slug>/infoproducto/usuarios`)

- `src/pages/infoproducto/InfoproductoHome.jsx` — landing del módulo (cards: Formación, Marketplace, Comunidad, Mentorías, Usuarios)
- `src/pages/infoproducto/UsuariosPage.jsx` — CRUD de allowlist con modal "Añadir usuario"

## Estado (2026-05-07)

✅ **Hitos 1-5 mergeables**:
- 5 commits en `feat/portal-infoproducto` de Dashboard-Ops-
- 3 commits en `feat/portal-infoproducto` de ejambre (+ 4 commits army arrastrados)
- Auditoría 10 bugs encontrada y corregida (commit `f7d883f` y `b2dad22`)

🟡 **Pendiente para go-live**:
1. Mergear ejambre PR #1: https://github.com/SASbot01/ejambre/pull/1
2. `PORTAL_JWT_SECRET=$(openssl rand -hex 64)` en server `.env`
3. Reiniciar enjambre-api
4. Mergear Dashboard-Ops- PR #22: https://github.com/aatshadow/Dashboard-Ops-/pull/22
5. Smoke test E2E

🔴 **Hito 6 NO HECHO** — cableado del contenido real en los tabs Formación/Marketplace del PortalHub. Hoy los tabs son placeholders. Sin Hito 6, el alumno entra y NO ve nada útil.

Para Hito 6 hace falta:
- Modo "público" en `TrainingHome` / `CursoDetail` / `LessonViewer` que no dependa de ClientContext ni de team auth
- Modo "público" en `MarketplaceHome` (si se quiere exponer)
- Decisión de producto: ¿el alumno ve TODO lo publicado o filtramos por cohorte/tag? Hoy NO hay relación entre `portal_users` y `cursos/lecciones`

## Deuda técnica conocida

- **Sin rate limiting** en `/api/portal/otp/request` → spam de mails posible
- **Auth real en endpoints admin** — hoy son anon, mismo patrón que MyIntegrations/FeedbackAdmin
- **JWT TTL 30 días sin revocation** — si admin desactiva alumno, /me revalida (fix aplicado), pero el alumno con JWT viejo aún puede hacer requests a otros endpoints hasta que /me los devuelva 401
- **Cleanup de `portal_otp_codes` expirados** sin cron
- **Logout button** en PortalHub falta
- **Multi-owner config dinámico** — hoy `asesorias-suiza` hardcoded
- **TENANT_LOGOS** centralizados pero BookPublic.jsx aún tiene su copia local

## Conexiones

- Sesión donde se construyó: [[Sesion-2026-05-07]]
- Cliente más interesado: [[AsesoriaSuiza-Portillo-Lukas]] · [[Hugo-EnFormaConHugo]] · [[FBA-Academy]]
- Auditoría sobre los componentes: [[Bugs-Criticos]]
- Plataforma: [[Modulo-Formacion-Infoproducto]]
