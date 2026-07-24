# Sala de edición — contexto del proyecto

App web personal de Nicolás (La Bombilla, agencia de marketing y producción audiovisual en Apartadó, Urabá). Un solo archivo HTML, sin dependencias externas, funciona offline.

---

## Para qué existe

Nicolás trabaja solo y procrastina la edición, que es la parte más pesada de su oficio. La app no es un gestor de tareas genérico: está diseñada contra ese problema específico.

**Principios de diseño que NO se deben romper:**

1. **Nunca "editar un video" como tarea.** Siempre una fase de varios videos juntos (guionizar los 6, después grabar los 6, después cortar los 6). Cambiar de cabeza es lo que agota; agrupar por fase rinde más.
2. **Una sola tarea visible a la vez.** Ver 20 tareas paraliza. El resto va en una pista lateral.
3. **La decisión se toma antes, no en el momento.** Si al abrir hay que elegir qué hacer, se pierde la mañana.
4. **Días flojos contemplados, no castigados.** Filtro de tareas de baja energía para que un mal día no se pierda ni genere culpa.
5. **Nada de gamificación vacía.** Racha y barras de progreso sí; puntos, insignias y confeti no.

**Riesgo permanente:** construir la app se convierte en la forma elegante de procrastinar la edición. Si Nicolás pide mejoras sin haber cerrado tareas, vale la pena señalarlo.

---

## Arquitectura (4 piezas)

| Pieza | Qué hace |
|---|---|
| **Chat de Claude** | Taller donde se fabrica y modifica el archivo. La app no vive acá. |
| **GitHub Pages** | Publica el archivo en una dirección web. Guarda **el programa**, no los datos. |
| **Navegador** | Donde la app corre. Cada aparato abre la misma URL. |
| **Supabase** | Base de datos en la nube. Guarda **el progreso** para que PC y celular vean lo mismo. |

- Repo: `github.com/naico251-boop/La-bombilla-prende`
- App: `https://naico251-boop.github.io/La-bombilla-prende/`
- El archivo debe llamarse exactamente `index.html` (minúscula, sin duplicar extensión).
- Supabase: tabla `sala` con columnas `id` (text, PK) y `datos` (jsonb), RLS activo con política abierta.
- La clave y el código de sala los pega el usuario en la app; **no van en el código** (el repo es público).

**Ciclo de actualización:** Claude entrega archivo → Nicolás lo sube a GitHub (Add file → Upload files → Commit) → recarga con Ctrl+Shift+R. El progreso no se pierde porque vive en Supabase.

---

## Identidad visual (La Bombilla)

- Deep Teal `#0B2428` (fondo), Rich Charcoal `#1A1A1A`, Alabaster `#F5F5F0`, Burnt Orange `#D36135` (acento), Moss `#6F8C5A` (positivo)
- Tipografías: Cormorant Garamond (títulos), Archivo (interfaz), IBM Plex Mono (cifras)
- **Las 5 fuentes van incrustadas en base64 dentro del archivo.** No usar Google Fonts: la app debe funcionar sin internet.
- Isotipo: bombilla con el filamento como forma de onda de audio. Va en el encabezado (SVG) y como ícono de pantalla de inicio (PNG base64).
- Metáfora general: sala de edición. "Toma actual", "pista", "render", barra de progreso como barra de render.

---

## Funcionalidad actual

**Tarea y tiempo**
- Bloque de 45 min. Calculado contra la hora real del reloj (`E.finBloque`), no contando segundos — así sigue corriendo con Chrome minimizado.
- Notificación del sistema al terminar el bloque.
- Botones: Empezar/Pausar · Listo · Saltar.

**Cola**
- Se ordena por **holgura** (tiempo disponible menos trabajo pendiente hasta la entrega), no por fecha de entrega.
- `E.fijada` — tarea elegida a mano, manda sobre el cálculo.
- `E.pospuestas` — lo saltado hoy baja al fondo. Ambas se reinician cada día.

**Modos**
- Día normal / Día flojo (filtra energía baja) / Hoy no edito (saca el día del cálculo sin ensuciar el aprendizaje).

**Proyectos**
- Se crean desde "Proyecto de video": nombre, cantidad, ¿grabado o desde cero? → genera las fases automáticamente con tiempos escalados.
- "Tarea suelta" para lo que no es video (campañas, CRM, reuniones), con selector de tipo.

**Cálculo y aprendizaje**
- Capacidad diaria: arranca en 3h/día, se reemplaza por el promedio real de los últimos 14 días cuando hay 5 o más medidos.
- Estimaciones por fase: se corrigen con el factor real/estimado de las últimas 8 mediciones.
- Panorama arriba: "Tenés Xh de trabajo y Yh disponibles. Vas corto Zh."
- Sección Capacidad: trabajado (hoy/semana/mes), disponible, comprometido, y veredicto de cuántos proyectos nuevos caben.
- Tipos de trabajo: edición, rodaje, estrategia, montaje. Se clasifican por fase (`FASE_TIPO`).

**Entregas**
- Una fecha por proyecto. Botón que descarga `.ics` con la entrega como evento de día completo (solo entregas, nunca bloques de trabajo en el calendario).

**Importar / Exportar** — el canal para planear con Claude.

---

## Formato de importación

```
@ PROYECTO | AAAA-MM-DD
PROYECTO | FASE | tarea | minutos | energia(alta/baja) | estado(hecho/pendiente)
```

Las líneas con `@` son fechas de entrega. Las demás, tareas. Las que empiezan con `#` se ignoran.

Al importar pregunta: **Aceptar** = agregar a lo existente · **Cancelar** = reemplazar todo.
Solo bloquea duplicados que estén **pendientes** (si la anterior ya está hecha, es una tanda nueva y entra).

**Convención para trabajo recurrente:** poner el período en el nombre del proyecto (`Santi — Tips IA Sem 30`, no `Tips de IA`). La app maneja una fecha de entrega por proyecto; un nombre eterno rompe el cálculo de holgura.

**Fases estándar (video ya grabado):** Material → Silencios → Corte bruto → Acabado/Animación → Subtítulos → Música → Entrega.
Si hay que grabarlo, antes van: Guion → Rodaje.

**Tiempos base por video** (se multiplican por la cantidad): Material 11min · Silencios 15 · Corte bruto 22 · Acabado 38 · Subtítulos 15 · Música 11 · Entrega 8.

---

## Detalles técnicos que importan

- **Todo el estado en el objeto `E`**, serializado a JSON. Al agregar campos nuevos, incluirlos en el estado inicial o los datos viejos rompen.
- `almacen.leer()` compara `sello` (timestamp) entre local y nube; gana el más reciente.
- Guarda siempre en local primero (funciona offline) y sube a Supabase si hay conexión. Reintenta al evento `online`.
- `E.horasDia` acepta dos formatos: número suelto (versión vieja) u objeto por tipo. Las funciones de suma manejan ambos — no romper esa compatibilidad.
- No usar `localStorage` como única fuente: el cajón del navegador es distinto por dirección (`file://` vs `github.io`), por eso existe Supabase.
- Rejilla de escritorio: `minmax(0, 1.5fr)` en las columnas es lo que evita que la pista de clips desborde la pantalla.

---

## Pendientes / ideas no implementadas

- Cierre de semana: resumen de lo hecho y lo que se corrió.
- Fases arrastrables para reordenar sin usar Saltar.
- Sincronización real con Google Calendar (evaluada y descartada: requiere OAuth, Google Cloud y pantalla de consentimiento; el `.ics` cubre el 90% del valor).
- Integración directa con la API de Claude desde la app (descartada: el repo es público, la clave quedaría expuesta).

---

## Cómo trabajar en este proyecto

1. El `index.html` actual debe estar subido como archivo del proyecto.
2. Al pedir un cambio, Claude modifica sobre ese archivo y devuelve la versión nueva.
3. Nicolás la sube a GitHub **y reemplaza el archivo del proyecto**. Si el proyecto se queda con una versión vieja, el siguiente chat modifica código desactualizado y se pierden cambios.
4. Para planear: Nicolás exporta la cola y la pega en el chat, o describe lo pendiente y Claude devuelve el bloque listo para importar.
