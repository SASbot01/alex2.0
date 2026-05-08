---
tags: [decision-cto, cargonex, arquitectura, adr]
status: accepted
created: 2026-05-07
decision-date: 2026-04-21
---

# ADR — Cargonex con Stack Separado de Dashboard-Ops

## Decisión

[[Cargonex]] se construye y se opera como un **producto independiente** de [[Dashboard-Ops]], no como otro `clientType` ni como un módulo. Repos separados, deploys separados, base de datos separada, dominio separado, branding separado.

## Estado

✅ **Accepted** — implementado y en producción desde 2026-04-21. Eilers Logistik live en `app.cargonex.co/EilersLogistik`.

## Contexto

Cuando entró Eilers como cliente:

- Eilers necesita gestión de flota (drivers, vehicles, dispatcher, ÜB photos, tour planning, OBD2). Nada de esto existe en Dashboard-Ops.
- Eilers es **alemán** → DSGVO no negociable. Requiere DPA, retention policy, sub-processor disclosure, procedimiento de export/delete.
- Eilers es **mobile-first** (drivers usan el portal en su móvil) → necesita PWA con SW + push real, no React-app-en-Vercel.
- Eilers es **on-prem-en-VPS** por preferencia del cliente — no Supabase shared, datos en territorio + control directo.

Las opciones evaluadas fueron tres:

### Opción A — Meterlo como `clientType: logistica` en Dashboard-Ops

Ya hay un `clientType: logistica` ([[IC-Logistics]]) en Dashboard-Ops. Tentador re-usar.

**Contras**:
- Dashboard-Ops asume Supabase como DB. Eilers requiere on-prem → split del backend de DO en dos modos = ñapa.
- Dashboard-Ops es React + Vite + Vercel. Cargonex necesita PWA con SW agresivo + offline + push. Se podría hacer en Vercel pero la latency de un round-trip Supabase desde Alemania para cada sync de 8000 records es horrible.
- Branding: Dashboard-Ops es "BlackWolf" para los clientes growth/consultoría. Eilers no quiere verse "powered by BlackWolf" en cada pantalla.
- DSGVO + Supabase es resoluble pero meter compromiso a un cliente cuando otros 15 tenants comparten infra es scope creep enorme.

### Opción B — Fork de Dashboard-Ops y customizar

Clonar el repo, cortar todo lo que no aplica (CRM, formación, marketplace, etc.), añadir lo de logística.

**Contras**:
- 50% de Dashboard-Ops es código React específico de growth/infoproducto que no aplica.
- Heredas el sistema de auth custom + multi-tenant Supabase + RLS allow-all = problemas de [[Bugs-Criticos]] que en Cargonex puedes evitar de raíz.
- Mantener dos forks divergentes ahoga.

### Opción C — Stack separado de cero (la elegida)

Repo nuevo (`eilerrs-portal`), stack mínimo y específico:
- Express ESM (sin framework de UI server-side complejo)
- Vanilla PWA (sin React — el SPA del fleet management funciona perfectamente sin)
- Postgres dedicado en el mismo VPS
- Cloudflare Tunnel (sin puertos públicos)
- Multi-tenant via slug routing + JSON files per-tenant + `cargonex-platform.json` registry

**Pros**:
- Stack ajustado al problema. Sin overhead.
- Aislado: si Cargonex tiene un bug, no afecta a los 16 tenants de Dashboard-Ops.
- DSGVO localizado: cumplimiento por producto, no global.
- Branding propio (cargonex.co), comercializable a otros clientes logística sin "BlackWolf" pegado.
- Velocidad de iteración alta — codebase pequeña, sin matriz de tipos de cliente.

**Contras (asumidos)**:
- Duplicación: cosas como auth-OTP, mailer, audit log se rehacen. Aceptado — son ~200 LOC cada una y la versión Cargonex es más simple.
- Operacionalmente dos stacks que mantener (parches, monitoring). Aceptado — Cargonex es deliberadamente conservador (Express + JSON + Postgres) y bajo en bugs.
- Si llega un cliente híbrido que necesita ambos productos → se le venden los dos por separado, no se intenta unificar (premature).

## Implicaciones

- **Repos**: `eilerrs-portal` (Cargonex) ↔ `Dashboard-Ops-` + `ejambre` (BlackWolf platform). Sin código compartido.
- **Hosting**: ambos en el mismo VPS pero stacks Docker separados — ver [[Host-Docker-Layout]] para evitar colisiones (puertos 3000 vs 3010, redes, volúmenes).
- **Identidades**: clientes de Cargonex no son rows en `clients` de Supabase. Son rows en `cargonex-platform.json#tenants`. No mezclar al hacer queries cross-producto.
- **Brain en Alex 2.0**: Cargonex tiene su propia [[01-Plataforma/Cargonex|sección de plataforma]] paralela; sus tenants viven en [[02-Clientes]] pero marcados con `platform: cargonex` en frontmatter para distinguirlos.
- **Modelo de negocio**: vector de ingreso #5, no extensión del #1 ([[Modelo-de-Negocio]]).

## Cuándo reconsiderar

Reconsiderar consolidación si:

1. Llegan ≥ 3 clientes logística que pidan "lo mismo que Eilers" → en ese momento tiene sentido producto Cargonex con marketing y pricing claros (ya estamos camino, eso lo confirma).
2. Aparece un cliente que necesita **ambas cosas** (un cliente growth + flota propia) — improbable, pero entonces evaluar puente.
3. Cuando Dashboard-Ops madure su capa de aislamiento (RLS real, billing, etc.) y deje de ser un riesgo metérsele cosas nuevas → más adelante, no hoy.

## Conexiones

- Producto: [[Cargonex]] · [[Arquitectura]]
- Cliente fundador: [[Eilers-Logistik]]
- Hardware compartido: [[Host-Docker-Layout]]
- Producto hermano (no se mezcla): [[Dashboard-Ops]] (descrito vía [[Arquitectura-General]])
- Negocio: [[Modelo-de-Negocio]]
