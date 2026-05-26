# Diseño — acción `refine` para el Council

> Estado: **implementado** — v2.2 del SKILL, 2026-05-22. Este documento es la especificación; el sistema (`SKILL.md` núcleo + `actions/`, `schemas/`, comandos, `CLAUDE.md`) ya la refleja. Documento de diseño arquitectónico de una acción que permite iterar sobre una deliberación ya cerrada sin rehacerla desde cero.

## 1. El problema que resuelve

Una deliberación cerrada produce un `outcome.md` congelado. Hoy, cualquier pregunta posterior —aclarar algo confuso, meter un dato nuevo, profundizar en un punto— solo tiene dos salidas: una deliberación nueva completa (panel nuevo, ~25 spawns, re-investigación web) o nada. No hay punto intermedio, y eso es un desperdicio cuando el seguimiento es pequeño.

Lo caro de una deliberación son tres cosas separables: **(a)** diseñar el panel, **(b)** la investigación web de los expertos, **(c)** las 4 rondas × N expertos. Una deliberación nueva tira las tres aunque la pregunta sea minúscula.

## 2. El modelo rector: tres tiers, coste proporcional al cambio

| Tier | Qué es | Coste |
|---|---|---|
| **1 · Aclaración** | La respuesta ya está en el material del run padre, solo mal expuesta | ~1 spawn (moderador) |
| **2 · Refinamiento** | Sub-pregunta real o dato nuevo; el panel existente es el correcto | ~6-12 spawns (panel reusado, ciclo comprimido) |
| **3 · Nuevo ángulo** | Necesita otra expertise o cambia el problema | Deliberación completa (`/council`, ya existe) |

El feature = Tier 1 + Tier 2. El Tier 3 ya es el `deliberate` actual.

## 3. Decisiones P1–P4 (ya confirmadas)

- **P1 · Identidad.** Acción `refine`, wrapper `/council-refine <branch> <problem-id> [<parent-run-id>] [<child-slug>]`. Padre y slug opcionales en CLI (el lead los pide si faltan). Precondición: el run padre existe y está completo (`outcome.md` presente). Comando agnóstico al tier.
- **P2 · Clasificación.** Cascada de 3 tests: ¿responde el moderador solo releyendo? → Tier 1. Si no, ¿panel competente y `problem.md` válido? → Tier 2. Si no → Tier 3 (fuera de `refine`). El lead propone, el usuario confirma. Ante duda, redondear arriba ofreciendo la opción barata. Upgrade 1→2 dentro de `refine`; 2→3 sale a `/council`. Sin downgrades.
- **P3 · Captura.** Artefacto nuevo `follow_up.md` (disparador + info nueva + ancla + inclinación opcional), capturado por mini-iterate sin schema formal; sustituye a `hypothesis.md`. Entregable: ninguno en Tier 1; `deliverable.md` propio (reusando el mecanismo existente) en Tier 2, capturado tras clasificar.
- **P4 · Contexto nuevo.** `refine` nunca escribe `problem.md`. La info nueva vive en `follow_up.md` (hogar operativo). Regla de precedencia: en conflicto, `follow_up.md` > `problem.md`, con overrides marcados explícitamente. Al cierre, el lead recomienda llevar el dato duradero a `problem.md` vía `iterate` (decide el usuario).

---

## 4. Decisiones P5–P14 (confirmadas)

### P5 · Metadatos por-run y enlace `parent_run`

**El conflicto.** Hoy los runs (`council/<run-id>/`) son carpetas sin metadatos; `meta.yaml` es por-problema. El enlace padre-hijo, el tier y el disparador no tienen dónde vivir, y no hay forma de distinguir un run-hermano (deliberación independiente) de un run-hijo (refinamiento).

**Recomendación.** Un archivo nuevo **`run_meta.yaml` dentro de cada carpeta de run** (para `deliberate` y para `refine`):
```yaml
run_id: 2026-05-22-bifacial
kind: deliberation | refinement
parent_run: 2026-05-22-comparativa-paneles   # ausente si kind: deliberation
tier: 2                                       # solo si kind: refinement
trigger: "<el follow-up en una línea>"
created_at: <ISO 8601>
run_status: in_progress | complete
```

**Por qué.** Los metadatos de un run pertenecen al run, no al `meta.yaml` del problema (separación limpia: el problema describe el problema; cada run se autodescribe). `kind` + `parent_run` distinguen hermano de hijo. Un índice por-problema sería cómodo para navegar, pero es **derivable** escaneando los `run_meta.yaml` — no es núcleo, queda como opcional. Coste: `deliberate` también pasa a escribir `run_meta.yaml` (retrofit pequeño).

### P6 · Ciclo de `status` del problema

**El conflicto.** El enum `status` es `draft | open | deliberated`. `deliberate` aborta si `status != open`, y al terminar lo deja en `deliberated` — lo que bloquea una segunda deliberación o un refinamiento (lo vivimos: hubo que forzar `open` a mano).

**Recomendación.** Redefinir `status` como **el ciclo de captura del problema, nada más**: `draft | open`. **Eliminar `deliberated`.** Saber si un problema se ha deliberado/refinado se deriva de los runs (`run_meta.yaml`). `deliberate` y `refine` requieren `status: open`.

**Por qué.** `deliberated` mezclaba dos cosas: "la captura está completa" y "ya se deliberó". La primera es estado del problema; la segunda es un hecho sobre los runs (P5). Separarlas elimina la trampa terminal y es el modelo correcto. Migración: `problem.schema.yaml` v0.2 → v0.3; los `meta.yaml` existentes con `deliberated` migran a `open` (cambio de una línea). *Alternativa más barata pero más turbia:* mantener `deliberated` y que ambas acciones acepten `open` o `deliberated` — no recomendada, conserva la conflación.

### P7 · Reúso del panel — heredar, analizar la relevancia y podar

**El conflicto.** El refinamiento hereda el panel del padre. ¿Mecánica? ¿Se hereda tal cual?

**Recomendación.** El panel del padre es el **punto de partida, no un dato fijo**. Un refinamiento es más estrecho que su deliberación padre — es normal que algún experto sobre para este follow-up concreto. Mecánica:
1. El lead lee el `panel.md` del padre (el roster heredado).
2. **Analiza la relevancia** de cada experto heredado frente al follow-up CONCRETO.
3. **Propone el panel del refinamiento** — el subconjunto relevante (mantiene ≥2-3 para que la crítica cruzada de B′ tenga sentido); puede añadir como máximo un experto nuevo si el follow-up abre un ángulo menor no cubierto.
4. **Checkpoint SIEMPRE**: presenta el panel propuesto al user con la razón de cada poda; el user confirma o ajusta.
5. Tras confirmar: copia al run hijo solo las `persona.md` de los expertos confirmados; escribe el `panel.md` del hijo con la lista del refinamiento y una nota de qué se heredó / podó y por qué.

Un rediseño mayor (el panel heredado casi no sirve) significa que era Tier 3 — P2 debería haberlo filtrado.

**Por qué.** El argumento original ("el panel ya lo aprobó el user, sáltate el checkpoint") es débil cuando el ALCANCE se ha estrechado: el user aprobó el panel para la pregunta amplia; el refinamiento es una sub-pregunta. Re-confirmar un panel podado para el alcance estrecho es más barato (menos expertos = menos spawns) y más enfocado. El checkpoint pasa de "solo si hay ajuste" a "siempre". Copiar solo las personas confirmadas (en vez de referenciar) mantiene cada run autocontenido y auditable.

### P8 · Estructura de rondas comprimida

**El conflicto.** ¿Qué rondas corre un Tier 2? ¿Cómo funciona el "debate condicional"? ¿Sobreviven los escalados barrier-de-ronda?

**Recomendación.** Rondas **adaptativas**:
- **A′ Propuestas** (siempre) — cada experto aborda el follow-up, anclado en el contexto del padre.
- **B′ Críticas** (siempre) — la verificación cruzada es la razón de ser de un council; no se salta.
- **C′ Debate** (condicional) — solo si B′ revela conflicto real. Misma mecánica de shuttle que `deliberate`.
- **D′ Posiciones finales** (condicional) — solo si C′ ocurrió.
- **Síntesis** (siempre).

Mínimo 2 rondas + síntesis; hasta 4 si emerge conflicto. La detección de conflicto = la misma lógica del STEP 5.a de `deliberate`, con una rama "cero conflictos → directo a síntesis". El usuario se consulta antes de C′ (como en 5.b) si hay conflicto. Los escalados barrier-de-ronda se mantienen sin cambios.

**Por qué.** Las 4 rondas existen para surface y resolver desacuerdo. Si un follow-up estrecho no genera desacuerdo, las rondas C′/D′ no aportan nada. Coste proporcional a lo contencioso que resulte. *(Nota: este "debate condicional" probablemente debería backportarse al `deliberate` principal — fuera de alcance aquí.)*

### P9 · Qué contexto del padre leen los expertos

**El conflicto.** El ahorro del Tier 2 viene de que los expertos no re-investigan: leen el trabajo verificado del padre. Hay que definir exactamente qué archivos, sin romper anti-leak ni "`problem.md` como fuente única".

**Recomendación.** Los expertos del hijo leen:
- *Base:* `problem.md`, y del run hijo `follow_up.md` + `deliverable.md` + su `persona.md`.
- *Heredado del padre, siempre:* `outcome.md`, `final_positions/`, `escalations/`, `user_directives.md`.
- *Heredado del padre, condicional:* `debate_summary.md` — si el follow-up toca un punto ya debatido.
- *NO heredado:* `proposals/`, `critiques/`, `debate/` crudos del padre — son material intermedio, parte ya corregido/superado; `final_positions/` y `outcome.md` ya llevan la versión corregida.

El prompt instruye: *los datos verificados del padre son el punto de partida; no re-investigues lo ya verificado, solo lo que el follow-up añade*. Si el follow-up es un **reto** al outcome (no una extensión), el prompt enmarca el outcome del padre como "lo que está en cuestión", no como asentado.

**Por qué.** `final_positions` + `outcome` son el material verificado y ya depurado de errores; las rondas intermedias crudas reintroducirían errores ya corregidos. La regla "`problem.md` fuente única" se **extiende explícitamente** para refinamientos: el contexto es `problem.md` + la salida verificada del padre + el follow-up. No es leak — el leak es una regla de diseño de panel; esto es continuación deliberada.

### P10 · Síntesis y `outcome.md` del hijo

**El conflicto.** ¿Quién escribe el `outcome.md` acotado del hijo, con qué prompt? ¿El hijo produce `debate_summary.md`?

**Recomendación.** Lo escribe un **moderador** (Task agent fresh, mismo rol que el STEP 7 de `deliberate`) con un prompt **acotado**: responde al follow-up según el `deliverable.md` del hijo, abre con la línea de encuadre/linaje (*"Esto refina la sección X de `<run padre>`, motivado por…"*), y **no re-vuelca** el outcome del padre — es autocontenido para el alcance del follow-up. Lee los archivos de ronda del hijo + el `outcome.md` del padre (para saber qué refina y no duplicarlo). `debate_summary.md` se produce **solo si corrió C′**; si no, basta el `outcome.md`. El lead escribe `lead_notes.md` como puente (igual que en `deliberate`) cuando hubo rondas.

**Por qué.** La imparcialidad de la síntesis se mantiene delegándola a un moderador fresh, como en `deliberate`. El prompt acotado es lo que hace el documento legible (P-#2): el usuario abre un documento y tiene respuesta completa a lo que preguntó, sin re-leer lo que no cambió. Artefactos proporcionales: si no hubo debate, no hay `debate_summary` que escribir.

### P11 · El apéndice/puntero en el padre

**El conflicto.** El padre debe apuntar al refinamiento sin que su análisis se altere. ¿Quién lo escribe? ¿Sección o archivo aparte? Choca con "el lead no edita archivos del moderador".

**Recomendación.** Al final del `outcome.md` del padre se **añade una sección de navegación** `## Refinamientos posteriores`, **append-only, sin tocar el contenido existente**. Una entrada por run hijo: fecha, run-id del hijo, tier, el follow-up en una línea, qué sección refina, y la ruta al `outcome.md` del hijo. Lo escribe el **lead** (es el orquestador con visibilidad de ambos runs; el moderador del hijo solo conoce el hijo). Sin breadcrumb inline a mitad del documento — respeta el "sin tocar el original" que pediste. Se escribe como último paso de `refine`.

**Por qué.** Un lector del `outcome.md` del padre **tiene que ver** que existen refinamientos, o actúa sobre información superada; un archivo aparte se perdería. Un apéndice append-only al final lo ve cualquiera que abra el archivo y no altera ni una línea del análisis. Requiere una **excepción estrecha y documentada** a la regla "el lead no edita archivos del moderador": el lead PUEDE añadir un apéndice puramente navegacional, nunca editorial.

### P12 · El flujo del Tier 1

**El conflicto.** La aclaración pura es muy distinta (moderador-solo, sin panel). ¿Su sub-flujo? ¿Dónde aterriza su output?

**Recomendación.** Tier 1 = un **run hijo mínimo**. Flujo: capturar `follow_up.md` → clasificar → spawnear el moderador (lee `outcome.md` + `debate_summary.md` + `final_positions/` del padre + el `follow_up.md`) → el moderador escribe el `outcome.md` del run hijo (una aclaración corta y acotada) → apéndice en el padre (P11). La carpeta del run hijo es mínima: `follow_up.md`, `run_meta.yaml`, `outcome.md` — sin `experts/`, sin rondas, sin `debate_summary.md`. El moderador lleva la escotilla de upgrade de P2: si no puede responder con fidelidad desde los archivos existentes, lo reporta en vez de inventar.

**Por qué.** Tratar el Tier 1 como un run hijo (aunque mínimo) hace que P5 (`run_meta`), P11 (apéndice) y P13 (cadenas) funcionen **igual** sin importar el tier — sin un caso especial estructural. La ligereza del Tier 1 está en el *proceso* (1 spawn, sin rondas), no en saltarse el modelo de datos. Nombre de archivo uniforme `outcome.md` en todos los tiers → P10/P11/P13 más simples.

### P13 · Encadenamiento y recursión

**El conflicto.** ¿Refinar un refinamiento? ¿Cómo se comportan las cadenas de `parent_run` y de apéndices? ¿Varios refinamientos hermanos?

**Recomendación.**
- **Refinar un refinamiento: permitido.** `refine` acepta como padre cualquier run completo, sea `kind: deliberation` o `kind: refinement`.
- `parent_run` guarda solo el **padre inmediato**; la ascendencia completa se deriva caminando la cadena.
- El apéndice de cada `outcome.md` lista **solo sus hijos directos**; las cadenas se navegan salto a salto (el outcome raíz no crece sin límite).
- **Hermanos:** varios follow-ups sobre el mismo padre = varios runs hijos con el mismo `parent_run` y varias entradas en el apéndice del padre. Sin colisión (cada hijo tiene su run-id; P1 aborta si el run-id ya existe).
- Herencia de contexto en cadena (interacción con P9): `refine` camina la ascendencia — los expertos reciben el contexto verificado completo de la deliberación **raíz** + el `outcome.md` acotado de cada refinamiento intermedio.
- Sin tope de profundidad duro, pero una cadena larga es un *smell* que sugiere lanzar un `/council` fresco.

**Por qué.** Guardar solo el padre inmediato + apéndices de hijos directos mantiene el modelo simple y sin crecimiento ilimitado. La profundidad 1 (el caso normal) es trivial; profundidades mayores solo caminan más la cadena.

### P14 · Arquitectura del SKILL: núcleo + archivos de acción

**El conflicto.** Una 4ª acción en un `SKILL.md` único: cada invocación carga el cuerpo entero —las cuatro acciones— aunque se use una. Coste de tokens en cada llamada y dilución de atención (los STEP relevantes compiten con tres procedimientos irrelevantes). Con `refine`, el contenido procedimental casi se duplica.

**Recomendación.** Partir el skill en **núcleo + archivos de acción**, manteniendo **un solo skill con una sola entrada**:

```
.claude/skills/council/
├── SKILL.md            # núcleo, SIEMPRE cargado
└── actions/
    ├── iterate.md
    ├── import.md
    ├── deliberate.md
    └── refine.md
```

- **Núcleo (`SKILL.md`)** — la "constitución" siempre cargada: ROLE, ACTION ROUTING, DATA MODEL, SCHEMA REFERENCE, STYLE & SAFETY, STRUCTURED MARKERS, CAPTURE DISCIPLINE, WRITING DISCIPLINE, INLINE VALIDATION, ESCALADO, WHAT NOT TO DO, REGLAS DEL LEAD, **la mecánica compartida** (template de spawn de rondas, barrier de escalado, síntesis por moderador), y un **STEP 0 de routing**: "según la acción, lee `actions/<acción>.md` y síguelo".
- **Archivos de acción (`actions/*.md`)** — solo los STEP de cada acción; referencian el núcleo, no lo duplican.

No son sub-skills: una sola entrada, una sola `description`. Son archivos de referencia empaquetados, leídos bajo demanda (progressive disclosure). El modelo carga el núcleo siempre + hace **un `Read`** del archivo de la acción que toca.

**Por qué.** Las disciplinas transversales siguen siempre cargadas — no se pierde lo bueno del archivo único. Una invocación de `iterate` deja de arrastrar `deliberate` y `refine`. Honra el *intento* de la regla del proyecto (cero partes móviles, sin hooks/scripts/sub-skills) relajando solo su letra ("un archivo"). Y fuerza el buen refactor: la mecánica que `refine` comparte con `deliberate` vive en el núcleo, referenciada sin duplicar.

**Implica** enmendar la regla del `CLAUDE.md` del proyecto: "todo vive en SKILL.md" → "núcleo `SKILL.md` + `actions/`; sin sub-skills, sin hooks, sin scripts".

**Puntos a tocar:**
- Partir el `SKILL.md` actual en núcleo + `actions/{iterate,import,deliberate}.md` (refactor, sin cambio de comportamiento).
- Nuevo `actions/refine.md`.
- En el núcleo: STEP 0 de routing; `ACTION ROUTING` + `INPUTS` con `refine`; `DATA MODEL` con `run_meta.yaml`, runs hijos, `parent_run`, apéndice en el `outcome.md` padre, y el cambio de schema de P6; excepción del apéndice en `WRITING DISCIPLINE` (P11); don'ts de `refine` en `WHAT NOT TO DO`.
- Nuevo `.claude/commands/council-refine.md`.
- `description` del skill y `CLAUDE.md` del proyecto (regla enmendada, comandos, diagrama de estructura).
- `run_meta.yaml` documentado inline en `DATA MODEL` (sin archivo de schema, como `meta.yaml`). `problem.schema.yaml` → v0.3 (P6). SKILL → **v2.2**.

**Orden de construcción:**
1. **Partir el `SKILL.md` actual** en núcleo + `actions/` (refactor puro, sin tocar comportamiento) y **verificar** que las 3 acciones existentes siguen funcionando.
2. Cambios de modelo de datos: `run_meta.yaml`, cambio de `status` (P6), bump de schema.
3. Escribir `actions/refine.md`.
4. Actualizaciones transversales en el núcleo.
5. Wrapper command + `description` + `CLAUDE.md`.

A verificar antes de implementar: la mecánica exacta de lectura bajo demanda de archivos empaquetados dentro de un skill.

**Por qué este orden.** El paso 1 es un refactor sin riesgo de comportamiento que conviene aislar y verificar solo; a partir de ahí, todo lo demás se apoya en el modelo de datos (paso 2) antes de escribir la acción nueva.

---

## 5. Cierre

Catorce problemas, una decisión confirmada para cada uno. Nada de esto está implementado todavía — este documento es la especificación. El orden de construcción es el de P14: primero partir el `SKILL.md` en núcleo + `actions/` (refactor verificable de forma aislada), luego el modelo de datos, luego `actions/refine.md`, luego las actualizaciones transversales y la documentación.
