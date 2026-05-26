# Propuesta — agronomo-riego (Ronda A)

## Posición

La hipótesis es válida como marco general, pero necesita un **requisito de calidad de efluente orientado al uso final** que no aparece explícito: el sistema de filtrado debe producir un agua que cumpla umbrales concretos para riego de frutales, hortícolas y ornamentales, y el diseño de almacenamiento debe impedir anaerobiosis. Sin estos dos criterios como entradas de diseño, se corre el riesgo de construir un sistema que "filtra" pero produce un agua que obtura goteros, acumula sodio en el suelo o se vuelve fitotóxica tras semanas en depósito.

Mi aportación al diseño se centra en tres ejes:

1. **Especificación de calidad del efluente** (parámetros objetivo que el sistema de filtrado debe alcanzar).
2. **Gestión del sodio en origen** (detergentes) como parte integral del diseño.
3. **Requisitos del almacenamiento** desde la perspectiva del agua para riego.

---

## Razones

### 1. Parámetros de calidad objetivo para el efluente

Para riego combinado (frutales + huerto + jardín) en clima mediterráneo, el efluente debe alcanzar como mínimo:

| Parámetro | Umbral objetivo | Motivo |
|-----------|----------------|--------|
| Sólidos suspendidos (SS) | < 50 mg/L | Evitar obturación de goteros (fuente: normas de fabricantes de riego localizado) |
| DBO₅ | < 25 mg/L | Evitar anaerobiosis en almacenamiento y proliferación de patógenos |
| Sodio (Na⁺) | < 3 meq/L (~69 mg/L) | Prevenir degradación de estructura del suelo — especialmente si hay fracción arcillosa (fuente: FAO Irrigation & Drainage Paper 29, rev. 1) |
| pH | 6.5–8.5 | Rango tolerable para mayoría de cultivos |
| Cloruro (Cl⁻) | < 140 mg/L | Umbrales de toxicidad foliar en frutales sensibles como cítricos (FAO Paper 29) |
| Boro (B) | < 1 mg/L | Toxicidad en cítricos y frutales de hueso (FAO Paper 29) |
| Coliformes fecales | < 1000 UFC/100 mL | Para riego de hortícolas que se consumen en fresco — aquí la UV puntual aporta valor real |

Estos valores son los que el sistema de biofiltración debe alcanzar. Si no se definen antes de dimensionar, se diseñará "a ciegas".

### 2. El sodio como enemigo silencioso: intervención en origen

Las aguas grises domésticas tienen su principal carga de sodio en **detergentes y jabones**. En un hogar de 2 personas, la concentración de Na⁺ en aguas grises puede oscilar entre 50–200 mg/L dependiendo de los productos usados (est. [verificar antes de decidir] — varía mucho con marca/tipo).

**Recomendación obligatoria como parte del diseño:** el sistema debe incluir una "especificación de insumos" — usar detergentes y jabones bajos en sodio (o basados en potasio). Esto no es un accesorio: es una etapa de control en origen que reduce la carga que el sistema de filtrado debe gestionar. El potasio, a diferencia del sodio, es un nutriente para las plantas.

### 3. Almacenamiento prolongado: el problema real

El `problem.md` menciona almacenamiento de "semanas/meses". Desde la perspectiva de riego, un agua estancada semanas sin oxígeno:

- Genera sulfuro de hidrógeno (H₂S) → fitotóxico para raíces.
- Favorece proliferación de algas y biofilm → obturación de goteros.
- Pierde el oxígeno disuelto → el agua "muerta" inhibe actividad microbiana beneficiosa en el suelo.

**Requisito de diseño para el almacenamiento:**
- Algún mecanismo de **recirculación o aireación** en el depósito (puede ser tan simple como una bomba solar de estanque con burbujeo, ~15–30 W).
- Alternativa: diseñar para **flujo continuo** (sin almacenamiento masivo) usando el agua a medida que se genera, con un depósito pulmón de pocos días, no semanas. En clima mediterráneo con veranos secos, la demanda de riego es alta justamente cuando se genera el agua → el desajuste temporal es menor de lo que parece.

### 4. Método de riego recomendado

- **Goteo subsuperficial** para frutales y huerto: evita contacto foliar (patógenos), reduce evaporación, pero exige SS < 50 mg/L y filtrado fino antes del emisor.
- **Riego por superficie/inundación** para jardín ornamental: tolera peor calidad, menos exigente.
- **NO aspersión** con agua regenerada sobre hortícolas de consumo en fresco — riesgo sanitario directo.

---

## Supuestos

- El suelo en la zona tiene componente calcáreo-arcilloso típico del mediterráneo costero (est. [verificar antes de decidir] — un análisis de suelo cambiaría las recomendaciones de sodio).
- La producción de aguas grises de 2 personas es ~150–200 L/día (fuente: estimaciones estándar de consumo doméstico español, ~70–100 L/persona/día en grises).
- Los frutales mencionados incluyen cítricos u otros sensibles a cloruro/boro (no especificado en `problem.md`).
- No hay riego con otra fuente de agua (red/pozo) que diluya la carga de sales del agua gris.

---

## Preguntas al user

- ¿Qué tipo de suelo hay en la parcela? (arcilloso, arenoso, franco…) — cambia drásticamente el riesgo de acumulación de sodio.
- ¿Qué frutales concretos se riegan? (cítricos, olivos, frutales de hueso…) — los umbrales de toxicidad varían mucho.
- ¿Hay otra fuente de agua disponible para riego (pozo, red municipal) que permita alternar o mezclar?
- ¿Se aceptaría la recomendación de cambiar a detergentes/jabones bajos en sodio como parte del diseño del sistema?
- ¿El almacenamiento prolongado (semanas/meses) es un requisito duro, o se aceptaría un modelo de "uso casi continuo" con depósito pulmón de pocos días + desbordamiento a humedal?
