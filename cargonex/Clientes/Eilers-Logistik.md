---
tags: [cliente, logistica, cargonex, dsgvo]
slug: EilersLogistik
client_type: logistica
status: live
platform: cargonex
created: 2026-05-07
---

# Eilers Logistik GmbH

Cliente fundador de [[Cargonex]]. **Importante**: NO es un tenant de [[Dashboard-Ops]] — corre en una plataforma distinta (`app.cargonex.co`), repo distinto, base de datos distinta. Si buscas un cliente en `central.blackwolfsec.io/<slug>`, este no aparece.

## Identidad

- **Empresa**: Eilers Logistik GmbH (Alemania)
- **Slug en Cargonex**: `EilersLogistik` (CamelCase, no kebab-case como en Dashboard-Ops)
- **clientId**: `cli_eilers0001`
- **Plan**: `enterprise`
- **Status**: `active`
- **Marca de legacy**: `legacy: true` — usa `data/db.json` y endpoint `/api/logs` directamente. NO usa `tenant-<id>.json` ni `/api/tenant-logs` como los tenants nuevos.

## URL del cliente

`https://app.cargonex.co/EilersLogistik` — aquí entra el cliente. La SPA detecta `legacy:true` vía `window.__CARGONEX_TENANT__` y bifurca hacia el endpoint legacy.

## Datos en producción (snapshot 2026-05-07)

| KPI | Valor |
|---|---|
| Drivers (Fahrer) | 231 |
| Vehicles (Fahrzeuge) | 162 |
| Records totales | 8 343 |
| Last sync | 2026-05-07 17:38 UTC |

Histórico voluminoso: la razón de `legacy:true`. Migrar Eilers a `tenant-cli_eilers0001.json` requiere migración de datos no trivial → no se ha hecho, no es prioritario.

## Verticales / módulos en uso

- **Dispatcher** — KPIs por turno, export Schicht
- **Fahrer Management** — 231 drivers, daños, tiempos, ausencias
- **Fahrzeuge** — 162 vehículos, mantenimiento, OBD2, fleet docu, ÜB photos
- **Tender** — calculadora de tarifas (iframe-based)
- **Tour Planner** — AI gateway con [[Cargonex#Stack]] (Anthropic claude-haiku-4-5)
- **Push Notifications** — VAPID web-push para drivers
- **i18n** — DE primary, EN/ES disponibles (cobertura completa todas las stations DNM4/DNM6/DNW9/HNM6/DHL/FEDEX/JDHH)
- **Auth** — email-OTP por defecto, password opcional

## Particularidades técnicas

- **Datos en `db.json`**, NO en `tenant-cli_eilers0001.json`. El método `getTenantDataFile()` retorna `db.json` cuando `tenant.legacy === true`.
- **Bootstrap automático**: cada arranque del contenedor `eilers-portal`, el log dice `[bootstrap] Eilers Logistik tenant ensured (slug: /EilersLogistik)`. Idempotente.
- **Roles**: el frontend usa `SUPERADMIN` (no `ADMIN`). Los `roleEmails` deben mapear a un rol que existe o el login carga blank. Ver [[Portal-Role-Names]] (memoria proyecto).

## Integraciones

| Integración | Estado | Detalle |
|---|---|---|
| SMTP / Resend | 🟡 cutover en progreso | Verificación de dominio `cargonex.co` pendiente. Hoy SMTP genérico |
| Cloudflare Tunnel | ✅ live | `app.cargonex.co` apunta al VPS |
| Anthropic | ✅ live | Tour planner (`claude-haiku-4-5`) |
| Postgres | ✅ live | KV + photos, backups diarios |
| WhatsApp | ❌ no aplica | Cargonex no integra WA (a diferencia del patrón [[Hugo-EnFormaConHugo]]) |
| Stripe | ❌ no aplica | Eilers paga por contrato anual offline. Sin billing automatizado |

## DSGVO / GDPR

🚨 **Bloqueante para go-live formal**: Eilers es alemán, GDPR es no-negociable. Pendiente:

- [ ] DPA (Data Processing Agreement) firmado con Eilers
- [ ] Aviso de privacidad publicado en `web.cargonex.co/privacy`
- [ ] Documentación de subprocessors (Cloudflare, Anthropic, Resend)
- [ ] Procedimiento de export/delete por sujeto (Art. 15 + 17)
- [ ] Logs de auditoría retentivos ≥ 1 año

Ver [[Politica-de-Secretos]] para la parte de exposure de claves — relacionada porque DSGVO exige report de breach en 72 h.

## Riesgos / Gotchas

1. **Migración legacy → multi-tenant pendiente** — si Cargonex escala, Eilers seguirá en `db.json` mientras los demás están en `tenant-<id>.json`. Heterogeneidad operativa permanente. Aceptable hoy, doloroso si se quiere bulk export/import cross-tenant.
2. **Datos en volumen Docker** — backup del volumen `eilers-data` ES el plan de recovery. Si se pierde el volumen y no hay snapshot de host, se pierde todo. El contenedor `eilers-backup` solo dumpea Postgres, NO el JSON volume.
3. **`PLATFORM_ADMIN_SECRET = testtoken123`** en producción mientras esto no se rote, cualquiera con ese string toma control de los datos de Eilers desde `/admin → DB Inspector`. Ver [[Bugs-Criticos]] · [[Playbook-Rotar-Secret]].
4. **Mobile PIN flow no probado** en device real — solo smoke local en :3099. Cliente puede reportar fallo en producción.
5. **i18n key=role**: si un role nuevo se mete y no se mapea en `roleEmails`, el login del usuario aterriza en blank. Verificar contra [[Portal-Role-Names]] siempre que se toque la matriz de roles.

## Conexiones

- Plataforma: [[Cargonex]] · [[Arquitectura]]
- Onboarding (cómo se haría hoy): [[Playbook-Crear-Tenant-Cargonex]]
- Decisión arquitectónica que afecta: [[ADR-Cargonex-Stack-Separado]]
- Otro vertical logística (en Dashboard-Ops, no Cargonex): [[IC-Logistics]] — ojo, NO confundir
- Compartiendo host: [[Host-Docker-Layout]]
