---
tags: [cliente, formacion, asesoria-suiza]
slug: asesorias-suiza
status: live
created: 2026-05-07
---

# Formación Portillo — "Cómo Trabajar en Suiza"

Programa completo de Portillo dentro del cliente [[AsesoriaSuiza-Portillo-Lukas]]. Vivo en `central.blackwolfsec.io/asesorias-suiza/formacion`.

## Estructura técnica (DB)

| Tabla | Registro | UUID |
|---|---|---|
| `training_routes` | "Cómo Trabajar en Suiza — Método Portillo" | `ead14774-808c-43a1-8634-9f07e16f0e9d` |
| `training_formations` | "Programa Completo Portillo" | `f0e8ef23-6b62-4a03-86dc-a99e4f914163` |
| `training_modules` | 6 módulos, ver abajo | (varios) |
| `training_lessons` | 31 lecciones distribuidas | (varios) |

Creado **2026-05-07** vía Management API. URLs de vídeo (`video_url`) pendientes — Alejandro las añade desde el admin.

## Syllabus completo

### 📚 Módulo 1 (position 1): Hoja de Ruta y Primeros Pasos

> El paso a paso inicial: bienvenida, auditoría experto nativo y preparación de la llamada estratégica.

1. El paso a paso a seguir
2. Vídeo 1 — Bienvenida e Instrucciones
3. Vídeo 2 — Cómo pasar la auditoría del experto nativo
4. Vídeo 3 — Preparación para la llamada estratégica

### 📚 Módulo 2 (position 2): Curso de Alemán A-1

> Fundamentos de alemán nivel A-1 — clases iniciales para el día a día y entrevistas.

1. A1 — Clase Inicial
2. A1 — Clase 2
3. A1 — Clase 3
4. A1 — Clase 4

### 📚 Módulo 3 (position 3): La Verdad Sin Filtros — Mentalidad y Estrategia

> Mentalidad correcta para no estrellarte: expectativas, errores caros, estrategia maestra invierno vs ciudad.

1. Lección 1.1 — Bienvenida: Esto no es suerte, es decisión
2. Lección 1.2 — Expectativas vs. Realidad: La película que te han contado
3. Lección 1.3 — Los errores que te van a costar miles de francos
4. Lección 1.4 — Checklist: Las herramientas para parecer un local
5. Lección 1.5 — El Timing y La Estrategia Maestra (Invierno vs Ciudad)

### 📚 Módulo 4 (position 4): El Mapa Legal — Permisos, Papeles y Plan de Salida

> Secuencia legal correcta, permisos, registro 14 días, CV suizo, cartas de recomendación, partner alojamiento.

1. Lección 2.1 — El Mapa General: La Secuencia Correcta
2. Lección 2.2 — Permisos de Residencia (Solo Pasaporte Europeo)
3. Lección 2.3 — El Registro: Los 14 días y el Papelito
4. Lección 2.4 — Tu CV Suizo y las Cartas de Recomendación (EL ORO)
5. Lección 2.5 — Planificar tu salida y el Partner de Alojamiento
6. Lección 2.6 — Checklist Final del Módulo 2

### 📚 Módulo 5 (position 5): Cómo Conseguir Trabajo Rápido — el Método Portillo

> La caza directa: Google Maps, lista increíble, email mágico, Probetag, rutina diaria y plan B presencial.

1. Lección 3.1 — Olvida los Portales, Bienvenido a la Caza Directa
2. Lección 3.2 — La Estrategia de Google Maps y la Lista Increíble
3. Lección 3.3 — El Email Mágico y la Simulación
4. Lección 3.4 — El Probetag: Tu examen final
5. Lección 3.5 — Rutina Diaria y Plan B: La Búsqueda Presencial
6. Lección 3.6 — Checklist Final del Módulo 3

### 📚 Módulo 6 (position 6): Tu Primer Mes — Alojamiento, Llegada y Lo Que Nadie Te Cuenta

> Aterrizaje real: alojamiento, partner, transporte, apps esenciales, normas de convivencia e integración.

1. Lección 4.1 — La Verdad sobre el Alojamiento (El reto real)
2. Lección 4.2 — El Partner de Alojamiento y CUIDADO con las Estafas
3. Lección 4.3 — Transporte: Cómo moverte sin que te multen
4. Lección 4.4 — Apps Esenciales y Normas de Convivencia
5. Lección 4.5 — Integración: No te quedes solo
6. Lección 4.6 — Cierre Final y Siguiente Paso

## Pendiente (Alejandro)

1. Añadir `video_url` a cada lección desde el admin (`/asesorias-suiza/formacion`).
2. Subir thumbnails / imágenes de cubierta a `training_routes.image_url` y `training_formations.image_url` si tienes assets.
3. Configurar `estimated_hours` en la formation cuando sepas duración total.
4. Confirmar que los alumnos del programa están en `portal_users` (allowlist OTP) — necesario para que entren al [[Portal-Infoproducto]] una vez se cablee Hito 6.

## Conexiones

- Cliente: [[AsesoriaSuiza-Portillo-Lukas]]
- Plataforma: [[Modulo-Formacion-Infoproducto]] · [[Portal-Infoproducto]]
- Schema DB: [[Database-Supabase]]
