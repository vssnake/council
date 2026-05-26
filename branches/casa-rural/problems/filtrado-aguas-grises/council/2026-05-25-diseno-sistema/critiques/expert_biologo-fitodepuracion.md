# Críticas — Biólogo de fitodepuración (Ronda B)

---

## Crítica a `proposals/expert_ingeniero-tratamiento-natural.md`

- **Punto fuerte**: la arquitectura de tres etapas en serie (pretratamiento → humedal HFSS-H → filtro de pulido) es sólida y coherente con la literatura. La inclusión del filtro vertical de pulido como tercera etapa es un acierto real para el objetivo de almacenamiento prolongado — reduce DBO residual por debajo de lo que el humedal solo puede garantizar en invierno.

- **Hueco 1 — Subdimensionado del humedal**: propone 5-6 m² totales (3 m²/persona). Este dimensionado asume rendimiento de verano o clima templado constante. En esta zona, con temperaturas del sustrato de 8-12°C en invierno, la actividad biológica cae un 60-70%. La referencia que yo manejo (García & Corzo, 2008, UPC) indica 4-6 m²/persona para HFSS con aguas grises en clima mediterráneo, precisamente por la corrección invernal. Con 5-6 m² totales para 2 personas se arriesga a un efluente deficiente de diciembre a febrero. Mi recomendación: **8-12 m² mínimo**, que es coherente con dimensionar para el peor caso estacional.

- **Hueco 2 — Selección de especies tratada como accesoria**: menciona "Phragmites australis, Typha latifolia u otras macrófitas locales" entre paréntesis y luego dice que está "fuera de su limitación". Esto es peligroso: la plantación no es un complemento estético del humedal, ES el motor biológico. La elección de especies, su distribución espacial en el lecho y la densidad de plantación determinan directamente la eficiencia de tratamiento. No puede quedar como un afterthought.

- **Hueco 3 — Granulometría del sustrato**: propone grava inferior 20-40 mm y superior 6-12 mm. La capa de 6-12 mm como medio principal es demasiado fina para aguas grises con carga de grasas y fibras. Colmatación prematura probable en 3-5 años (Nivala et al., 2012). Recomiendo 8-16 mm como capa principal (70% del volumen) y reservar granulometría fina solo para la zona de salida.

- **Supuesto cuestionable**: asume TRH de 3-5 días como suficiente. En invierno, con actividad biológica reducida, se necesitan 5-7 días para alcanzar la misma reducción de DBO. Esto refuerza la necesidad de más superficie (mayor volumen de sustrato = mayor TRH a mismo caudal).

- **De acuerdo con**: la inclusión de UV como opción para almacenamiento >2-3 semanas, el diseño de la trampa de grasas con cesta extraíble, y la pendiente de fondo 0.5-1%.

---

## Crítica a `proposals/expert_agronomo-riego.md`

- **Punto fuerte**: la tabla de parámetros de calidad objetivo (SS, DBO₅, Na⁺, pH, Cl⁻, B, coliformes) es exactamente lo que faltaba como criterio de diseño. Tener umbrales concretos permite dimensionar el sistema de tratamiento con un objetivo medible. Especialmente valioso el punto sobre el sodio y la intervención en origen (detergentes).

- **Punto fuerte 2**: la propuesta de "uso casi continuo" con depósito pulmón de pocos días en verano es biológicamente más coherente que almacenamiento de meses. Reduce la exigencia de calidad del efluente y simplifica el sistema.

- **Hueco — Falta de conexión con la biología del humedal**: los parámetros están bien definidos como *salida*, pero no se indica si un humedal HFSS puede alcanzarlos consistentemente. Desde mi experiencia: DBO₅ < 25 mg/L es alcanzable en verano (efluente típico 15-20 mg/L con buen dimensionado) pero en invierno puede quedar en 30-40 mg/L. SS < 50 mg/L es fácilmente alcanzable con humedal subsuperficial. En cuanto al sodio: el humedal NO reduce sodio significativamente — la absorción por plantas es marginal frente a la carga de entrada. La intervención en origen (cambio de detergentes) es la única solución real, como correctamente identifica.

- **Supuesto cuestionable**: la producción estimada de 150-200 L/día (70-100 L/persona en grises) me parece baja si se incluyen todas las fuentes (fregadero, lavadora, lavavajillas, duchas, lavabos). Con lavadora incluida, 200-300 L/día es más realista para dos personas activas. Esto afecta al dimensionado de todo el sistema.

- **Pregunta derivada**: propone goteo subsuperficial con SS < 50 mg/L. Hay que validar que el sistema de filtrado + pulido pueda mantener ese umbral de forma consistente. Si no se cumple, los goteros se obturan y el user abandonará el sistema de riego por goteo — se necesita un filtro de malla antes de los emisores como seguro adicional.

---

## Crítica a `proposals/expert_tecnico-almacenamiento-hidrico.md`

- **Punto fuerte**: el análisis del desfase estacional producción-consumo es excelente y cuantificado. El cálculo de 6.300-10.800 L de acumulación invernal es realista. El rebosadero obligatorio es imprescindible y muchos diseños lo olvidan.

- **Punto fuerte 2**: la propuesta de "chimenea solar" para aireación pasiva sin electricidad es elegante y coherente con la filosofía del user.

- **Hueco biológico — Pérdida en humedal subestimada en verano**: estima pérdida por evaporación + absorción del 20-30%. En un humedal mediterráneo en julio-agosto, con Phragmites + Typha en plena actividad vegetativa, la evapotranspiración puede consumir el 40-60% del caudal de entrada (Headley et al., 2012; datos propios de sistemas similares en litoral mediterráneo). Con entrada de 150-250 L/día en verano, el efluente disponible puede ser solo 60-100 L/día, no 70-120 L. Para 200-400 m² de riego en verano esto es claramente insuficiente como fuente única.

- **Supuesto cuestionable — DBO₅ <30 mg/L de entrada al depósito**: asume que el filtrado previo logra esto. En invierno, como ya indiqué, un humedal puede producir 30-40 mg/L de DBO₅. Sin la etapa de pulido (filtro vertical) que propone el ingeniero, el supuesto se rompe. El diseño de almacenamiento DEPENDE de que haya una tercera etapa de tratamiento — esto debería ser un requisito explícito, no un supuesto.

- **Entrada sumergida — discrepancia**: propone entrada sumergida "para evitar oxigenación excesiva (turbulencia → resuspensión sedimentos)". Entiendo el razonamiento hidráulico, pero desde la biología, si el agua entra sin oxigenar a un depósito cerrado con DBO residual, la anoxia se establece más rápido. Prefiero una entrada con caída o venturi que airee parcialmente el efluente al entrar — complementario a la ventilación pasiva.

---

## Crítica a `proposals/expert_arquetipo-esceptico-operacional.md`

- **Punto fuerte — Pregunta sobre el operador**: la cuestión de quién mantiene y con qué habilidades es fundamental. La respuesta del user ("las que hagan falta, pero hacerlo lo menos dependiente posible") valida el enfoque de sobredimensionar para reducir frecuencia de intervención.

- **Punto fuerte 2 — Escenario "usuario ausente 4-8 semanas"**: la respuesta del user confirma que hay vacaciones de semanas. Esto es un escenario de diseño real: el humedal sin caudal de entrada durante 2-4 semanas en verano. Mi aportación: un humedal HFSS bien establecido (>2 años) sobrevive sin caudal 4-6 semanas en verano gracias a la reserva de humedad en el sustrato y la resiliencia de las raíces de Phragmites (entran en estado de estrés pero no mueren). Typha es más vulnerable a la desecación prolongada. **Diseño defensivo**: incluir un rebosadero alto en el humedal que retenga un nivel mínimo de agua en el sustrato incluso sin entrada.

- **Hueco — "La UV no es puntual, es el componente crítico"**: esta afirmación es excesiva. La UV es crítica solo para el almacenamiento prolongado (>2-3 semanas). Si se adopta el modelo de uso casi continuo con depósito pulmón (como sugiere el agrónomo), la UV pasa a ser genuinamente opcional. El escéptico plantea un falso dilema: o UV permanente o el sistema falla. La realidad es que hay un gradiente de soluciones según el tiempo de almacenamiento requerido.

- **Supuesto cuestionable — "La evapotranspiración puede vaciar un humedal"**: es un riesgo real pero manejable. Con 200-300 L/día de entrada y un humedal de 8-12 m² × 0.5 m profundidad, hay ~1600-2400 L de volumen poroso. Incluso con evapotranspiración de 10-15 mm/día (extremo para agosto), se necesitarían semanas sin entrada para que el nivel baje críticamente. El problema real no es que se vacíe, sino que el efluente disponible para almacenamiento se reduce drásticamente.

- **De acuerdo con**: la señal temprana de fallo no puede ser "huele". Propongo como indicadores: encharcamiento superficial visible (colmatación), amarillamiento prematuro de macrófitas fuera de ciclo estacional, nivel de agua visible sobre el sustrato. Son señales visuales simples que un usuario no técnico puede interpretar.

---

## Posición ajustada

Tras leer las propuestas, mantengo mi posición base con tres ajustes:

1. **Acepto la tercera etapa de pulido** (filtro vertical de arena) propuesta por el ingeniero como necesaria para el objetivo de almacenamiento. Solo con el humedal, el efluente invernal no garantiza DBO₅ <25 mg/L.

2. **Incorporo el modelo de "flujo casi continuo"** del agrónomo como escenario preferente para verano (depósito pulmón de pocos días + riego directo), y reservo el almacenamiento prolongado solo para el desfase invernal, cuando la demanda de riego es baja o nula.

3. **Mantengo firmemente el dimensionado de 8-12 m²** para el humedal frente a los 5-6 m² del ingeniero. Esto es no negociable desde la biología: dimensionar para verano es un error que se paga cada invierno.

---

## Preguntas al user

- ¿Qué detergentes y productos de limpieza se usan actualmente? (lejía, productos con cloro activo, antical agresivo) — estos pueden matar el biofilm del humedal.
- ¿Se acepta un humedal de 8-12 m² de superficie? Para referencia visual: es aproximadamente el tamaño de un parking de dos coches, o una habitación grande.
- ¿Hay alguna fuente de agua complementaria para riego en verano (pozo, red municipal)? — el efluente del humedal no cubrirá los 200-400 m² de riego en los meses pico.
