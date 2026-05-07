---
tags: [moc, clientes]
created: 2026-05-07
---

# MOC — Clientes

Map of Content de tenants en producción. **Dos plataformas, dos cohortes**:

- **Dashboard-Ops** (16 tenants growth/consultoría/etc en Supabase + Vercel)
- **[[Cargonex]]** (1 tenant logística en VPS dedicado + Postgres on-prem)

Cada uno tiene su brain con identidad, accesos, infraestructura, KPIs y gotchas.

## Vivos en producción — Dashboard-Ops (`central.blackwolfsec.io`)

| Slug | Cliente | Tipo | Estado | Brain |
|---|---|---|---|---|
| `enformaconhugo` | Hugo (En Forma con Hugo) | growth (infoproducto) | live · Stripe live · Hotmart pendiente | [[Hugo-EnFormaConHugo]] |
| `asesorias-suiza` | Portillo + Lukas | consultoria multi-owner | live · Stripe Portillo (CHF) · GCal freebusy | [[AsesoriaSuiza-Portillo-Lukas]] |
| `fba-academy` | FBA Academy | growth (store_client) | live · onboarding store clients | [[FBA-Academy]] |
| `yc-logistics` | IC Logistics | logistica | live · landing bootcamp China integrada · light theme forzado | [[IC-Logistics]] |
| `creator-founder` | Creator Founder | growth | live · evento Madrid 2-may-2026 · WA banneo 26-abr | [[Creator-Founder]] |
| `detras-de-camara` | Abel Casal | growth | live · lanzamiento 28-abr-2026 · Resend pendiente config | [[Detras-de-Camara]] |
| `cristian` | Cristian Ibarsies | software_developing | live · solo Tickets + Tasks | [[Cristian-Software]] |
| `black-wolf` | BlackWolf interno | admin | live · super-admin console | [[BlackWolf-Internal]] |

(El resto de slugs están en seed pero menos activos. Ver `clients` table en Supabase.)

## Vivos en producción — Cargonex (`app.cargonex.co`)

| Slug | Cliente | Tipo | Estado | Brain |
|---|---|---|---|---|
| `EilersLogistik` | Eilers Logistik GmbH | logistica (DE) | live · 231 Fahrer / 162 Fahrzeuge · 8343 records · `legacy:true` (en `db.json`) · DSGVO docs pendientes | [[Eilers-Logistik]] |

> ⚠️ **No confundir** [[IC-Logistics]] (Dashboard-Ops, slug `yc-logistics`, ES/EN/ZH) con [[Eilers-Logistik]] (Cargonex, slug `EilersLogistik`, DE). Son plataformas distintas con bases de datos distintas, aunque ambos sean vertical "logística".

## Patrones reusables

- [[Playbook-Onboarding-Cliente-Nuevo]] — Fase 0-5 onboarding **Dashboard-Ops**
- [[Playbook-Crear-Tenant-Cargonex]] — onboarding **Cargonex** (PI-key + slug routing)
- [[Tipos-de-Cliente]] — `growth`, `consultoria`, `software_developing`, `manufactura`, `logistica`, `admin` (Dashboard-Ops)
- [[Multi-Owner]] — patrón asesoria-suiza (Portillo/Lukas) y cómo escalar

## Quién paga, cuándo

Ver [[Modelo-de-Negocio]] · [[Stripe]] · [[Hotmart]] (pendiente).

- Tenants Dashboard-Ops: pricing a medida, billing manual hoy.
- Eilers (Cargonex): contrato anual offline, sin Stripe integrado.

## Conexiones

- Cómo se les sirve: [[_moc/MOC-Plataforma]] (ambas plataformas)
- Riesgos cross-tenant Dashboard-Ops: [[Bugs-Criticos#C6]] (RLS permisivas)
- Riesgos cross-tenant Cargonex: [[Cargonex#Estado]] (`PLATFORM_ADMIN_SECRET` placeholder)
