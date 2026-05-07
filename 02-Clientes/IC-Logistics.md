---
tags: [cliente, logistica]
slug: yc-logistics
client_type: logistica
status: live
created: 2026-05-07
---

# IC Logistics

Cliente vertical logística. Slug en DB es `yc-logistics` (typo histórico — no es `ic-logistics`). Tema light forzado.

## Identidad

- **Slug DB**: `yc-logistics` (typo histórico que se conserva)
- **clientType**: `logistica`
- **Idiomas**: `en` + `zh` (chino simplificado)
- **Tema**: 🌞 LIGHT forzado (en `FORCE_LIGHT_SLUGS` en ClientApp.jsx y `portalTheme.js`)

## Productos / oferta

- Bootcamp logística China
- Servicios de logística empresarial

## Integración landing → CRM

Landing del bootcamp China integrada al CRM via `POST /api/forms/icl-submit`. El form en la landing → endpoint en enjambre-api → crea lead en pipeline IC Logistics Web.

## Pendientes

- Calculadora de presupuesto integrada a la landing — pendiente
- Más pipelines según vertical de servicio

## Particularidades técnicas

- **Light theme forzado** — ver [[Sistema-Theming]] · `FORCE_LIGHT_SLUGS = new Set(['yc-logistics'])` en `ClientApp.jsx:272` y `portalTheme.js`
- **Bilingüe en+zh** — usa el patrón ad-hoc `L(es, en, zh)` con `zh` poblado
- **Toggle theme oculto** — `lightOnly` en `MyProfilePage.jsx` para no permitir cambio del usuario

## Conexiones

- Tema: [[Sistema-Theming]]
- Forms: [[Webhooks]] (`/api/forms/icl-submit`)
- Idioma: [[Sistema-i18n]]
- Ejemplo de tenant que rompió el portal antes del fix del 2026-05-07 (theme dark hardcoded en PortalLogin/Hub) — ver [[Sesion-2026-05-07]]
