# Retrospectiva: Historias de usuario FlowSync MVP — S4 vs. backlog en Linear

**Fecha de la retrospectiva:** 2026-08-04
**Fuentes comparadas:**
- `entregables/sesion-04/output.md` — inventario inicial de 19 historias generado en S4 (documento propio)
- Proyecto **FlowSync MVP** en Linear (equipo Ws-cecum) — 21 issues, WS-5 a WS-25, construidas por el mentor en el directo de S4

Leído con la perspectiva de después del directo de S4 y del contenido de S5.

---

## 1. ¿El alcance sigue ceñido al MVP, o se coló algo fuera de scope?

En general sí se ciñó — no encontré ninguna historia "inventada" que no tuviera anclaje en el PRD. Pero el ejercicio de Linear expone algo que mi documento no vio: **dos historias de proceso que el PRD pedía explícitamente y que yo nunca generé** porque estaba pensando solo en "como usuario, quiero X", no en el trabajo de equipo necesario para entregar con confianza:

- **WS-18 (spike técnico de Google Calendar API)** — el PRD §7 lo recomienda de forma explícita antes de comprometer la épica de sync. Yo fui directo a escribir las 6 historias de sincronización como si la API fuera un commodity conocido.
- **WS-25 (suite E2E)** — ancla directamente al criterio de éxito §6 ("completar el flujo entero sin ayuda externa"). Un criterio de éxito medible en el PRD y ninguna historia mía lo verificaba con una prueba automatizada.

Ninguna de las dos es "scope creep" — al contrario, son omisiones mías. El único caso que sí me genera dudas de disciplina de alcance es que WS-20 termina proponiendo un backfill retroactivo de tareas preexistentes (5.2c) como "candidata a post-MVP" — ese es exactamente el tipo de expansión silenciosa que el PRD advierte que "puede inflar el alcance", y el mentor lo contuvo etiquetándolo `needs-splitting` y dejándolo condicionado al spike. Yo no lo hubiera detectado a tiempo.

---

## 2. Criterios de aceptación incompletos o poco verificables

Aquí es donde más se nota la diferencia de madurez. Casos concretos donde mis AC eran genéricos y los de Linear son verificables:

- **Exportación CSV**: mi AC decía "formato CSV estándar con cabecera" — no verificable como test. WS-17 añade el AC real: qué pasa cuando el título o la descripción de una tarea contiene comas, comillas o saltos de línea. Ese es el caso que rompe un exportador CSV en producción y yo lo omití por completo.
- **Filtrado (US-3.1 / WS-14)**: yo solo especifiqué "sin filtro se ven todas". Linear distingue explícitamente "sin resultados de un filtro aplicado" vs. "estado vacío de cuenta nueva" — son dos UI states distintos que mi AC hubiera dejado a interpretación del dev.
- **Listado responsive (US-2.2 / WS-10)**: añadí rendimiento con 200 tareas pero no dije nada de mobile. WS-10 sí lo hace verificable ("usable sin scroll horizontal en móvil").
- **Tokens de Google (US-5.1 / WS-19)**: mi AC nunca mencionó que los tokens deben guardarse de forma segura y no ser accesibles por otros usuarios. Es un requisito de seguridad implícito que debí escribir yo mismo, no dejarlo para después.

---

## 3. Historias que cambiaron de naturaleza / criterios "asumidos" vs. correctamente bloqueados

Este es el hallazgo que más me hizo repensar cómo trabajé en S4: la historia de **ordenación por relevancia (US-3.2)**.

Yo la resolví con un "asumido" silencioso: orden ascendente por fecha límite, tareas sin fecha al final, recalculado en cada cambio. Sonaba razonable y seguí adelante.

**WS-15 hace algo distinto y más correcto**: marca la historia como `⚠️ Bloqueante de producto` porque el PRD §3.3 delega explícitamente ese criterio "a decisión del equipo durante refinamiento". No la estima en firme, no inventa una respuesta — la deja abierta hasta que alguien con autoridad de producto decida.

Con la perspectiva de hoy, mi "asumido" fue el error típico de IA generando historias sola: cuando el PRD dice "a decidir en refinamiento", la respuesta correcta no es rellenar el vacío con una suposición razonable y seguir — es señalar el vacío como bloqueante. Ese es un patrón que veo repetido en varias historias de sync (WS-20, WS-21, WS-22, WS-24): cosas que yo resolví con "(asumido)" y quedaron enterradas en el criterio de aceptación, Linear las etiqueta explícitamente como **"gap detectado en poke-holes"** y las deja condicionadas a decisiones del spike, no a mi criterio de redacción.

Otro cambio de naturaleza claro: **US-5.2 (tareas → eventos, WS-20)** pasó de ser una única historia monolítica en mi documento a estar formalmente dividida en tres sub-historias (5.2a/b/c) por superar el tamaño de un sprint (`needs-splitting`, `size:XL`) y por depender de una decisión que solo existe después del spike WS-18 — dependencia que en mi documento original no existía como concepto.

---

## 4. Historias nuevas que deberían estar y no aparecieron

Más allá de WS-18 y WS-25 (punto 1), Linear descubre **gaps funcionales reales dentro de historias existentes** que yo no capturé como AC:

- **Backfill al conectar Google (WS-20)**: ¿qué pasa con las tareas que *ya tenían* fecha límite antes de conectar la cuenta? Mi historia original solo cubría tareas creadas/editadas *después* de conectar. Es un salto de "primera conexión" que cualquier usuario real va a esperar y yo no lo contemplé.
- **Quitar fecha límite de una tarea sincronizada (WS-21)**: si el usuario borra la fecha al editar, ¿se borra el evento? Mi AC no decía nada — silencio total sobre ese camino.
- **Idempotencia en reintentos (WS-24)**: si un reintento de sync se dispara sobre una operación que en realidad ya tuvo éxito, ¿se duplica el evento? Yo mencioné reintentos pero nunca dije qué pasa si el reintento es innecesario.
- **Revocación de acceso desde el lado de Google (WS-24)**: si el usuario revoca el permiso desde su cuenta de Google (no desde FlowSync), mi historia de "fallo temporal de API" no distinguía ese caso del de una caída de servicio — y son flujos de UX completamente distintos (uno se resuelve solo, el otro exige reconectar).

Estos cuatro son, para mí, la evidencia más clara de que **generar historias de una sola pasada sobre el PRD tiene un techo**: hace falta un segundo pase adversarial ("poke holes") para encontrar las transiciones de estado que el PRD no verbaliza explícitamente pero que el dominio exige.

---

## 5. Priorización: mentor vs. mi documento — ¿quién acertó?

Mi documento no prioriza nada — es una lista plana en el orden de los módulos del PRD (Auth → Tareas → Organización → Exportación → Sync), sin señal de qué entra en qué sprint.

El mentor sí tomó una decisión de secuenciación real. En Linear, el primer ciclo (`Todo`, con `cycleId` asignado) contiene exactamente: **WS-5, 6, 7, 8 (todo EPIC 1 Auth) + WS-9, 10 (crear y listar tareas, no editar/borrar/cambiar estado) + WS-25 (E2E)**. Todo lo demás —edición, borrado, cambio de estado, organización completa, exportación, y toda la épica de sync incluido el spike— se queda en Backlog sin ciclo.

Es una estrategia de **"walking skeleton"**: entregar el camino feliz completo y probado de punta a punta (registro → login → crear tarea → verla listada → logout) antes de tocar cualquier otra cosa, incluso antes que operaciones tan "core" como editar o borrar una tarea.

**¿Acertó?** En dirección, sí, y más que mi enfoque de "todo el CRUD es un bloque": aísla el riesgo de integración (¿realmente funciona el flujo end-to-end?) antes de construir features adicionales sobre una base sin validar. Es coherente con el criterio de éxito §6 del PRD, que habla exactamente de ese flujo.

Donde sí discreparía con la secuenciación del mentor: **dejar el spike de Google Calendar (WS-18) fuera del primer ciclo**. Es `size:M`, no bloquea nada del camino feliz de auth+tareas, y es la pieza de mayor incertidumbre técnica de todo el backlog — el PRD mismo pide hacerlo temprano para poder estimar con fundamento el resto de la épica más grande y riesgosa. Corriéndolo en paralelo (otro dev, mismo ciclo) se hubiera reducido el riesgo de que la épica de sync explote en estimación más adelante, sin costar nada al camino feliz. El mentor optimizó puramente por "shippable end-to-end lo antes posible"; yo hubiera optimizado por "de-riesgo lo más incierto lo antes posible, en paralelo". Ambas son válidas — pero no son la misma decisión, y creo que la mía se ajusta mejor a lo que el propio PRD pide en §7.

---

## Resumen ejecutivo

| Pregunta | Conclusión |
|---|---|
| ¿Alcance ceñido al MVP? | Sí en general. Faltaron 2 historias de proceso (spike, E2E) que el PRD sí pedía. |
| ¿AC incompletos? | Sí — CSV escaping, filtro vs. estado vacío, mobile, seguridad de tokens. |
| ¿Historias que cambiaron de naturaleza? | US-3.2 (ordenación) pasó de "asumida" a bloqueante de producto; US-5.2 se partió en 3 por tamaño y dependencia del spike. |
| ¿Historias nuevas necesarias? | Backfill al conectar, quitar fecha de tarea sincronizada, idempotencia en reintentos, revocación de acceso desde Google. |
| ¿Quién acertó en priorización? | El mentor acertó en la estrategia de walking skeleton (auth + crear/listar + E2E primero). Yo hubiera adelantado el spike de Google en paralelo por ser barato y de alto valor informativo. |

---

## Cierre de Parte A: ajustes concretos si rehiciera el backlog hoy

**Ajuste:** Meter el spike de Google Calendar (equivalente a WS-18) en el primer sprint, en paralelo al trabajo de Auth + crear/listar tareas, en vez de dejarlo sin programar.
**Motivo:** Es barato (`size:M`), no bloquea el camino feliz de auth+crear+listar, y es la pieza de mayor incertidumbre técnica de todo el backlog. El PRD §7 pide explícitamente resolverlo temprano para poder estimar con fundamento el resto de la épica de sync — dejarlo fuera del primer ciclo pospone sin necesidad el descubrimiento del mayor riesgo del proyecto.

**Ajuste:** Incluir desde la primera pasada historias de proceso —spike técnico y suite E2E— ancladas a las secciones del PRD que las piden (§7 y §6 respectivamente), no solo historias "como usuario quiero X".
**Motivo:** Generé 19 historias centradas únicamente en funcionalidad de cara al usuario y nunca until que el PRD también manda entregables de proceso. Un backlog fiel al PRD tiene que transcribir tanto las features como los riesgos y criterios de éxito que el documento pide gestionar explícitamente.

**Ajuste:** Marcar como bloqueante — no resolver con "(asumido: ...)" — cualquier historia donde el PRD delega explícitamente una decisión al equipo o al refinamiento, como la ordenación por relevancia (US-3.2).
**Motivo:** Mi patrón de rellenar la ambigüedad con un "asumido" razonable y seguir adelante enmascara decisiones de producto reales. Cuando el PRD dice "a decisión del equipo en refinamiento", no es un vacío menor que yo deba resolver por mi cuenta — es una señal de que la historia no debería estimarse en firme todavía.

**Ajuste:** Hacer un segundo pase adversarial ("poke holes") sobre las historias de sincronización antes de darlas por completas, buscando transiciones de estado no verbalizadas explícitamente en el PRD.
**Motivo:** Los gaps más importantes que encontré —backfill al conectar, quitar fecha de una tarea sincronizada, idempotencia en reintentos, revocación de acceso desde Google— no aparecieron en la primera generación de historias. Solo salieron a la luz cuando alguien cuestionó cada criterio de aceptación buscando el caso límite, no como parte del pase inicial sobre el PRD.