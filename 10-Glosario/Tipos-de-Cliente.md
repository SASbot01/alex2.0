---
tags: [glosario, referencia]
created: 2026-05-07
---

# Tipos de Cliente (clientType)

`clients.client_type` define qué módulos ve cada tenant en Dashboard-Ops. Es el switch principal para multi-vertical.

| clientType | Caso de uso | Módulos clave | Ejemplo |
|---|---|---|---|
| `growth` | Infoproductos, formación con Marketplace | CRM, Sales, Formación, Comunidad, Marketplace, Mentorías, Marketing | [[Hugo-EnFormaConHugo]], [[FBA-Academy]], [[Creator-Founder]], [[Detras-de-Camara]] |
| `consultoria` | Consultoría / servicios profesionales | CRM, Bookings, Sales (sin Marketplace, sin Formación heavy) | [[AsesoriaSuiza-Portillo-Lukas]] |
| `software_developing` | Agencias / freelance / dev | Solo Tickets + Tasks (sin CRM, Sales, Formación) | [[Cristian-Software]] |
| `manufactura` | Manufacturing ERP | Manufacturing, CRM, Sales | (futuro) |
| `logistica` | Logística / bootcamps | Landings integradas, CRM, Forms, soporte ticket | [[IC-Logistics]] |
| `admin` | BlackWolf interno super-admin | Todo + paneles internos | `black-wolf` |

## Feature flags + clientType

El sidebar y las rutas se gatean con DOS layers:

1. **`clientType`** — define el set base de módulos (hardcoded en `ClientApp.jsx`)
2. **`clients.config.features`** (jsonb) — overrides finos por tenant

Ej: un cliente `consultoria` puede activar `formacion: true` en su `features` para tener acceso al módulo aunque el default del clientType lo oculte.

## Cómo se enforce

```javascript
// ClientApp.jsx (1573 LOC monolítico)
const isGrowth = clientConfig?.clientType === 'growth'
const isConsultoria = clientConfig?.clientType === 'consultoria'

const feat = getEffectiveFeatures(clientConfig, clientSlug)
const showCrm = feat.crm
const showFormacion = feat.formacion
// etc.
```

## Gotchas

- **`clientType` hardcoded en 20+ sitios** del frontend. Si añades un nuevo tipo, hay que tocar todos. Ver [[Deuda-Tecnica]] punto "ClientType hardcoded en 20+ sitios".
- **No hay enum** ni constants. Strings literales en todos lados.
- **Caso especial FBA Academy**: es `growth` pero con `userType='store_client'` y excepciones (no Marketplace). Ver `forceInfoProductosRedirect = rawIsStores && isGrowthRenameClient` en `ClientApp.jsx:710`.

## Conexiones

- Cómo se onboarda un tipo: [[Playbook-Onboarding-Cliente-Nuevo]]
- Casos vivos: [[_moc/MOC-Clientes]]
- Refactor pendiente: [[Sprint-7-DX-Cleanup]]
