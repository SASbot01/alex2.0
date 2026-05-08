---
tags: [cliente, software_developing]
slug: cristian
client_type: software_developing
status: live
created: 2026-05-07
---

# Cristian — Software Project

Primer cliente del nuevo `clientType = software_developing`. Director: Cristian Ibarsies. Solo Tickets + Task Management — el tenant no necesita CRM, ventas, formación, etc.

## Identidad

- **Slug**: `cristian`
- **clientType**: `software_developing`
- **Director**: Cristian Ibarsies

## Por qué importa

Es el caso que **valida que la plataforma puede servir clientes técnicos sin todo el bagaje SaaS de growth/consultoria**. Marca el patrón para futuros clientes tipo "agencia/freelance que solo necesita gestión de proyectos".

## Módulos visibles

- ✅ [[Modulo-Tickets]]
- ✅ [[Modulo-Tareas]] (task-management)
- ❌ CRM, Sales, Formación, Marketplace, Stores, etc. — ocultos

## Patrón replicable

Cuando llegue otro `software_developing`, el código ya está preparado: `clientType === 'software_developing'` filtra el sidebar y los routes en ClientApp.jsx.

## Conexiones

- Tipos: [[Tipos-de-Cliente]] · [[ClientTypes]]
- Módulos: [[Modulo-Tickets]] · [[Modulo-Tareas]]
