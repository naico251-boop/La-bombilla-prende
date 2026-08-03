# Sala de edición — contexto del proyecto

App web personal de Nicolás (La Bombilla, agencia de marketing digital y producción audiovisual en Apartadó, Urabá, Antioquia). Un solo archivo HTML autocontenido, sin dependencias externas, funciona 100% offline.

Nicolás trabaja solo, maneja varios clientes a la vez y habla en español colombiano con voseo. Responderle en ese registro.

---

## 1. Para qué existe

Nicolás procrastina la edición, que es la parte más pesada de su oficio: solitaria, larga, sin final claro, con cien microdecisiones por minuto de timeline. La app no es un gestor de tareas genérico — cada decisión de diseño ataca ese problema específico.

### Principios que NO se deben romper

- Nunca "editar un video" como tarea. Siempre una fase de varios videos juntos (organizar los 6, después cortar silencios de los 6, después animar los 6). Cambiar de contexto es lo que agota.
- Una sola tarea visible a la vez. Ver 20 tareas paraliza. El resto va en una pista lateral horizontal.
- La decisión se toma antes, no en el momento. Si al abrir hay que elegir qué hacer, se pierde la mañana.
- Días flojos contemplados, no castigados. Filtro de baja energía para que un mal día no se pierda ni genere culpa.
- Nada de gamificación vacía. Racha y barras de progreso sí; puntos, insignias y confeti no.
- El rozamiento es intencional. "Saltar" cuesta un clic a propósito: obliga a preguntarse si de verdad se quiere otra tarea o se está esquivando esta.

### Riesgo permanente (importante)

Construir la app se convierte en la forma elegante de procrastinar la edición. Ya pasó varias veces: días enteros mejorando la herramienta sin editar nada. Si Nicolás pide mejoras sin haber cerrado tareas, vale la pena señalarlo con franqueza y proponer un trato (cerrar X fases primero). No es rigidez: es literalmente el problema que la app existe para resolver.

### Sobre el uso de los datos

El promedio de horas sirve para planear (¿cuánto puedo prometer?), no para evaluarse (¿estoy siendo suficiente?). Una semana floja tiene mil causas y el número no las distingue. No convertir la app en un marcador de desempeño.

---

## 2. Arquitectura (4 piezas)

| Pieza | Qué hace |
|---|---|
| Chat de Claude | Taller donde se fabrica y modifica el archivo. La app no vive acá. |
| GitHub Pages | Publica el archivo en una dirección web. Guarda el programa, no los datos. |
| Navegador | Donde la app corre. Cada aparato abre la misma URL. |
| Supabase | Base de datos en la nube. Guarda el progreso para que PC y celular vean lo mismo. |

Repo: `github.com/naico251-boop/La-bombilla-prende`
App: `https://naico251-boop.github.io/La-bombilla-prende/`
Supabase: proyecto `komlqdnxndacimlrdxhq`, tabla `sala` con columnas `id` (text, PK) y `datos` (jsonb), RLS activo con política abierta.

### Reglas duras

- El archivo debe llamarse exactamente `index.html` — minúscula, sin duplicar extensión. Windows oculta las extensiones y ya causó dos errores (`index.html.html` y `Index.html`). Debe existir un solo archivo con ese nombre en el repo.
- La clave de Supabase y el código de sala NO van en el código. El repo es público. Nicolás los pega en la app y quedan guardados en el navegador de cada aparato.
- La configuración de sincronización se pega una sola vez por aparato. Solo se pierde si se borran los datos de navegación.

### Ciclo de actualización

1. Claude entrega el archivo.
2. Nicolás lo sube a GitHub: Add file → Upload files → Commit changes (mismo nombre = reemplaza).
3. Recarga con Ctrl+Shift+R. En iPhone, cerrar la app del selector y reabrir.
4. Reemplaza el archivo del proyecto de Claude, borrando el anterior.

El progreso no se pierde al actualizar: vive en Supabase, no en el archivo.

---

## 3. Identidad visual (La Bombilla)

Colores: Deep Teal `#0B2428` (fondo), Rich Charcoal `#1A1A1A` (tarjeta), Alabaster `#F5F5F0` (texto), Burnt Orange `#D36135` (acento), Moss `#6F8C5A` (positivo), Muted `#5E7378` (secundario).

Tipografías: Cormorant Garamond (títulos), Archivo (interfaz), IBM Plex Mono (cifras y etiquetas).

Las 5 fuentes van incrustadas en base64 dentro del archivo. No usar Google Fonts nunca: la app debe verse igual sin internet.

Isotipo: el logo de La Bombilla, incrustado en base64 dentro del archivo en tres lugares — encabezado (como `<img class="bombilla">`, PNG en base64), favicon (PNG en base64) e ícono de pantalla de inicio (`apple-touch-icon`, PNG 180×180). Todo embebido, nunca como enlace externo, para que la app se vea igual sin internet.

Metáfora general: sala de edición. "Toma actual", "pista" de clips, barra de progreso como barra de render, "corriendo" en vez de "activo".

---

## 4. Funcionalidad actual (completa)

### Chequeo de la mañana

Lista de rutina arriba del todo, colapsable. Cada ítem tiene una nota editable ("monitorear frecuencia", "ver CPL de Passiflora"). Se desmarca sola cada día conservando textos y notas. No suma a la racha ni a la capacidad — es rutina, no producción. Sirve como rampa de entrada al día.

### Tarea actual y cronómetro

Cronómetro que cuenta hacia arriba, calculado contra la hora real del reloj (`E.inicioBloque` = timestamp de arranque, `E.acumSeg` = segundos ya acumulados de tramos anteriores), no contando segundos uno por uno — así sigue corriendo con Chrome minimizado o la pantalla apagada.

- La barra de progreso se llena contra el estimado de la tarea (tope 100%), como una barra de render.
- Muestra en qué fase va dentro del proyecto de esa tarea, ej. "Montaje · Fase 1 de 2".
- Notificación del sistema a los 30 minutos de estar corriendo ("Media hora corriendo"), para invitar a pausar cuando se termine el corte.
- Botones: Empezar/Pausar · Listo · Saltar.
- Sobrevive a cerrar el navegador: al volver, el cronómetro sigue donde iba.

### Cola y orden

Se ordena por holgura (tiempo disponible menos trabajo pendiente hasta la entrega), no por fecha de entrega. Un proyecto que entrega en 10 días pero necesita 7 fases es más urgente que uno que entrega en 3 y solo necesita exportar.

- `E.fijada` — tarea elegida a mano tocando su clip; manda sobre el cálculo.
- `E.pospuestas` — lo saltado hoy baja al fondo aunque tenga poca holgura.
- Ambas se reinician cada día.
- Sin fechas de entrega puestas, se respeta el orden manual.

### Modos del día

Día normal / Día flojo (filtra solo tareas de energía baja) / Hoy no edito (saca el día del cálculo de capacidad sin ensuciar el aprendizaje del ritmo real).

### Crear trabajo

- Proyecto de video: nombre, cantidad, ¿ya grabado o desde cero? → genera la cadena de fases automáticamente con tiempos escalados por cantidad. Se pueden destildar fases que no apliquen. Fecha de entrega opcional.
- Tarea suelta: para lo que no es video (campañas, CRM, reuniones, investigación). El campo de proyecto es un input con lista de sugerencias de los proyectos actuales: si escribís un cliente/proyecto que no existe, se crea al vuelo. La fase es opcional — si se deja vacía, usa el tipo de trabajo como etiqueta. Fecha de entrega opcional.
- Al agregar un proyecto o una tarea, aparece un aviso corto de confirmación ("Proyecto agregado" / "Tarea agregada").

### Proyectos terminados

Cuando un proyecto queda 100% hecho, se guarda el momento en que se completó (`E.completados`). Ese día seguís viendo su barra llena; al cumplir 24 horas, el proyecto se retira solo de la lista de "Render por proyecto" para que no se acumule. Si al proyecto le entra después una tarea pendiente nueva (por ejemplo al importar una tanda nueva), reaparece automáticamente. Ocultarlo no afecta los cálculos: las tareas siguen guardadas, solo se dejan de pintar.

### Cálculo y aprendizaje

- Capacidad diaria: arranca en 3h/día (lunes a viernes; sábado no cuenta por defecto). Se reemplaza por el promedio real de los últimos 14 días cuando hay 5 o más días medidos.
- Estimaciones por fase: se corrigen con el factor real/estimado de las últimas 8 mediciones. Si el corte bruto tarda 1.5×, la app empieza a estimarlo así.
- Panorama (arriba): "Tenés Xh de trabajo y Yh disponibles hasta la última entrega. Vas corto Zh." Naranja si no alcanza, con sugerencia de qué mover.
- Capacidad (columna derecha): trabajado hoy/semana/mes con desglose por tipo, disponible en semana/mes/30 días, comprometido, y veredicto: "Con 28h libres te caben — Proyecto de 4 videos: 2 · Campaña de pauta: 6 · CRM completo: 3". Incluye 30% extra por reuniones y correcciones.
- Tipos de trabajo: edición, rodaje, estrategia, montaje. Se clasifican automáticamente por fase (`FASE_TIPO`).

### Entregas y calendario

- Una fecha de entrega por proyecto, editable desde la barra de progreso.
- Botón que descarga un `.ics` con la entrega como evento de día completo.
- Decisión de diseño: solo las entregas van al calendario, nunca bloques de trabajo. Agendar "editar Casa del Ser miércoles 8am" se rompe el primer día que un cliente llama, y un calendario que miente se deja de mirar.

### Sincronización y offline

- Guarda siempre en local primero (funciona sin internet) y sube a Supabase si hay conexión.
- `almacen.leer()` compara el campo `sello` (timestamp) entre local y nube; gana el más reciente.
- Reintenta subir al evento `online`.
- Estado visible abajo: "Sincronizado" / "Guardado en este aparato" / "Sin internet — se sube al reconectar".

---

## 5. Formato de importación

```
@ PROYECTO | AAAA-MM-DD
PROYECTO | FASE | tarea | minutos | energia(alta/baja) | estado(hecho/pendiente)
```

Las líneas con `@` son fechas de entrega. Las demás, tareas. Las que empiezan con `#` se ignoran.

Al importar pregunta: Aceptar = agregar a lo existente · Cancelar = reemplazar todo.

Solo bloquea duplicados que estén pendientes. Si una idéntica ya está hecha, es una tanda nueva y entra normal.

### Convención para trabajo recurrente

Poner el período en el nombre del proyecto: `Santi — Tips IA Sem 30`, no `Tips de IA`. La app maneja una fecha de entrega por proyecto; un nombre eterno rompe el cálculo de holgura y las barras nunca se llenan.

### Fases estándar

El generador de "Proyecto de video" crea la cadena automáticamente:

- Video ya grabado: Montaje → Acabado.
- Desde cero: antes van Guion → Rodaje.

El campo `FASE` del formato de importación es texto libre: al pegar una cola a mano podés usar las fases que quieras (Material, Silencios, Corte bruto, Subtítulos, Música, Entrega, etc.) si querés partir el trabajo más fino. Las dos fases de arriba son solo lo que genera el formulario automático. Los proyectos viejos que ya están en la cola conservan las fases con que se crearon.

### Tiempos base por video (se multiplican por la cantidad)

| Fase | Min/video | Energía | Contenido |
|---|---|---|---|
| Guion | 22 | alta | (solo "desde cero") |
| Rodaje | 38 | alta | (solo "desde cero") |
| Montaje | 60 | baja | material, silencios, estructura y subtítulos |
| Acabado | 55 | alta | animaciones, música y entrega |

Al armar una cola: partir cada cliente en proyectos separados si tienen fechas distintas (ej. `Passiflora — Animación 8B`, `Passiflora — Entrega CRM`, `Passiflora — Datos lotes`). Una reunión de entrega se parte en dos: preparar la demo (estrategia) + la reunión (montaje).

---

## 6. Detalles técnicos que importan

- Todo el estado vive en el objeto `E`, serializado a JSON. Al agregar campos nuevos (por ejemplo `E.completados`), incluirlos en el estado inicial o los datos viejos rompen al cargar.
- `E.horasDia` acepta dos formatos: número suelto (versión vieja) u objeto por tipo de trabajo. `sumaEntre()` y `capacidadMin()` manejan ambos — no romper esa compatibilidad.
- `localStorage` no puede ser la única fuente: el cajón del navegador es distinto por dirección (`file://` vs `github.io`). Por eso existe Supabase. Esto ya causó un susto: el progreso "desapareció" al pasar del archivo local a GitHub.
- No usar `localStorage` en artifacts de Claude — falla ahí. La app usa `window.storage` cuando está disponible y `localStorage` cuando no.
- Rejilla de escritorio: `grid-template-columns: minmax(0,1.5fr) minmax(320px,1fr)`. El `minmax(0,...)` es lo que evita que la pista de clips desborde la pantalla.
- Elementos con `display:flex` en CSS ignoran el atributo `hidden`. Hace falta `#id[hidden]{display:none}` explícito. Este bug hizo que los dos formularios se vieran a la vez.
- El cronómetro nunca debe contar segundo a segundo con un timer de JS que sume: Chrome congela los timers de pestañas en segundo plano. Se calcula siempre contra la hora real (`E.inicioBloque` + `E.acumSeg`), y el reloj visible solo repinta lo que ya ocurrió.
- Los estimados solo se aprenden si la tarea pasó por el cronómetro (`E.inicioBloque` se marca al dar Empezar; el tiempo real corrido se lee con `segsTranscurridos()`). Si Nicolás edita sin dar play, todos los cálculos mienten. Es la debilidad estructural del sistema y vale recordarlo.

---

## 7. Historial de versiones

| # | Qué se hizo |
|---|---|
| 1 | Versión inicial: tarea única visible, pista de fases, temporizador de 45 min, modo día flojo, racha, registro del día. |
| 2 | Arreglo del guardado (faltaba el parámetro `shared`) e indicador de estado. |
| 3 | Botón de agregar tareas. |
| 4 | Clips de la pista tocables para adelantar una tarea urgente. |
| 5 | Fases reales de edición (el material ya venía grabado): Material → Silencios → Corte bruto → Animación → Subtítulos → Música → Entrega. Mitad de las fases quedaron de baja energía, lo que hizo útil el modo día flojo. |
| 6 | Layout de escritorio en dos columnas + guardado con respaldo en `localStorage` para funcionar fuera de Claude. |
| 7 | Fuentes incrustadas en base64 → 100% offline, cero llamadas a servidores externos. |
| 8 | Logo de La Bombilla (SVG en encabezado + ícono de pantalla de inicio) y sincronización con Supabase. |
| 9 | Temporizador basado en hora real → sigue corriendo con Chrome minimizado. Notificación del sistema al terminar. |
| 10 | Formulario partido en dos pestañas: "Proyecto de video" (genera fases solas) y "Tarea suelta". Ya no se puede crear una tarea gigante sin fases. |
| 11 | Arreglo del desborde horizontal en escritorio y reorganización de columnas: tarea + pista a la izquierda; agregar, progreso y registro a la derecha. |
| 12 | Exportar / Importar cola en texto plano — el canal para planear con Claude. |
| 13 | Fechas de entrega, holgura y panorama. Capacidad diaria aprendida del uso real. Estimaciones que se corrigen solas. Botón "Hoy no edito". Exportación de entregas a `.ics`. |
| 14 | Panel de Capacidad: trabajado por día/semana/mes, disponible, comprometido, y veredicto de cuántos proyectos nuevos caben. |
| 15 | Tipos de trabajo (edición, rodaje, estrategia, montaje). La app dejó de asumir que todo era edición. |
| 16 | Importar con opción de agregar en vez de reemplazar. Antiduplicado solo contra tareas pendientes. |
| 17 | Fechas de entrega dentro del formato de importación (líneas con `@`). |
| 18 | Arreglo: la elección manual (`fijada` / `pospuestas`) ahora manda sobre el orden por holgura. Antes, saltar solo avanzaba dentro del mismo proyecto. |
| 19 | Arreglo del CSS que mostraba los dos formularios a la vez. Fase opcional en tarea suelta. |
| 20 | Chequeo de la mañana: rutina diaria con notas, se desmarca sola, no cuenta para racha ni capacidad. |
| 21 | Tarea suelta puede crear proyecto/cliente nuevo al vuelo; confirmación al agregar tarea o proyecto; arreglo del guardado de tareas sueltas; "Fase X de Y" en el bloque del cronómetro; fases de video reducidas a dos (Montaje y Acabado); cronómetro que cuenta hacia arriba con hora real; notificación del sistema a los 30 min; logo nuevo de La Bombilla en encabezado, favicon e ícono de inicio. |
| 22 | Los proyectos/tareas sin fecha de entrega ya aparecen siempre en "Render por proyecto" (render a prueba de huérfanos); fecha de entrega opcional al crear un Proyecto de video o Tarea suelta, sin pisar la fecha de un proyecto existente si se deja vacío. |
| 23 | Los proyectos terminados desaparecen de la lista un día (24 h) después de quedar 100% hechos. Se guarda el momento en que se completó cada uno (`E.completados`); si vuelve a entrar una tarea pendiente, el proyecto reaparece. |

Esta tabla se mantiene viva. Cada vez que se haga un cambio en la app, Claude debe entregar la fila nueva lista para pegar acá, con este formato:

```
| 24 | Descripción de qué se agregó o arregló, en una línea. |
```

Si el cambio fue grande o tiene varias partes, además de la fila entrega un párrafo corto explicando qué se hizo y por qué, para pegar al final de este documento.

---

## 8. Ideas evaluadas y descartadas (con la razón)

- Sincronización real con Google Calendar — requiere OAuth, proyecto en Google Cloud y pantalla de consentimiento; Google marca las apps sin verificar con advertencias. El `.ics` cubre el 90% del valor sin nada de eso.
- Agendar cada tarea a una hora fija — se rompe el primer día que un cliente llama, y arrastra en cascada todos los bloques siguientes.
- Integración directa con la API de Claude desde la app — el repo es público, la clave quedaría expuesta. El importar/exportar cubre el caso de uso real.
- Alojar en Vercel/Netlify con contraseña — se perdería el guardado ya funcionando. Los datos de Supabase ya son privados por código de sala.

## 9. Pendientes / ideas no implementadas

### Prioritario: captura por voz desde el celular

El problema real: las tareas se pierden cuando se le ocurren a Nicolás manejando, almorzando o camino a una dirección. Abrir la app y escribir no es opción en esos momentos, y el importar/exportar no cubre este caso.

Arquitectura propuesta — tabla `bandeja` en Supabase:

En vez de escribir directo sobre el JSON del estado (que obliga a leer, modificar y devolver todo el objeto), crear una tabla aparte:

```sql
create table bandeja (
  id bigserial primary key,
  texto text,
  destino text default 'cola',   -- 'cola' | 'chequeo'
  creado timestamptz default now(),
  procesado boolean default false
);
```

Agregar es un simple INSERT. La app, al cargar, lee lo no procesado, lo suma a la cola o al chequeo, y marca `procesado = true`. Desacopla la captura del estado, que es lo que lo hace simple y robusto.

Dos caminos para llenarla, no excluyentes:

- Atajo de Siri (iOS Shortcuts) — hace un POST directo a la API REST que Supabase ya expone. No necesita servidor nuevo. "Oye Siri, anotar en la bandeja" → dicta → queda guardado. Es el camino más simple y el más difícil de romper.
- Servidor MCP en Supabase Edge Functions — permite además que Claude lea la cola, revise la capacidad y agregue tareas conversando. Más potente, pero es una pieza más que se puede caer (protocolo MCP en evolución, límites del plan gratis, depuración a ciegas desde los logs).

Recomendación: empezar por el atajo de Siri, que resuelve la captura con casi cero fragilidad. El MCP después, si el uso lo justifica.

### Otras ideas

- Cierre de semana: resumen de lo hecho y lo que se corrió, para armar la semana siguiente en 15 minutos.
- Fases arrastrables para reordenar sin usar Saltar.
- Planificación mensual: el domingo antes de que arranque el mes, cargar todo de una.

---

## 10. Cómo trabajar en este proyecto

- El `index.html` actual debe estar subido como archivo del proyecto. Sin eso, Claude no puede ver cómo está hecho el código.
- Al pedir un cambio, Claude modifica sobre ese archivo y devuelve la versión nueva completa.
- Nicolás la sube a GitHub y reemplaza el archivo del proyecto de Claude, borrando el anterior. Si el proyecto se queda con una versión vieja, el siguiente chat modifica código desactualizado y se pierden cambios.
- Para planear: Nicolás exporta la cola y la pega en el chat, o simplemente describe lo pendiente (clientes, cuántos videos, si están grabados, qué no es video, fechas) y Claude devuelve el bloque listo para importar.

### Cierre obligatorio de cada cambio

Al terminar cualquier modificación de la app, Claude debe entregar siempre las tres cosas, sin que Nicolás las pida:

1. El archivo `index.html` nuevo, completo.
2. La fila para el historial (sección 7), lista para copiar y pegar.
3. El recordatorio de los dos pasos: subir el archivo a GitHub, y reemplazar el archivo del proyecto borrando el anterior.

No dar por cerrado un cambio sin esas tres. Si el proyecto se queda con una versión vieja del `index.html`, el siguiente chat modifica código desactualizado y se pierde trabajo.
