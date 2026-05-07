---
tags: [auditoria, flaquezas]
created: 2026-05-07
---

# Flaquezas Estructurales

Patrones que duelen, no items individuales. Estos son los 10 ejes de mejora estructural — los individuales viven en [[Bugs-Criticos]] y [[Deuda-Tecnica]].

## 1. Auth en localStorage plaintext

162 referencias a localStorage en 49 archivos. Tokens, emails, slugs, flags de admin. Cualquier XSS = full takeover.

**Por qué duele**: la postura de seguridad de la plataforma queda definida por esto. Vender a empresa con CISO requiere migrar a httpOnly cookies firmadas + CSRF.

## 2. RLS permisiva (multi-tenant solo en frontend)

127 tablas con RLS habilitada pero todas las policies son `USING (true)`. El aislamiento es voluntario, no enforced. Un atacante con anon key + endpoint Supabase puede leer/escribir entre tenants.

**Por qué duele**: leak cross-tenant es riesgo legal en cliente con datos personales (Hugo, Portillo, etc.).

## 3. Sin observability

Cuando algo se rompe en producción, no tienes forma de saber por qué sin SSH al server. 167 console.log sin estructura, sin request_id, sin level. Logs van a stdout efímero.

**Por qué duele**: cada incidente que reporta un cliente cuesta horas de investigación. Esto NO escala.

## 4. Sin testing automatizado

0 unit tests, 0 e2e, 0 linter, 0 TypeScript. Cada deploy es ruleta rusa. Refactorizar god files sin tests = imposible.

**Por qué duele**: no puedes mover rápido si cada cambio puede romper algo silenciosamente. La confianza del equipo (cuando crezca) cae.

## 5. God files

ClientApp.jsx (1573) + data.js (3649) + CrmPage.jsx (4524) = 9700 LOC mezclados. Cualquier cambio tiene blast-radius incierto. La función L() para i18n está duplicada 41 veces.

**Por qué duele**: todo cambio en estos archivos te bloquea pensando si rompes otra cosa. Velocidad de feature ↓ con el tiempo.

## 6. WhatsApp es bomba de tiempo

`whatsapp-web.js` es no oficial, fácilmente detectable, banneable. **Ya hubo banneo el 26-abr** y la respuesta fue parchear. La fragilidad es estructural, no parchear-able.

**Por qué duele**: oferta clave de la plataforma ("CRM 2-way con WhatsApp") cuelga de un hilo. Cualquier cliente puede perderlo en cualquier momento.

## 7. Secrets en chat (cultura)

Comments del propio `.env` admiten "rotar tras exposición en chat". Las keys exposed siguen activas. El reflejo cultural ante una key leaked no está consolidado.

**Por qué duele**: la próxima exposición pasará. Y la siguiente. Sin un playbook claro y reflejo automático de rotación, la deuda de seguridad crece.

## 8. Army incompleto

General + CRM commander operativos. OPS/DEV/FORMS son markdown sin handlers. Si Donna les delega, falla silenciosamente. Donna→Dona bridge (executor) no está construido.

**Por qué duele**: la promesa "Donna gestiona todo" tiene 60% de cobertura real. Cuando Alejandro decida usarlo en serio, va a chocar.

## 9. Sin API gateway entre frontend y enjambre-api

Cada llamada del frontend va directa al backend. No hay capa común para policy / rate limiting / auth verification / metrics.

**Por qué duele**: añadir rate limiting global, observability cross-cutting, o auth token rotation hay que hacerlo en N sitios. Mantenimiento exponencial.

## 10. Knowledge en cabeza (bus factor 1)

16 tablas DB sin uso documentado. 4 fuentes de "config tenant" (clients.config, email_config, manychat_config, whatsapp_config). Patrones convivientes (legacy y nuevo) sin clear deprecation. Onboarding cliente vive en cabeza.

**Por qué duele**: si Alejandro desaparece 1 mes, nadie reconstruye esto. Si entra un dev senior, le cuesta 2 meses arrancar. Este vault [[HOME]] es el primer paso para arreglarlo.

## Conexiones

- Lo bueno que tienes: [[Fortalezas]]
- Sangrando hoy: [[Bugs-Criticos]]
- Plan: [[Roadmap-Estrategico]]
