---
tags: [integraciones, whatsapp, rfp, decision]
created: 2026-05-08
updated: 2026-05-08
status: research
decision_pending: true
---

# RFP — Migración whatsapp-web.js → WhatsApp Business API oficial

Sprint Arreglos `[whatsapp] RFP a 360dialog/Twilio/Meta WhatsApp Business API` (4h, high). Comparativa de proveedores oficiales para reemplazar el connector actual basado en `whatsapp-web.js` (no-oficial, riesgo de banned).

## Contexto

### Estado actual
- `enjambre-api/src/connectors/whatsapp.js` usa `whatsapp-web.js` (Chromium headless + WhatsApp Web).
- 5 sesiones simultáneas (limit `WA_MAX_SESSIONS`), cada una con `.wwebjs_auth/<sessionKey>` profile.
- **Riesgo crítico**: WhatsApp puede bannear cualquier número usado vía web.js sin previo aviso. Ya pasó con Creator Founder 2026-04-26 (memoria `project_creator_founder_event.md`). Recovery = perder histórico de chats + reescanear QR.
- Setter automático + Anthropic Claude responde inbound → genera tráfico considerable que aumenta riesgo banned.

### Por qué migrar ahora
1. Cliente principal (BlackWolf), Asesoría Suiza (Lukas + Portillo, 2 cuentas), En Forma con Hugo, Detrás de Cámara — 4+ tenants confiando en WA. Un banned masivo = incidente nivel 1.
2. Compliance: WA Business Solution Providers son la única vía oficial. Para clientes empresa o regulated industries (Lukas seguros) es preferible.
3. Features oficiales: interactive buttons, message templates con verificación, payments inline, click-to-WA ads tracking.

## Proveedores evaluados

### 1. Meta Direct (Cloud API)

**Modelo**: API HTTP propia de Meta. Sin intermediario.

| Feature | Detalle |
|---|---|
| Pricing | Pay-per-conversation: ~$0.005-0.04 USD según país. Free: 1,000 servicio/mes. Marketing/utility separados. |
| Setup time | ~3-5 días: app Meta + verificación negocio + número WA + permisos `whatsapp_business_messaging`. |
| Multi-tenant | 1 número WA = 1 phone_number_id. Cada cliente necesita su número propio. |
| Templates | Aprobación Meta 24-48h. Variables ${1}, ${2}. |
| Inbound media | URL temporal (~5 días). Hay que descargar y persistir. |
| Webhooks | Sí, signature verification HMAC-SHA256. |
| Ratelimit | Tier 1: 1k convs/24h. Sube con calidad / volumen. |
| Onboarding tenants | Cada cliente verifica su negocio Meta + número (proceso owner). |

**Pros**:
- Más barato a escala.
- Sin intermediario → menos latencia + más control.
- Same vendor lock-in que ya tenemos (CAPI, Ads).

**Cons**:
- Onboarding por cliente más complejo (Meta Business verification).
- Soporte Meta = colas Help Center, no chat con BSP.
- Cada cliente necesita su número WA propio (no se puede multiplexar).

### 2. 360dialog (BSP)

**Modelo**: Reseller oficial Meta. API tipo Cloud API + valor añadido.

| Feature | Detalle |
|---|---|
| Pricing | Setup: ~5 EUR/número. Mensualidad: 5-15 EUR/número. Plus pricing Meta on top (~$0.02/conv). |
| Setup time | ~24h (más rápido por la verificación pre-negociada con Meta). |
| Multi-tenant | Pricing escala lineal por número. Multi-tenant nativo. |
| Templates | Submit via API o dashboard 360dialog. |
| Inbound | Webhook + media URL. |
| Onboarding tenants | 360dialog handles Meta verification en su cuenta master, cliente solo añade número. |

**Pros**:
- Tier mid-market: por debajo de Twilio y por encima de DIY Meta.
- Onboarding cliente más rápido (24h vs 5d).
- Soporte humano.

**Cons**:
- Margen 360dialog encima de Meta = ~30-50% más caro a escala.
- Lock-in si cambiamos de BSP requiere re-verificar con Meta.

### 3. Twilio

**Modelo**: BSP enterprise. API uniforme con su SDK existente.

| Feature | Detalle |
|---|---|
| Pricing | $0.005/msg out + $0.005-0.04/conv según país. Setup gratis. |
| Setup time | ~7-14 días (proceso WhatsApp Business strict). |
| Multi-tenant | Subaccounts Twilio nativas. Cada cliente → subaccount con billing separado. |
| Templates | Submit via Twilio Console o API. |
| Inbound | Webhook. Media incluida en URL persistente. |
| SDK | Maduro: `twilio` npm package. |

**Pros**:
- SDK enterprise-grade, docs excelentes.
- Subaccounts → multi-tenant trivial + facturación per-cliente.
- Soporte enterprise (24/7 si pagas plan high).

**Cons**:
- Más caro (margen Twilio + Meta + onboarding lento).
- Lock-in Twilio fuerte.
- Pricing complejo (SMS + WhatsApp + Voice mixed accounts).

## Comparativa rápida

| Criterio | Meta Direct | 360dialog | Twilio |
|---|---|---|---|
| Coste mensual base (5 números) | ~$0 + uso | ~25-75 EUR + uso | $0 + uso (más alto) |
| Coste por conversación | ★★★ (más barato) | ★★ | ★ (más caro) |
| Setup time | 3-5d/cliente | 1-2d/cliente | 7-14d/cliente |
| Multi-tenant nativo | ★ (manual) | ★★ | ★★★ (subaccounts) |
| SDK / DX | ★★ (HTTP raw) | ★★ (HTTP) | ★★★ (SDK maduro) |
| Soporte | ★ (Meta queue) | ★★ (humano) | ★★★ (enterprise) |
| Migration path desde web.js | Mismo (rewrite connector) | Mismo | Mismo |

## Recomendación

### Tier por escala

- **<5 números**: **Meta Direct (Cloud API)**. Sin overhead de BSP, control total, onboarding manejable.
- **5-50 números**: **360dialog**. Onboarding rápido + soporte humano + costo razonable.
- **50+ números o enterprise clients**: **Twilio**. Subaccounts + facturación segregada.

**BlackWolf hoy**: ~5 números (BlackWolf + 2 Asesoría Suiza + Hugo + Detrás de Cámara). Frontera entre Meta Direct y 360dialog.

### Decisión sugerida (a confirmar)

**Iniciar con 360dialog** porque:
1. Onboarding cliente en 24h vs 5d con Meta (Lukas + Portillo verificarán seguros, complican Meta verification por industria regulada).
2. Soporte humano útil para debug (Meta queue es opaca).
3. Coste mid (~30-50 EUR/mes en total = no determinante).
4. Si crece a 50+ números → migrar a Twilio (no se reescribe nada, solo cambia el HTTP target).

### Alternativa: dual-track

Mantener `whatsapp-web.js` para **inbound** (chats existentes con clientes finales que no aceptan migrar al número oficial) y migrar **outbound** broadcast/setter a 360dialog. Reduce riesgo banned drásticamente porque outbound es lo que más triggera flags.

## Plan de migración (propuesta)

Esfuerzo estimado: **16h** (matchea la tarea Sprint Arreglos `[whatsapp] Adapter abstrayendo whatsapp-web.js vs WA Business API`).

1. Crear `enjambre-api/src/connectors/whatsapp/` con dos backends:
   - `web-js.js` (actual)
   - `business-api-360dialog.js` (nuevo)
2. Interfaz común: `sendMessage(to, body)`, `sendTemplate(to, template, vars)`, `onMessage(handler)`, `getStatus()`.
3. Toggle por env var: `WA_BACKEND=web-js | 360dialog | meta`.
4. Testing en sandbox 360dialog con 1 número de prueba.
5. Cutover por cliente:
   - Asesoría Suiza Portillo (low risk, baja conversación frecuencia).
   - Asesoría Suiza Lukas (alta frecuencia, requiere migración de templates aprobados).
   - Hugo (alta frecuencia setter).
   - Detrás de Cámara (post-evento).
   - BlackWolf (último, son agentes internos).

## Acciones inmediatas (esta semana)

- [ ] **Decisión Alejandro**: ¿Meta Direct, 360dialog o Twilio?
- [ ] Si 360dialog: crear cuenta sandbox + obtener API key (1h).
- [ ] Programar migración cliente piloto (Asesoría Suiza Portillo).

## Conexiones

- Tarea hermana: `[whatsapp] Adapter abstrayendo whatsapp-web.js vs WA Business API` (16h, high) — implementación concreta tras decisión.
- Memoria: incidente 2026-04-26 baneo Creator Founder.
- Sprint Arreglos `[whatsapp]` cerrados hoy: detección banned, encriptación at-rest análisis.

## Histórico de proveedores

Estado de mercado WhatsApp BSP a 2026:
- Meta entró a Cloud API directo en 2022, eliminando intermediarios obligatorios.
- 360dialog adquirido por Sinch en 2023 (sigue operando como brand).
- Twilio mantiene posición enterprise, sigue siendo referencia para multi-account.
- Otros BSPs (Vonage, MessageBird, AiSensy) consultados pero menos relevantes para nuestra escala.
