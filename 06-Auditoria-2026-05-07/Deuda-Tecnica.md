---
tags: [auditoria, deuda]
created: 2026-05-07
---

# Deuda Técnica

Lo que no rompe en producción HOY pero te va a hacer pagar más cada mes.

## God files (frontend)

### ClientApp.jsx — 1573 LOC, 336 rutas, 120 lazy-loads

El router monolítico de Dashboard-Ops. Cada cambio tiene blast-radius incierto. Tiene rutas duplicadas en root vs MiniLayouts, condiciones por `clientType` inline (20+), feature flags inline (30+).

**Fix**: dividir en sub-routers por módulo. `CrmRouter.jsx`, `StoresRouter.jsx`, `MarketingRouter.jsx`. Usar `<Outlet>` para nested. Cada sub-router < 200 LOC.

**Esfuerzo**: ~3 semanas. Plan en [[Sprint-7-DX-Cleanup]].

### data.js — 3649 LOC, 30+ funciones

God file con `getDashboard`, `getSales`, `getCrmContacts`, `getStores`, etc. todo mezclado. Cinco archivos hacen `import * as dataUtils from '../utils/data'` arrastrando los 3649 LOC.

**Fix**: modularizar a `data/sales.js`, `data/crm.js`, `data/teams.js`, `data/stores.js`. Barrel export en `data/index.js`. Tree-shaking funcional.

**Esfuerzo**: 2 semanas. Plan en [[Sprint-7-DX-Cleanup]].

### CrmPage.jsx — 4524 LOC, 9 modales inline

Mega-componente. 11 definiciones de la función `L()` para i18n en distintos scopes. Modales (PipelineEditor, CustomFields, NewContact, SaleRegistration, EmailCompose, etc.) viven inline.

**Fix**: sacar cada modal a `components/Modal*.jsx`. Dividir CrmPage en `CrmContactsView`, `CrmPipelinesView`, `CrmActivitiesView`. Cada uno <500 LOC.

**Esfuerzo**: 2-3 semanas.

## i18n duplicado 41 veces

`const L = (es, en, zh) => zh ? (zh ?? en) : en ? en : es` aparece **41 veces** en el codebase. Solo CrmPage.jsx tiene 11 definiciones.

**Fix**: extraer a `src/hooks/useL.js` que lee `en` y `zh` de `useClient()`. Migrar progresivamente. Cuando esté centralizado, valorar i18next.

**Esfuerzo**: 1 semana. Plan en [[Sprint-7-DX-Cleanup]].

## Auth en localStorage (162 referencias)

162 referencias a `localStorage`/`sessionStorage` en 49 archivos. Tokens, emails, slugs de cliente, flags de admin... todo plaintext, accesible por cualquier script con XSS.

**Fix**: migrar a httpOnly cookies firmadas por backend + CSRF tokens. Esto es parte estructural de [[Sprint-4-Webhooks-Secrets]] o un sprint propio.

## 2642 colores hex hardcoded

`grep '#[0-9A-Fa-f]\{6\}' src/pages/` devuelve 2642 ocurrencias. Top: `#fff` (147), `#EF4444` (222 contando lower y upper), `#3B82F6` (139). Mezcla con `var(--text)` en algunos sitios.

**Fix**: centralizar paleta en `styles/palette.js`. Migrar componentes a CSS variables `var(--color-primary)`. Cuando 80% sean variables, cambiar tema = 1 archivo.

**Esfuerzo**: 1-2 semanas en paralelo a otro trabajo.

## 0 tests, 0 linter, 0 TypeScript

- Sin `vitest` / `jest` / `mocha`
- Sin ESLint
- Sin TypeScript ni JSDoc tipos

**Fix**:
1. Añadir `vitest` + `@testing-library/react`. Tests de `utils/`, `hooks/`, `permissions.js`. Aim: 80% coverage en utils críticos.
2. Añadir `Playwright` para 5 flujos críticos: login, crear contacto, crear venta, mandar email, login portal.
3. ESLint + Prettier configurados.
4. TypeScript gradual: empezar por archivos nuevos. JSDoc en archivos viejos.

**Esfuerzo**: 3 semanas. Plan en [[Sprint-3-Testing]].

## Sin observability (backend)

- 167 `console.log/error` sin estructura
- Sin request_id / correlation ID
- Sin nivel (info/warn/error)
- Sin Sentry / Logtail / Grafana
- Logs van a stdout efímero (si reinicia el contenedor, perdidos)

**Fix**: 
- Adoptar `pino` con structured JSON
- Middleware Fastify para inyectar `correlationId` por request
- Sentry frontend + backend
- Logtail/Grafana para retención

**Esfuerzo**: 2 semanas. Plan en [[Sprint-2-Observability]].

## Sin error handler global (backend)

Cada endpoint maneja errores inline. Algunos filtran detalles (`detail: err.message` revela queries de Postgres, API keys de Resend). No hay estructura de respuesta uniforme.

**Fix**: `app.setErrorHandler((err, req, reply) => ...)` centralizado. Enmascarar detalles. Stack trace solo en logs privados.

**Esfuerzo**: 2 días. Parte de [[Sprint-2-Observability]].

## Modales inline (~6500 LOC)

Páginas grandes (CrmPage, EmailMarketing, CalendlyPanel, InstallmentPayments) tienen modales escritos inline en lugar de componentes propios. Total estimado: 6500 LOC que deberían estar en `components/`.

**Fix**: extraer + tipos compartidos. Es trabajo de pico, no estructural.

## Componentes shared infrautilizados

48 componentes en `src/components/` para 210 páginas = 23% de reutilización. Patrones obvios (Button, Card, Modal, Form, Table, EmptyState) viven hardcoded.

**Fix**: crear UI lib interna + Storybook. Plan en [[Sprint-8-Component-Library]].

## XLSX import * (400KB)

`src/components/ImportModal.jsx` hace `import * as XLSX from 'xlsx'` (400KB). Cualquier página que renderice ImportModal arrastra los 400KB.

**Fix**: dynamic import.
```javascript
const XLSX = await import('xlsx')
```

**Esfuerzo**: 30 min.

## ClientType hardcoded en 20+ sitios

Cada `clientType === 'growth'`, `=== 'consultoria'`, etc. está hardcoded en lógica del frontend. Si añades un tipo nuevo, tocas 20 archivos.

**Fix**: enum en `src/constants/clientTypes.js`. Helper `isGrowth(client)`, `isConsultoria(client)`. Config-driven.

## Sin schema validation (zod/yup)

Forms y API requests sin validación de shape. 115 columnas JSONB sin validar. Drift entre código que escribe y código que lee.

**Fix**: añadir `zod`. Schemas en `src/schemas/`. Validar en boundary (formularios + API responses + JSON DB).

**Esfuerzo**: 1-2 semanas en paralelo.

## Conexiones

- Estructural: [[Sprint-7-DX-Cleanup]] · [[Sprint-3-Testing]] · [[Sprint-8-Component-Library]]
- Crítico: [[Bugs-Criticos]]
- Por qué duele: [[Flaquezas-Estructurales]]
