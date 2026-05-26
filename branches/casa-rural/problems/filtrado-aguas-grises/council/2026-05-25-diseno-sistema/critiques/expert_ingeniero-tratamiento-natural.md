## Crítica a `proposals/expert_biologo-fitodepuracion.md`

- **Punto fuerte**: la selección de HFSS como tipo de humedal es exactamente lo que yo recomendaría — flujo subsuperficial horizontal es la opción más robusta para este contexto (sin mosquitos, sin olores superficiales, autoconstruible). La justificación bibliográfica es sólida (García et al., Vymazal, Kadlec & Wallace).

- **Punto fuerte**: el dimensionado para rendimiento invernal es la heurística correcta. Si funciona en enero, en julio sobra.

- **Hueco: pretratamiento insuficientemente detallado**. Propone trampa de grasas + sedimentador, pero no define la configuración hidráulica del sedimentador (deflectores, relación largo/ancho mínima 3:1 para evitar cortocircuitos). Un bidón de 200 L como trampa de grasas para 250 L/día da un tiempo de retención de ~19 horas si se usa solo para grasas — eso es excesivo para un desgrasador y corto para un decantador. Hay confusión funcional entre los dos elementos. Yo separaría: desgrasador de 60–80 L (retención 15–20 min) + decantador independiente de 500 L con deflectores.

- **Supuesto cuestionable**: asume ~250 L/día para 2 personas. El técnico de almacenamiento estima 100–150 L/día citando INE. La diferencia (100–250 L/día) es un factor ×2.5 que cambia el dimensionado del humedal de 8 m² a potencialmente 5 m². Debería marcarse más claramente como `est. [verificar antes de decidir]` con rango.

- **Problema con la granulometría propuesta**: grava 8–16 mm como capa principal es conservador pero adecuado. Sin embargo, la zona de salida con 4–8 mm me preocupa: si el pretratamiento falla o se retrasa la limpieza, esos finos se colmatarán primero y de forma irreversible. Yo usaría 8–12 mm en la zona de salida.

- **Hueco: pendiente y distribución del influente**. Menciona 1–2% de pendiente, pero no describe el sistema de distribución en la entrada. Un punto único de entrada crea caminos preferenciales y zonas muertas — en un lecho de 8–12 m² esto reduce la superficie efectiva un 30–40%. Necesita una tubería de distribución con múltiples puntos de entrada a lo ancho del lecho.

---

## Crítica a `proposals/expert_agronomo-riego.md`

- **Punto fuerte**: la tabla de parámetros de calidad objetivo es exactamente lo que falta en las demás propuestas. Definir SS <50 mg/L y DBO₅ <25 mg/L como criterio de diseño "de salida" le da al ingeniero de tratamiento un target concreto.

- **Punto fuerte**: la recomendación de detergentes bajos en sodio como "etapa de control en origen" es brillante y práctica. Un humedal NO elimina sodio — no hay proceso biológico que lo retire. La intervención en origen es la única solución real.

- **Hueco**: la propuesta de "flujo continuo con depósito pulmón de pocos días" contradice el escenario de vacaciones. Si no hay consumo de riego porque el user está ausente y el sistema sigue con riego automatizado, vale. Pero si no hay riego durante la ausencia, necesitas almacenamiento o desbordamiento controlado.

- **Supuesto cuestionable**: "150–200 L/día" para 2 personas en grises. La horquilla real para España es 70–130 L/persona/día en aguas grises, según datos del INE y estudios del CEDEX. Para 2 personas: 140–260 L/día. El rango es demasiado amplio para dimensionar sin medición real.

- **Problema: coliformes fecales <1000 UFC/100 mL como crítico**. En aguas grises puras (sin mezcla con negras) los coliformes son generalmente bajos. El problema real son tensioactivos y DBO. Poner coliformes como crítico puede sesgar hacia UV obligatoria cuando quizá no sea necesario.

---

## Crítica a `proposals/expert_tecnico-almacenamiento-hidrico.md`

- **Punto fuerte**: el depósito PEAD enterrado con ventilación pasiva es pragmático. La "chimenea solar" es una solución elegante de coste casi nulo.

- **Punto fuerte**: el rebosadero obligatorio resuelve el problema invernal.

- **Problema serio: entrada sumergida "evita oxigenación excesiva"**. Esto es un error conceptual. El efluente del humedal llega con O₂ disuelto bajo (típico de HFSS: 1–3 mg/L). Lo que NECESITAS al entrar al depósito es maximizar la oxigenación — caída libre, salpicadura, cascada — para saturar el agua y retrasar la anaerobiosis. La entrada sumergida acelera la degradación. Propongo lo contrario: entrada por cascada.

- **Hueco: la aireación pasiva NO escala a volúmenes grandes**. Un tubo DN75 solo oxigena la capa superficial. En un tanque de 10.000 L vertical, el 80% estará en anoxia a partir de la segunda semana. Para >3.000 L necesitas recirculación activa o diseño horizontal.

- **Supuesto cuestionable**: producción de 100–150 L/día citando INE. El dato INE de ~70 L/persona/día incluye TODAS las aguas residuales, no solo grises. La fracción gris es 60–70% del total: ~84–98 L/día para 2 personas. `est. [verificar antes de decidir]`.

---

## Crítica a `proposals/expert_arquetipo-esceptico-operacional.md`

- **Punto fuerte**: la pregunta sobre señales tempranas de fallo es correcta y nadie más la aborda. Desde mi dominio: señal temprana en humedal = encharcamiento superficial; en depósito = turbidez o pH <6.5.

- **Problema: exagera la fragilidad del sistema natural**. Un HFSS bien dimensionado NO requiere intervención semanal. Requiere: limpieza de desgrasador cada 2–4 semanas (5 min), verificación visual mensual, poda anual. El escéptico aplica heurística de usuario urbano a contexto rural.

- **Problema: "la UV no es puntual, es el componente crítico" es parcialmente errónea**. Con aireación activa (bomba solar aceptada), la UV es complemento, no imprescindible. La degradación es por consumo de O₂, no por patógenos. La aireación es el componente crítico del almacenamiento.

---

## Posición ajustada

Mi posición original se mantiene. Incorporo matices:

1. **Del agrónomo**: adopto la tabla de parámetros de calidad como criterio de diseño de salida. Control de sodio en origen es imprescindible.

2. **Del técnico de almacenamiento**: el depósito necesita aireación activa (bomba solar) para >2–3 semanas. Pero la entrada debe ser por cascada, NO sumergida.

3. **Del escéptico**: incorporo protocolo de "modo ausencia" — válvula de bypass al rebosadero al salir de vacaciones. El humedal tolera 4–6 semanas sin flujo si mantiene humedad residual.

## Preguntas al user

- ¿Hay pendiente natural en el terreno entre la salida de aguas grises de la casa y la zona disponible para el sistema?
- ¿El pozo negro existente podría servir como punto de rebosadero/overflow?
