# Críticas — agronomo-riego (Ronda B)

## Crítica a `proposals/expert_ingeniero-tratamiento-natural.md`

- **Punto fuerte**: arquitectura de tres etapas bien razonada, con lógica clara de por qué el filtro de pulido es necesario para alcanzar DBO₅ baja antes del almacenamiento. El diagrama de flujo es claro y el dimensionado 5-6 m² para 2 personas es conservador-adecuado.
- **Punto fuerte**: reconoce que la UV se justifica sólo si almacenamiento > 2-3 semanas. Buen criterio de proporcionalidad.
- **Hueco crítico — calidad del efluente sin parámetros de riego**: el ingeniero define su éxito como "DBO₅ < 15-20 mg/L y SS < 20 mg/L" para almacenamiento sin anaerobiosis. Eso es correcto para el depósito, pero **no menciona los parámetros que importan para el destino final (riego)**: sodio, cloruros, boro, pH. Un efluente puede tener DBO perfecta y estar cargado de Na⁺ de los detergentes, lo cual destruye suelos en 2-3 años. El sistema de tratamiento no elimina sodio — pasa directamente. Esto es un punto ciego del diseño.
- **Hueco — gestión del sodio en origen**: no aparece ninguna mención a la composición de los detergentes como variable de entrada al sistema. Sin esta recomendación, se puede construir un humedal perfecto que produce un efluente estructuralmente inadecuado para riego prolongado.
- **Hueco menor — método de riego**: dice "RIEGO" como destino genérico, sin diferenciar entre goteo (que exige SS <50 y filtrado fino) y superficie (más tolerante). El SS < 20 mg/L que propone sería compatible con goteo, pero no menciona la necesidad de un filtro de malla/disco antes de los emisores — el biofilm puede formarse en el depósito y taparlo igualmente.
- **Pregunta no resuelta**: la estimación de caudal (150-250 L/día) es razonable pero con 200-400 m² de riego (dato del user), en verano la demanda hídrica (~3-5 L/m²/semana en mediterráneo seco) = 85-285 L/día solo de riego. Es decir, en verano el balance producción-consumo podría ser casi neutro o deficitario. Esto cambia radicalmente la necesidad de almacenamiento prolongado: quizá no sea necesario almacenar meses, sino días.

---

## Crítica a `proposals/expert_biologo-fitodepuracion.md`

- **Punto fuerte**: la selección de policultivo (Phragmites + Typha + Iris) con justificación de resiliencia es excelente y bien argumentada. La heurística de diversidad > monocultivo es sólida.
- **Punto fuerte**: dimensionado para rendimiento invernal (peor caso) es la decisión correcta.
- **Problema serio — dimensionado 8-12 m²**: propone 4-6 m² por persona equivalente, duplicando al ingeniero (que usa 2-3 m²/persona). La diferencia es significativa: el biólogo cita García & Corzo 2008 (guía para España) y el ingeniero cita UN-Habitat 2008. Ambas son fuentes legítimas, pero para aguas grises (no aguas negras), 4-6 m²/persona es un dimensionado para aguas residuales completas. Las aguas grises tienen menor carga orgánica que las aguas mixtas. Proponer 8-12 m² puede ser sobredimensionado → mayor evapotranspiración en verano → menos agua disponible para riego. Desde mi perspectiva (la del cultivo que necesita esa agua), sobredimensionar el humedal es contraproducente: pierdo caudal por evaporación.
- **Hueco — pérdida de agua por evapotranspiración**: el biólogo menciona que en verano la evapotranspiración puede reducir el efluente un 30-50%. Si el humedal es de 12 m² con macrófitas de alta ET (Typha es una "bomba de agua"), la pérdida puede ser aún mayor. Con 200 L/día de entrada y 40-50% de pérdida, quedan 100-120 L/día de efluente. Para 200-400 m² de riego en verano, eso es insuficiente. **La elección de especies de alta ET entra en conflicto directo con el objetivo de maximizar agua disponible para riego.**
- **Hueco — sin mención de sodio/sales**: igual que el ingeniero, no aparece el problema del sodio ni la calidad química del efluente para riego. El Typha absorbe N y P (bien), pero no absorbe Na⁺ en cantidades significativas.
- **Conflicto potencial con mi posición**: el biólogo no diferencia entre "agua biológicamente buena" y "agua agronómicamente buena". Un efluente con DBO baja y patógenos controlados puede seguir siendo fitotóxico o sodificante a largo plazo.

---

## Crítica a `proposals/expert_tecnico-almacenamiento-hidrico.md`

- **Punto fuerte**: el análisis del desfase estacional (producción invernal vs. consumo estival) es el más claro de todas las propuestas. El cálculo de 6.300-10.800 L acumulables en invierno está bien razonado.
- **Punto fuerte**: la solución de aireación pasiva con chimenea solar es elegante, barata y coherente con la filosofía del proyecto.
- **Punto fuerte**: el rebosadero obligatorio es un detalle que nadie más menciona y es absolutamente necesario.
- **Problema — dimensionado de demanda de riego subestimado**: estima 200-500 L/día en verano. Con los datos del user (200-400 m² de riego), y usando tasas de riego mediterráneo estándar para frutales+huerto (~4-6 L/m²/semana en goteo eficiente), la demanda real sería: 200 m² × 5 L/m²/semana ÷ 7 = 143 L/día (mínimo) hasta 400 m² × 6 L/m²/semana ÷ 7 = 343 L/día (máximo). El rango 200-500 es correcto por orden de magnitud, pero importa el detalle: si la producción neta de efluente tras humedal es ~100-150 L/día, **en verano no hay excedente para almacenar — al contrario, hay déficit**. El depósito de 5.000-10.000 L se llenaría en invierno pero se vaciaría en pocas semanas de verano.
- **Problema — calidad del agua tras almacenamiento prolongado**: la toma flotante (agua superficial, más oxigenada) es buena idea, pero no resuelve el problema de la calidad química. Agua estancada 3 meses, aunque aireada, concentra sales por evaporación parcial (el depósito tiene ventilación → algo de evaporación). Si el agua ya entra con Na⁺ cercano al umbral, tras evaporación en depósito puede superar 3 meq/L. Falta monitorización de conductividad eléctrica (CE) como indicador de salinidad antes de regar.
- **Hueco menor**: la entrada sumergida "para evitar oxigenación excesiva" contradice parcialmente la necesidad de mantener el depósito aerobio. Habría que balancear: ¿queremos oxigenar o no? Creo que sí queremos, y la entrada debería favorecer aireación, no evitarla.

---

## Crítica a `proposals/expert_arquetipo-esceptico-operacional.md`

- **Punto fuerte**: las preguntas operacionales son las correctas. El escenario "usuario ausente 4-8 semanas" es real (el user confirmó que puede haber vacaciones de semanas) y nadie más lo abordó como riesgo de diseño.
- **Punto fuerte**: la observación de que "cuando huele, ya está degradado" y la pregunta por señales tempranas de fallo es pertinente. Desde mi dominio confirmo: cuando el agua huele a sulfhídrico y se aplica a raíces, el daño ya está hecho.
- **Punto fuerte**: cuestionar si la UV es "puntual" o es "el componente crítico" es una observación aguda y correcta.
- **Problema — no aporta solución**: identifica problemas reales pero no propone alternativas ni criterios de aceptación. ¿Cuál es su propuesta? ¿No hacer el sistema? ¿Hacerlo con almacenamiento corto? ¿Añadir automatización? La crítica sin propuesta deja al consejo sin un camino concreto que evaluar.
- **Hueco — no evalúa el escenario de "no hacer nada"**: actualmente las aguas grises se vierten directamente al suelo sin tratamiento. El riesgo operacional de un sistema imperfecto con mantenimiento irregular sigue siendo mejor que el statu quo (vertido directo). Falta esta comparación como baseline.
- **Desacuerdo parcial**: la afirmación de que "todo sistema que requiere intervención semanal será abandonado en 6 meses" es una heurística útil pero no universal. El user dijo "las horas que hagan falta" — aunque matizó que prefiere minimizar. Un sistema con mantenimiento mensual (no semanal) es viable para alguien con huerto activo (que ya dedica tiempo semanal al riego y cultivo).

---

## Posición ajustada

Mi posición de Ronda A se mantiene en lo esencial, con estos refinamientos basados en los datos del user:

1. **El almacenamiento de meses es probablemente innecesario en verano**: con 200-400 m² de riego y producción de ~100-150 L/día de efluente, en verano se consume todo lo que se produce (o más). El almacenamiento largo es un escenario invernal, cuando no se riega. Esto refuerza mi propuesta de depósito pulmón + rebosadero a humedal/infiltración.

2. **El dimensionado del humedal debe equilibrar depuración vs. pérdida por evapotranspiración**: apoyo al ingeniero (5-6 m²) sobre el biólogo (8-12 m²). Desde la perspectiva del riego, cada m² extra de humedal con Typha es agua que se evapora en verano y no llega al cultivo.

3. **El sodio sigue siendo el punto ciego del consejo**: ninguna propuesta lo aborda como parámetro de diseño. Insisto: sin especificación de detergentes bajos en sodio como requisito del sistema, el agua será nominalmente "tratada" pero agronómicamente inadecuada a largo plazo.

4. **Añado**: para el escenario de vacaciones (2-4 semanas sin ocupación), el humedal no muere — las macrófitas sobreviven con la humedad residual del sustrato. El depósito con aireación solar sigue funcionando. No es un escenario crítico si el sistema está bien dimensionado.

---

## Preguntas al user

- ¿Qué tipo de suelo hay en la parcela? (arcilloso, arenoso, franco…) — sigue sin respuesta y es determinante para el riesgo de sodificación.
- ¿Qué frutales concretos se riegan? (cítricos son muy sensibles a cloruros; olivos son tolerantes) — cambia los umbrales objetivo.
- ¿Se aceptaría como requisito del sistema usar detergentes/jabones bajos en sodio (basados en potasio)? Son algo más caros pero protegen el suelo a largo plazo.
- ¿Hay otra fuente de agua para riego (pozo, red) que complemente las aguas grises en verano cuando la producción no cubra la demanda?
