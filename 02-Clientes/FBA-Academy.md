---
tags: [cliente, growth, store_client]
slug: fba-academy
client_type: growth
status: live
created: 2026-05-07
---

# FBA Academy

Cliente growth con tipo de usuario `store_client` (alumnos que tienen tiendas Amazon FBA). Onboarding distinto al de team — el alumno gestiona su propia tienda, no el negocio del cliente.

## Identidad

- **Slug**: `fba-academy`
- **clientType**: `growth`
- **Vertical**: formación FBA Amazon + gestión de tiendas
- **Idioma**: español
- **Logo**: `/assets/logos/fba-academy.jpeg` (en TENANT_LOGOS)

## Modelo

Los alumnos de FBA Academy abren tiendas Amazon FBA. La plataforma les da:
- [[Modulo-Stores]] — gestión de su tienda (estado, pasos, progreso)
- [[Modulo-Tickets]] — soporte con su gestor
- [[Modulo-Formacion-Infoproducto]] — cursos de la academia

## Particularidades técnicas

- **`userType = 'store_client'`** — onboarding minimal con 4 pasos (welcome, store, training, tickets)
- **Onboarding diferente** al de team users (solo 4 cards en lugar del flow completo)
- **Tabs específicas**:
  - `/fba-academy/tiendas` — dashboard de su tienda
  - `/fba-academy/tiendas/tickets` — sus tickets
  - `/fba-academy/formacion` (ahora `/infoproducto`) — sus cursos
- **No tiene `clientType === 'growth'` para Marketplace** — ver excepción `clientSlug !== 'fba-academy'` en `forceInfoProductosRedirect` de ClientApp.jsx:710

## Bookings públicos

- URL branded: `central.blackwolfsec.io/fba-academy/agendas` — public booking con logo y nombre.
- Logo per-tenant via TENANT_LOGOS (centralizado tras [[Sesion-2026-05-07]]).

## Pendientes / observaciones

- Verificar que post-deploy del [[Portal-Infoproducto]] el portal `/portal/fba-academy` funciona (ya está logo cableado).
- KPIs específicos de stores (% tienda activa, ticket lifecycle) — ver [[Modulo-Stores]].

## Conexiones

- Tipos de cliente: [[Tipos-de-Cliente]] · [[ClientTypes]]
- Módulos clave: [[Modulo-Stores]] · [[Modulo-Tickets]] · [[Modulo-Formacion-Infoproducto]]
