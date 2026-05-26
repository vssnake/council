# Críticas — Técnico de almacenamiento hídrico (Ronda B)

---

## Crítica a `proposals/expert_ingeniero-tratamiento-natural.md`

- **Punto fuerte**: el diseño de tres etapas (pretratamiento → humedal → pulido) es coherente con la necesidad de entregar un efluente con DBO₅ < 20 mg/L al depósito. Esto es exactamente lo que yo necesito como entrada para dimensionar un almacenamiento estable. La mención de ventilación pasiva y depósito opaco es correcta.

- **Hueco 1 — almacenamiento tratado como caja negra**: la propuesta dice "depósito cerrado, opaco" y "ventilación pasiva" pero no desarrolla nada más. No hay dimensionado del depósito, no hay criterio de volumen (¿cuántos días de autonomía?), no hay rebosadero, no hay gestión del excedente invernal. El almacenamiento NO es un accesorio al final del diagrama — es la mitad del problema según el `problem.md`.

- **Hueco 2 — flujo por gravedad hasta el depósito**: el filtro vertical de pulido funciona por carga intermitente (sifón), pero la salida del pulido al depósito dice "gravedad o bombeo mínimo". Si el depósito está enterrado o semi-enterrado (que es lo recomendable para estabilidad térmica), la cota de salida del filtro de pulido debe estar por encima del nivel máximo del depósito. Nadie ha verificado el desnivel disponible — y el user no ha respondido aún a esa pregunta. Este es un punto de diseño que puede obligar a bombeo, contradeciendo la premisa de "sin electricidad en operación normal".

- **Hueco 3 — escenario vacaciones (semanas sin entrada)**: el user confirmó que puede haber semanas sin ocupación. ¿Qué pasa con el agua ya almacenada en el depósito durante ese periodo sin consumo de riego y sin nueva entrada que genere circulación? Con DBO residual de 15-20 mg/L, unas 3-4 semanas sin movimiento en depósito cerrado = anoxia = olores. La UV pre-almacenamiento ayuda pero no elimina la materia orgánica — solo mata bacterias. La DBO residual sigue siendo sustrato para recolonización.

- **Punto débil menor**: la estimación de 150-250 L/día es razonable, pero para dimensionar almacenamiento necesito cruzar eso con la demanda de riego estacional. En invierno, si se producen 200 L/día y se riegan 0 L/día, el depósito se llena en semanas. No hay mención de rebosadero ni derivación del excedente.

---

## Crítica a `proposals/expert_biologo-fitodepuracion.md`

- **Punto fuerte**: el dimensionado de 8-12 m² para rendimiento invernal es más conservador y realista que los 5-6 m² del ingeniero. Para mi trabajo, un humedal sobredimensionado que entregue DBO más baja es mejor — reduce la exigencia sobre el almacenamiento.

- **Punto fuerte 2**: la mención explícita de que el efluente "no es estéril" y que se vuelve anóxico en días si se almacena sin oxigenación es exactamente correcta. Es el único experto que lo dice con esa claridad.

- **Hueco 1 — solución de almacenamiento con plantas flotantes (Lemna/Eichhornia)**: esto es un red flag directo de mi dominio. Un estanque abierto con plantas flotantes en clima mediterráneo = mosquitos + eutrofización descontrolada + evaporación masiva en verano. No es una solución de almacenamiento — es un segundo humedal que requiere gestión activa. Si el objetivo es almacenar semanas/meses, necesitamos depósito cerrado, no balsa abierta.

- **Hueco 2 — dimensionado de almacenamiento ausente**: igual que el ingeniero, se centra en el tratamiento y deja el almacenamiento como una nota al margen ("depósito cerrado tipo IBC/cuba" o "estanque/balsa abierta"). Un IBC de 1000 L se llena en 4-5 días con la producción estimada. ¿Cuántos IBCs? ¿Qué volumen total? ¿Cómo se conectan? ¿Dónde va el excedente? Nada de esto está desarrollado.

- **Hueco 3 — evapotranspiración reduce efluente 30-50% en verano**: esto es relevante para mi dimensionado. Si en verano solo llegan 100-150 L/día al depósito (en vez de 200-250), pero la demanda de riego para 200-400 m² es de 400-800 L/día (est. en verano mediterráneo para riego mixto), el depósito se vacía rápido. El sistema NO es autosuficiente para riego en verano — necesita complemento de otra fuente. Esto debería estar explícito.

---

## Crítica a `proposals/expert_agronomo-riego.md`

- **Punto fuerte**: la especificación de parámetros de calidad objetivo (SS < 50 mg/L, DBO₅ < 25 mg/L) como entrada de diseño es muy útil. Me da un criterio concreto para evaluar cuánto tiempo puede aguantar el agua almacenada sin degradarse.

- **Punto fuerte 2**: la propuesta de "flujo continuo con depósito pulmón de pocos días" es, desde mi dominio, la solución más robusta operativamente. Un depósito pulmón de 3-5 días (600-1000 L) con aireación pasiva NO genera olores, es fácil de dimensionar y no necesita UV. El problema es que contradice el requisito del user de "almacenar semanas/meses". Pero merece discusión.

- **Hueco 1 — la bomba solar de aireación como solución mágica**: propone "bomba solar de estanque con burbujeo, ~15-30 W" para el almacenamiento largo. En la práctica: una bomba solar de 15-30 W genera burbujeo durante las horas de sol (6-10 h en invierno, 10-14 h en verano). De noche, la aireación cesa. Con DBO residual de 20-25 mg/L, unas 10-12 horas sin oxigenación pueden no ser suficientes para mantener aerobiosis continua en un depósito grande (>2000 L). Se necesita evaluar si la aireación intermitente es suficiente o si hace falta batería/recirculación nocturna.

- **Hueco 2 — el sodio es relevante pero no afecta al almacenamiento**: toda la sección de sodio y detergentes es pertinente para riego pero irrelevante para la conservación del agua almacenada. El sodio no causa degradación en depósito. Me parece bien como requisito del sistema global, pero no cambia el diseño del almacenamiento.

- **Hueco 3 — no cuantifica el balance hídrico estacional**: dice que "el desajuste temporal es menor de lo que parece" en verano, pero no lo cuantifica. Producción verano: ~100-150 L/día (post-evapotranspiración del humedal). Demanda riego 200-400 m² en verano: est. 2-4 L/m²/día para riego mixto mediterráneo → 400-1600 L/día. El déficit es enorme. El depósito NO puede cubrir la demanda estival solo con aguas grises — se necesita fuente complementaria o aceptar que solo se riega una fracción de la superficie.

---

## Crítica a `proposals/expert_arquetipo-esceptico-operacional.md`

- **Punto fuerte**: el cuestionamiento de la compatibilidad entre "almacenamiento prolongado" y "tratamiento exclusivamente biológico" es exactamente la tensión central del problema. Y la pregunta "¿qué pasa en vacaciones?" está directamente en mi dominio: agua parada en depósito sin circulación ni consumo = degradación asegurada.

- **Punto fuerte 2**: la observación de que "cuando huele ya está degradado" es correcta. Una señal temprana sería medir ORP (potencial redox) o simplemente tener una válvula de muestreo en el depósito para oler antes de que el volumen completo se degrade.

- **Hueco 1 — no propone solución, solo problemas**: esto es legítimo desde su rol de escéptico, pero deja un vacío: después de cuestionar todo, ¿cuál es la alternativa? ¿Reducir el almacenamiento a días? ¿Aceptar un sistema híbrido con más tecnología? ¿Sobredimensionar todo? La ausencia de contrapropuesta dificulta que el council converja.

- **Hueco 2 — sobreestima el fallo del humedal en verano**: dice que "la evapotranspiración podría superar el input" — con 200-250 L/día de entrada y un humedal de 8-12 m², la evapotranspiración en esta zona (est. 5-8 mm/día en julio) consume ~40-96 L/día del humedal. Reduce la salida pero no la elimina. El humedal no se "seca" con ese caudal de entrada si está bien dimensionado. Es un riesgo real pero no catastrófico.

- **Discrepancia parcial**: el escéptico plantea que la UV "no es puntual sino el componente crítico". Desde mi dominio, parcialmente de acuerdo: sin UV, el almacenamiento largo (>2-3 semanas) de un efluente con DBO residual 15-20 mg/L requiere aireación activa obligatoria. Con UV + aireación pasiva, puede aguantar 4-6 semanas. La UV reduce la carga bacteriana que consume oxígeno en el depósito — no es la solución completa, pero sí reduce significativamente la velocidad de degradación.

---

## Posición ajustada

Mi posición de la Ronda A no cambia sustancialmente, pero la refuerzo con estas observaciones cruzadas:

1. **El almacenamiento debe diseñarse para dos escenarios**: (a) verano con uso continuo → depósito pulmón 3-5 días, sin problemas; (b) invierno/vacaciones con acumulación → depósito mayor (2000-3000 L) con aireación solar + rebosadero obligatorio hacia zanja de infiltración o retorno al humedal.

2. **El rebosadero es el elemento que nadie menciona** y es absolutamente obligatorio. En invierno: ~200 L/día de producción × 90 días sin riego significativo = 18.000 L de agua que tiene que ir a algún sitio. Un depósito razonable (2000-3000 L) se desborda en 10-15 días. El excedente debe derivarse a zanja de infiltración, riego de fondo de parcela, o retorno al humedal como recirculación.

3. **Depósito PEAD agrícola (confirmado como aceptable por el user)**, opaco, semi-enterrado para estabilidad térmica. Capacidad: 2000-3000 L (10-15 días de autonomía). Aireación con bomba solar (confirmada como aceptable). Ventilación pasiva mediante tubo PVC con sombrero chino en la parte superior.

4. **UV pre-almacenamiento es recomendable** pero no suficiente sola — debe combinarse con aireación. La UV reduce carga bacteriana pero no elimina DBO residual. Sin aireación, incluso un efluente desinfectado se vuelve anóxico si tiene DBO > 10 mg/L.

---

## Preguntas al user

- ¿Hay desnivel entre la zona donde iría el sistema de tratamiento y la zona de riego? (pendiente de respuesta de ronda anterior — sigue siendo crítica para definir si el depósito va enterrado, semi-enterrado o en superficie, y si se necesita bombeo).
- ¿Dispones de otra fuente de agua para riego en verano (pozo, red municipal)? Si la respuesta es no, el sistema de aguas grises no será autosuficiente para regar 200-400 m² en julio-agosto.
- ¿Aceptarías una zanja de infiltración o zona de dispersión para el excedente invernal, o prefieres recircular al humedal?
