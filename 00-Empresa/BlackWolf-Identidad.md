---
tags: [empresa, identidad]
created: 2026-05-07
---

# BlackWolf — Identidad

## Qué es

BlackWolf es una empresa de tecnología y operaciones B2B liderada por Alejandro Silvestre Fuentes (CTO) que vende un stack vertical de productos a empresas de información, formación, growth y servicios.

**Dominio principal**: `blackwolfsec.io`

## Marcas / sub-productos

- **Dashboard-Ops** — el SaaS multi-tenant que sirve a los clientes (CRM, ventas, formación, agendas, marketing, IA agents). Sirve en `central.blackwolfsec.io`.
- **Wolf Trader** — ver [[Stack-de-Productos]]
- **SOC** — operaciones de seguridad. Webhook receiver en `/api/webhooks/soc`. Ver `soc.blackwolfsec.io`
- **Donna** — asistente ejecutiva única, viva en Telegram, coordina los demás sistemas. Ver [[Donna-Telegram]]
- **Enjambre** — backend que ejecuta para Donna y los productos. Ver [[Enjambre-API]]

## Identidad técnica

- **Repos GitHub** principales:
  - `aatshadow/Dashboard-Ops-` — frontend SaaS
  - `SASbot01/ejambre` — monorepo backend (enjambre-api, enjambre-dashboard, docker)
  - `SASbot01/alex2.0` — este vault (segundo cerebro)
- **Hosting**:
  - Frontend: [[Vercel]] auto-deploy desde main
  - Backend: server propio detrás de [[Cloudflare]] Tunnel
- **Identidades de commit**:
  - `SASbot01 / silvestrefuentesalejandro@gmail.com`
  - `aatshadow` (alternativa)
  - Vercel **rechaza deploys** de otros autores — ver [[Naming-Conventions]]

## Identidad operativa

- **CTO + dueño técnico**: Alejandro. Habla español de España. Interlocutor senior. Ver [[Alejandro-CTO]]
- **Email**: `alejandro.cto@blackwolfsec.io`
- **Idiomas operativos**: español primary, inglés y chino para tenants específicos (yc-logistics, asesoria-suiza)

## Qué NO es

- No es una agencia de marketing.
- No es una consultora generalista.
- No es un infoproducto propio de Alejandro (aunque sirve a clientes que sí lo son).
- No es product-led-growth puro: cada cliente es onboardeado a mano según [[Playbook-Onboarding-Cliente-Nuevo]].

## Conexiones

- [[Modelo-de-Negocio]]
- [[Stack-de-Productos]]
- [[Equipo]]
