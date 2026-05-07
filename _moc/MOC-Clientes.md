---
tags: [moc, clientes]
created: 2026-05-07
---

# MOC — Clientes

Map of Content de los 16 tenants en producción. Cada uno tiene su brain con identidad, accesos, infraestructura, KPIs y gotchas.

## Vivos en producción

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

## Patrones reusables

- [[Playbook-Onboarding-Cliente-Nuevo]] — Fase 0-5 del onboarding
- [[Tipos-de-Cliente]] — `growth`, `consultoria`, `software_developing`, `manufactura`, `logistica`, `admin`
- [[Multi-Owner]] — patrón asesoria-suiza (Portillo/Lukas) y cómo escalar

## Quién paga, cuándo

Ver [[Modelo-de-Negocio]] · [[Stripe]] · [[Hotmart]] (pendiente).

## Conexiones

- Cómo se les sirve: [[_moc/MOC-Plataforma]]
- Riesgos cross-tenant: [[Bugs-Criticos#C6]] (RLS permisivas)
