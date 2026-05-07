---
tags: [empresa, negocio]
created: 2026-05-07
---

# Modelo de Negocio

## Vectores de ingreso

### 1. SaaS multi-tenant Dashboard-Ops

Cada cliente paga por usar el [[Dashboard-Ops]] adaptado a su `clientType` y sus features. Hoy hay 16 tenants en producción con grados muy distintos de uso (algunos con 1000+ contactos, otros con <10).

- **Pricing**: no hay catálogo público. Cada onboarding es a medida.
- **Features per-cliente**: controladas por `clients.config.features` (jsonb) + `clientType` (`growth`, `consultoria`, `software_developing`, `manufactura`, `logistica`, `admin`).
- **Friction de cobro**: alta hoy. No hay billing automático integrado al producto. Stripe está conectado solo para 2 cuentas (Hugo y Portillo) y para que sus productos cobren a SUS alumnos, no para que paguen el SaaS.
- **Oportunidad**: meter cobro recurrente del SaaS por Stripe con webhook al CRM interno de BlackWolf (tabla `clients.billing_*`).

### 2. Service revenue (implementación + customización)

Clientes nuevos pagan por el onboarding + configuración inicial:
- Crear el tenant en Supabase
- Configurar pipelines del CRM
- Conectar Resend, Stripe, WhatsApp, Google
- Subir landings + grupos WA + replays a `info_producto_assets` (Supabase)
- Configurar `email_config`, `whatsapp_config`, etc.

Ver [[Playbook-Onboarding-Cliente-Nuevo]] para el flujo Fase 0-5.

### 3. Infoproductos de los clientes

BlackWolf NO vende infoproductos propios, pero la plataforma está optimizada para clientes que SÍ lo hacen (Hugo, Asesoría Suiza, Detrás de Cámara, Creator Founder). El [[Modulo-Formacion-Infoproducto]] + [[Portal-Infoproducto]] es una capa core.

### 4. Growth services (Wolf Trader, SOC, otros)

- **Wolf Trader** — trading desk
- **SOC** — security ops, webhook en `/api/webhooks/soc`

### 5. SaaS multi-tenant [[Cargonex]] (segunda línea de producto)

Plataforma white-label para gestión de flotas de logística, productizada en `app.cargonex.co` con marketing propio en `web.cargonex.co`. Stack y deploy completamente separados de Dashboard-Ops — ver [[ADR-Cargonex-Stack-Separado]].

- **Pricing**: tier `starter` · `professional` · `enterprise` (no enforced en producto, solo display). Cliente fundador [[Eilers-Logistik]] paga por contrato anual offline.
- **Vertical**: logística europea, mobile-first, DSGVO-conforme.
- **Onboarding**: PI-key (`CX-XXXXXXXX-XXXX`) emitido en `/admin` → cliente activa en `/portal` → aterriza en `/<Slug>` con DB aislada. Ver [[Playbook-Crear-Tenant-Cargonex]].
- **Friction de cobro**: alta (igual que Dashboard-Ops). No hay billing automático integrado. Si se quiere escalar a más clientes logística, meter Stripe Connect.
- **Oportunidad**: si el patrón Cargonex funciona con Eilers, productizar landings + pricing + Stripe automatizado para vender a más clientes logística europeos sin overhead operativo.

## Cómo gana cada cliente típico

| Cliente tipo | Cómo gana | Cómo BlackWolf le ayuda |
|---|---|---|
| Infoproducto (Hugo, Abel) | Vender curso/programa por Stripe/Hotmart | CRM + Stripe sync + landings + emails Resend + comunidad |
| Consultoría (Portillo) | Sesiones 1:1 cobradas por Stripe | Bookings GCal freebusy + CRM + multi-owner scope |
| Stores/FBA (FBA Academy) | Gestión Amazon de tiendas de alumnos | Módulo Stores + tickets de soporte + onboarding sequence |
| Logística (IC Logistics) | Bootcamps + servicios | Landing integrada al CRM via `/api/forms/icl-submit` |
| Eventos (Creator Founder) | Vender entradas a eventos físicos | Pipeline Evento + check-in HMAC + campaña Resend con QR |
| Software (Cristian) | Desarrollo a medida | Solo Tickets + Tasks (clientType `software_developing`) |

## Costes operativos

- **Anthropic**: capa cognitiva (Donna + Army + workers) consume tokens. Tracking en [[AI-Cost-Tracking]] vía `brain_decisions.decision_type='ai_usage'`.
- **Supabase**: plan Pro+ esperable (PITR + storage)
- **Vercel**: frontend
- **Cloudflare**: tunnel gratis
- **Resend, Stripe**: pagados por los TENANTS (con sus propias keys), no por BlackWolf
- **Server enjambre-api**: este host (cwd `/home/blackwolfsec/`)

## Riesgos del modelo

1. **Dependencia de whatsapp-web.js** — un banneo masivo te tumba la oferta de "CRM 2-way con WhatsApp" en TODOS los clientes. Ver [[Decision-WhatsApp]].
2. **Multi-tenancy permisivo** — un leak cross-tenant es riesgo legal grave. Ver [[Bugs-Criticos#C6]].
3. **Pricing no documentado** — depende de Alejandro como persona. Bus factor 1.
4. **Bus factor del onboarding** — playbook existe pero el conocimiento real está en cabeza.

## Conexiones

- Quiénes pagan hoy: [[_moc/MOC-Clientes]]
- Cómo cobran ellos: [[Stripe]] · [[Hotmart]]
- Qué reciben: [[Stack-de-Productos]]
