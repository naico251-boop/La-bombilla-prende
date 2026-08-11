# Sala de edición — contexto del proyecto

App web personal de Nicolás (La Bombilla, agencia de marketing digital y producción audiovisual en Apartadó, Urabá, Antioquia). Un solo archivo HTML autocontenido, sin dependencias externas, funciona 100% offline.

Nicolás trabaja solo, maneja varios clientes a la vez y habla en español colombiano con voseo. Responderle en ese registro.

---

## 1. Para qué existe

Nicolás procrastina la edición, que es la parte más pesada de su oficio: solitaria, larga, sin final claro, con cien microdecisiones por minuto de timeline. La app no es un gestor de tareas genérico — cada decisión de diseño ataca ese problema específico.

**Principios que NO se deben romper**

- Nunca "editar un video" como tarea. Siempre una fase de varios videos juntos (organizar los 6, después cortar silencios de los 6, después animar los 6). Cambiar de contexto es lo que agota.
- Una sola tarea visible a la vez. Ver 20 tareas paraliza. El resto va en una pista lateral horizontal.
- La decisión se toma antes, no en el momento. Si al abrir hay que elegir qué hacer, se pierde la mañana.
- Días flojos contemplados, no castigados. Filtro de baja energía para que un mal día no se pierda ni genere culpa.
- Nada de gamificación vacía. Racha y barras de progreso sí; puntos, insignias y confeti no.
- El rozamiento es intencional. "Saltar" cuesta un clic a propósito: obliga a preguntarse si de verdad se quiere otra tarea o se está esquivando esta.

**Riesgo permanente (importante)**

Construir la app se convierte en la forma elegante de procrastinar la edición. Ya pasó varias veces: días enteros mejorando la herramienta sin editar nada. Si Nicolás pide mejoras sin haber cerrado tareas, vale la pena señalarlo con franqueza y proponer un trato (cerrar X fases primero). No es rigidez: es literalmente el problema que la app existe para resolver.

**Sobre el uso de los datos**

El promedio de horas sirve para planear (¿cuánto puedo prometer?), no para evaluarse (¿estoy siendo suficiente?). Una semana floja tiene mil causas y el número no las distingue. No convertir la app en un marcador de desempeño.

---

## 2. Arquitectura (4 piezas)

| Pieza | Qué hace |
|---|---|
| Chat de Claude | Taller donde se fabrica y modifica el archivo. La app no vive acá. |
| GitHub Pages | Publica el archivo en una dirección web. Guarda el programa, no los datos. |
| Navegador | Donde la app corre. Cada aparato abre la misma URL. |
| Supabase | Base de datos en la nube. Guarda el progreso para que PC y celular vean lo mismo. |

- Repo: `github.com/naico251-boop/La-bombilla-prende`
- App: `https://naico251-boop.github.io/La-bombilla-prende/`
- Supabase: proyecto `komlqdnxndacimlrdxhq`, tabla `sala` con columnas `id` (text, PK) y `datos` (jsonb), RLS activo con política abierta.

**Reglas duras**

- El archivo debe llamarse exactamente `index.html` — minúscula, sin duplicar extensión. Windows oculta las extensiones y ya causó dos errores (`index.html.html` y `Index.html`). Debe existir un solo archivo con ese nombre en el repo.
- La clave de Supabase y el código de sala NO van en el código. El repo es público. Nicolás los pega en la app y quedan guardados en el navegador de cada aparato.
- La configuración de sincronización se pega una sola vez por aparato. Solo se pierde si se borran los datos de navegación.

**Ciclo de actualización**

1. Claude entrega el archivo.
2. Nicolás lo sube a GitHub: Add file → Upload files → Commit changes (mismo nombre = reemplaza).
3. Recarga con Ctrl+Shift+R. En iPhone, cerrar la app del selector y reabrir.
4. Reemplaza el archivo del proyecto de Claude, borrando el anterior.

El progreso no se pierde al actualizar: vive en Supabase, no en el archivo.

**Nota sobre trabajar con Claude Code:** Nicolás también hace cambios con Claude Code desde su PC, sobre la carpeta local `C:\VIDEO CLIENTES\LA BOMBILLA\app\Sala de Edición`. Claude Code edita el archivo local pero NO sube a GitHub por sí solo (salvo que el repo esté clonado con Git y credenciales configuradas): subir a GitHub es un paso manual aparte. Cuando se pide un cambio en el chat de Claude, conviene trabajar sobre la versión publicada en GitHub (la más reciente), no sobre una copia vieja.

---

## 3. Identidad visual (La Bombilla)

- Colores: Deep Teal `#0B2428` (fondo), Rich Charcoal `#1A1A1A` (tarjeta), Alabaster `#F5F5F0` (texto), Burnt Orange `#D36135` (acento), Moss `#6F8C5A` (positivo), Muted `#5E7378` (secundario).
- Tipografías: Cormorant Garamond (títulos), Archivo (interfaz), IBM Plex Mono (cifras y etiquetas).
- Las 5 fuentes van incrustadas en base64 dentro del archivo. No usar Google Fonts nunca: la app debe verse igual sin internet.
- Logo: el logo de La Bombilla va incrustado (base64/SVG) en tres lugares, siempre embebido, nunca como URL externa: encabezado, favicon e ícono de pantalla de inicio (`apple-touch-icon`, PNG 180×180). El ícono de inicio en iOS no respeta transparencia, así que lleva fondo teal `#0B2428` si el logo es claro.
- Metáfora general: sala de edición. "Toma actual", "pista" de clips, barra de progreso como barra de render, "corriendo" en vez de "activo".

---

## 4. Funcionalidad actual (completa)

**Chequeo de la mañana**

Lista de rutina arriba del todo, colapsable. Cada ítem tiene una nota editable ("monitorear frecuencia", "ver CPL de Passiflora"). Se desmarca sola cada día conservando textos y notas. No suma a la racha ni a la capacidad — es rutina, no producción. Sirve como rampa de entrada al día.

**Tarea actual y cronómetro**

- Cronómetro que cuenta HACIA ARRIBA desde 0 y no se detiene solo. Antes era un bloque de 45 min en cuenta regresiva; se cambió para no perder el dato de cuánto se demoró una tarea que pasa de la hora.
- Se calcula contra la hora real del reloj (`E.inicioBloque` = timestamp de inicio, `E.acumSeg` = lo ya acumulado), no contando segundos — así sigue corriendo con Chrome minimizado, la pestaña en segundo plano o la pantalla apagada.
- El tiempo se muestra como `H:MM:SS` cuando pasa de una hora (ej. `5:50:12`) y, debajo, una lectura en horas decimales (ej. `5.83 h`), porque Nicolás piensa el tiempo en horas.
- Notificación del sistema cada 30 min mientras corre (`E.avisoMult` lleva el último múltiplo avisado), para acordarse de pausar al terminar cada tarea. Además suena un aviso de tres bips (Web Audio, generado por la app, sin archivos externos). El sonido se desbloquea con el gesto de darle "Empezar" (los navegadores bloquean el audio automático hasta que hay una interacción). Un banner recordatorio aparece a los 5 s de abrir si faltan los permisos de notificación, y no vuelve a salir una vez concedidos.
- Botones: Empezar/Pausar · Listo · Saltar.
- Sobrevive a cerrar el navegador: al volver, el cronómetro sigue donde iba.
- Muestra "Fase X de Y" del proyecto de esa tarea.

**Cola y orden**

- Se ordena por holgura (tiempo disponible menos trabajo pendiente hasta la entrega), no por fecha de entrega. Un proyecto que entrega en 10 días pero necesita muchas fases es más urgente que uno que entrega en 3 y solo necesita exportar.
- `E.fijada` — tarea elegida a mano tocando su clip; manda sobre el cálculo.
- `E.pospuestas` — lo saltado hoy baja al fondo aunque tenga poca holgura.
- Ambas se reinician cada día.
- Sin fechas de entrega puestas, se respeta el orden manual.

**Modos del día**

Día normal / Día flojo (filtra solo tareas de energía baja) / Hoy no edito (saca el día del cálculo de capacidad sin ensuciar el aprendizaje del ritmo real).

**Crear trabajo**

- Proyecto de video: cliente, nombre, cantidad, ¿ya grabado o desde cero? → genera la cadena de fases automáticamente con tiempos escalados por cantidad. Se pueden destildar fases que no apliquen. Campo de fecha de entrega opcional al crear.
- Tarea suelta: para lo que no es video (campañas, CRM, reuniones, investigación). La fase es opcional — si se deja vacía, usa el tipo de trabajo como etiqueta. Campo de fecha de entrega opcional. Cada tarea suelta es su propia barra (su propio "proyecto"), así nunca se suma al conteo de fases de un proyecto de video.
- Campo Cliente en ambos formularios: casilla de texto con desplegable; se elige un cliente existente o se escribe uno nuevo, que se crea al vuelo. La lista de la derecha se agrupa por cliente.
- Al agregar un proyecto o una tarea, confirma con un aviso breve ("Proyecto agregado" / "Tarea agregada").

**Dividir un proyecto (tandas por trancón)**

- Botón "dividir" en la barra de progreso de cada proyecto de varios videos. Disponible en cualquier momento, no solo al crear.
- Resuelve el caso real: arrancás con 9 videos y 2 quedan trancados por insumos. Tocás "dividir", decís cuántos sacar, y arma un grupo aparte con su propia barra y su propia fecha de entrega (editable).
- Se lleva SOLO las fases pendientes de esos videos, repartidas por proporción (M/N del tiempo de cada fase). Las fases ya hechas se quedan en el original. A ambos les baja la cantidad en el nombre (9 → 7 y "— aparte (2)").
- La proporción es un estimado (asume que los videos pesan parecido); los tiempos se pueden ajustar a mano.

**Sugerir tandas**

- Botón "sugerir tandas" en la barra de progreso, solo bajo demanda (nunca salta solo).
- Estima cuántos videos por semana caben según la capacidad real (o las 3h base) y el peso de las fases pendientes, y propone el tamaño de tanda. No fija fechas: esas las pone Nicolás a mano con "dividir".
- Es capacidad de EDICIÓN, no cadencia del cliente: si hay tiempo de sobra dirá "te caben todos juntos". Y asume la semana entera dedicada a ese proyecto, así que el número real es un techo, no una promesa. Referencia para planear, no meta ni marcador.

**Cálculo y aprendizaje**

- Capacidad diaria: arranca en 3h/día (lunes a viernes; sábado no cuenta por defecto). Se reemplaza por el promedio real de los últimos 14 días cuando hay 5 o más días medidos (`capacidadMin()`).
- Estimaciones por fase: se corrigen con el factor real/estimado de las últimas 8 mediciones. Si el montaje tarda 1.5×, la app empieza a estimarlo así.
- Panorama (arriba): "Tenés Xh de trabajo y Yh disponibles hasta la última entrega. Vas corto Zh." Naranja si no alcanza, con sugerencia de qué mover.
- Capacidad (columna derecha): trabajado hoy/semana/mes con desglose por tipo, disponible en semana/mes/30 días, comprometido, y veredicto de cuántos proyectos nuevos caben. Incluye 30% extra por reuniones y correcciones.
- Tipos de trabajo: edición, rodaje, estrategia, montaje. Se clasifican automáticamente por fase (`FASE_TIPO`).

**Entregas y calendario**

- Una fecha de entrega por proyecto, editable desde la barra de progreso o al crear.
- Los proyectos/tareas sin fecha también aparecen siempre en "Render por proyecto"; las tareas sin cliente caen en un grupo "Sin proyecto" (render a prueba de huérfanos). La lista se agrupa por cliente, con un encabezado por cliente y un resumen de pendientes; el cliente sale del atributo explícito, o del prefijo antes de " — ", o del nombre.
- Los proyectos terminados desaparecen de la lista un día (24 h) después de quedar 100% hechos (`E.completados`). Si vuelve a entrar una tarea pendiente, el proyecto reaparece.
- Botón que descarga un `.ics` con la entrega como evento de día completo.
- Decisión de diseño: solo las entregas van al calendario, nunca bloques de trabajo. Agendar "editar Casa del Ser miércoles 8am" se rompe el primer día que un cliente llama, y un calendario que miente se deja de mirar.

**Sincronización y offline**

- Guarda siempre en local primero (funciona sin internet) y sube a Supabase si hay conexión.
- `almacen.leer()` compara el campo `sello` (timestamp) entre local y nube; gana el más reciente. Vuelve a comprobar el local después de esperar la nube, para no pisar un cambio recién guardado.
- Reintenta subir al evento `online`.
- Estado visible abajo: "Sincronizado" / "Guardado en este aparato" / "Sin internet — se sube al reconectar".

---

## 5. Formato de importación

```
@ PROYECTO | AAAA-MM-DD
PROYECTO | FASE | tarea | minutos | energia(alta/baja) | estado(hecho/pendiente)
```

Las líneas con `@` son fechas de entrega. Las demás, tareas. Las que empiezan con `#` se ignoran.

Al importar pregunta: Aceptar = agregar a lo existente · Cancelar = reemplazar todo. Solo bloquea duplicados que estén pendientes. Si una idéntica ya está hecha, es una tanda nueva y entra normal.

**Convención para trabajo recurrente**

Poner el período en el nombre del proyecto: `Santi — Tips IA Sem 30`, no `Tips de IA`. La app maneja una fecha de entrega por proyecto; un nombre eterno rompe el cálculo de holgura y las barras nunca se llenan.

**Fases estándar (actual)**

Las fases de video se redujeron a dos, para no fragmentar tanto el trabajo:

- Video ya grabado: **Montaje → Acabado**.
- Desde cero: antes van **Guion → Rodaje**.

Qué incluye cada una:
- Montaje: material, silencios / cortes de silencio, estructura, corte bruto y subtítulos.
- Acabado: animaciones, música y entrega.

**Tiempos base por video (se multiplican por la cantidad)**

| Fase | Min/video | Energía |
|---|---|---|
| Guion | 22 | alta |
| Rodaje | 38 | alta |
| Montaje | 60 | baja |
| Acabado | 55 | alta |

Nota: los proyectos viejos y las tareas de arranque (semilla) pueden conservar la cadena de 7 fases anterior (Material, Silencios, Corte bruto, Animación/Acabado, Subtítulos, Música, Entrega). El cambio de fases aplica a los proyectos nuevos creados desde el formulario.

Al armar una cola: partir cada cliente en proyectos separados si tienen fechas distintas (ej. `Passiflora — Animación 8B`, `Passiflora — Entrega CRM`, `Passiflora — Datos lotes`). Una reunión de entrega se parte en dos: preparar la demo (estrategia) + la reunión (montaje).

---

## 6. Detalles técnicos que importan

- Todo el estado vive en el objeto `E`, serializado a JSON. Al agregar campos nuevos, incluirlos en el estado inicial o los datos viejos rompen al cargar.
- `E.horasDia` acepta dos formatos: número suelto (versión vieja) u objeto por tipo de trabajo. `sumaEntre()` y `capacidadMin()` manejan ambos — no romper esa compatibilidad.
- `localStorage` no puede ser la única fuente: el cajón del navegador es distinto por dirección (`file://` vs `github.io`). Por eso existe Supabase. Esto ya causó un susto: el progreso "desapareció" al pasar del archivo local a GitHub. Las tareas de prueba creadas en el archivo local NO aparecen en la app publicada salvo que Supabase esté sincronizando ambas.
- No usar `localStorage` en artifacts de Claude — falla ahí. La app usa `window.storage` cuando está disponible y `localStorage` cuando no.
- Rejilla de escritorio: `grid-template-columns: minmax(0,1.5fr) minmax(320px,1fr)`. El `minmax(0,...)` es lo que evita que la pista de clips desborde la pantalla.
- Elementos con `display:flex` en CSS ignoran el atributo `hidden`. Hace falta `#id[hidden]{display:none}` explícito. Este bug hizo que los dos formularios se vieran a la vez (pasó dos veces, en `#modo-proy`/`#modo-suelta` y en el contenedor `#form`).
- El cronómetro nunca debe descontar/contar segundo a segundo: Chrome congela los timers de pestañas en segundo plano. Todo se calcula contra `Date.now()`.
- Los estimados solo se aprenden si la tarea pasó por el cronómetro (`t.inicio`/`E.inicioBloque` se marca al dar Empezar). Si Nicolás edita sin dar play, todos los cálculos mienten. Es la debilidad estructural del sistema y vale recordarlo.

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
| 24 | Botón "Dividir" en la barra de progreso: saca N videos a un grupo aparte con su propia fecha de entrega, llevándose solo las fases pendientes por proporción y bajando la cantidad del original; se puede dividir en cualquier momento. Cronómetro ahora muestra horas (H:MM:SS) más una lectura en horas decimales (ej. 5.83 h). |
| 25 | Botón "sugerir tandas" en la barra de progreso (solo bajo demanda): estima cuántos videos por semana caben según la capacidad real y el peso de las fases pendientes, y propone el tamaño de tanda; no fija fechas, esas van a mano con "dividir". |
| 26 | Campo "Cliente" en ambos formularios (elegir existente o crear nuevo); la lista de la derecha se agrupa por cliente con encabezado y resumen de pendientes; cada tarea suelta pasa a ser su propia barra, sin sumarse al conteo de fases de un proyecto de video. Agrupación por cliente explícito, o por el prefijo antes de " — ", o por nombre. |
| 27 | Aviso sonoro (tres bips por Web Audio, sin archivos externos) junto con la notificación de los 30 min; el sonido se desbloquea al darle Empezar. Banner recordatorio a los 5 s de abrir, solo si faltan los permisos de notificación, con botón para cerrarlo. |

Esta tabla se mantiene viva. Cada vez que se haga un cambio en la app, Claude debe entregar la fila nueva lista para pegar acá, con este formato:

```
| 28 | Descripción de qué se agregó o arregló, en una línea. |
```

Si el cambio fue grande o tiene varias partes, además de la fila entrega un párrafo corto explicando qué se hizo y por qué, para pegar al final de este documento.

---

## 8. Ideas evaluadas y descartadas (con la razón)

- Sincronización real con Google Calendar — requiere OAuth, proyecto en Google Cloud y pantalla de consentimiento; Google marca las apps sin verificar con advertencias. El `.ics` cubre el 90% del valor sin nada de eso.
- Agendar cada tarea a una hora fija — se rompe el primer día que un cliente llama, y arrastra en cascada todos los bloques siguientes.
- Integración directa con la API de Claude desde la app — el repo es público, la clave quedaría expuesta. El importar/exportar cubre el caso de uso real.
- Alojar en Vercel/Netlify con contraseña — se perdería el guardado ya funcionando. Los datos de Supabase ya son privados por código de sala.
- Sugerencia de tandas con regla fija (ej. "siempre lunes") — descartada: los días de entrega varían, y un calendario rígido miente. La sugerencia final es por capacidad y bajo demanda; las fechas las pone Nicolás a mano.

---

## 9. Pendientes / ideas no implementadas

**Prioritario: captura por voz desde el celular**

El problema real: las tareas se pierden cuando se le ocurren a Nicolás manejando, almorzando o camino a una dirección. Abrir la app y escribir no es opción en esos momentos, y el importar/exportar no cubre este caso.

Arquitectura propuesta — tabla `bandeja` en Supabase:

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
- Servidor MCP en Supabase Edge Functions — permite además que Claude lea la cola, revise la capacidad y agregue tareas conversando. Más potente, pero es una pieza más que se puede caer.

Recomendación: empezar por el atajo de Siri, que resuelve la captura con casi cero fragilidad. El MCP después, si el uso lo justifica.

**Otras ideas**

- Cierre de semana: resumen de lo hecho y lo que se corrió, para armar la semana siguiente en 15 minutos.
- Fases arrastrables para reordenar sin usar Saltar.
- Planificación mensual: el domingo antes de que arranque el mes, cargar todo de una.

---

## 10. Cómo trabajar en este proyecto

- El `index.html` actual debe estar subido como archivo del proyecto. Sin eso, Claude no puede ver cómo está hecho el código. Ojo: si el archivo del proyecto quedó viejo, conviene trabajar sobre la versión publicada en GitHub (la más reciente) para no editar código desactualizado.
- Al pedir un cambio, Claude modifica sobre ese archivo y devuelve la versión nueva completa.
- Nicolás la sube a GitHub y reemplaza el archivo del proyecto de Claude, borrando el anterior. Si el proyecto se queda con una versión vieja, el siguiente chat modifica código desactualizado y se pierden cambios.
- Para planear: Nicolás exporta la cola y la pega en el chat, o simplemente describe lo pendiente (clientes, cuántos videos, si están grabados, qué no es video, fechas) y Claude devuelve el bloque listo para importar.

**Cierre obligatorio de cada cambio**

Al terminar cualquier modificación de la app, Claude debe entregar siempre las tres cosas, sin que Nicolás las pida:

1. El archivo `index.html` nuevo, completo.
2. La fila para el historial (sección 7), lista para copiar y pegar.
3. El recordatorio de los dos pasos: subir el archivo a GitHub, y reemplazar el archivo del proyecto borrando el anterior.

No dar por cerrado un cambio sin esas tres. Si el proyecto se queda con una versión vieja del `index.html`, el siguiente chat modifica código desactualizado y se pierde trabajo.
