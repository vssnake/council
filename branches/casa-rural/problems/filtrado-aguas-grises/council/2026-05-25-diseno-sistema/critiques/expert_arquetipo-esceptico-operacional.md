# Críticas — Escéptico operacional (Ronda B)

---

## Crítica a `proposals/expert_ingeniero-tratamiento-natural.md`

- **Punto fuerte**: el diseño en tres etapas por gravedad sin partes móviles es lo más robusto que puede proponerse desde el punto de vista de abandono parcial. Bien que marque "verificar antes de decidir" en los parámetros clave.

- **Hueco grave — mantenimiento semanal de la cesta de sólidos**: el propio ingeniero escribe "limpieza de la cesta semanal". El user ya respondió que quiere el sistema "lo menos dependiente del mantenimiento posible". Una tarea semanal obligatoria ES la dependencia. ¿Qué ocurre si esa cesta no se limpia en 3-4 semanas? ¿Se tapona y el agua rebosa por algún sitio no previsto? ¿O simplemente se acumula y sigue pasando? Si la respuesta es "se desborda", entonces la cesta semanal es un punto único de fallo operacional, no un detalle menor.

- **Hueco — ausencia total de escenario "vacaciones"**: el user confirmó que habrá períodos de semanas sin ocupación. El sistema deja de recibir agua. ¿Qué pasa con el humedal HFSS sin entrada de agua durante 3-4-6 semanas en julio-agosto? ¿Se seca el sustrato? ¿Muere el biofilm? ¿Hay que "re-arrancar" algo al volver? La propuesta no menciona este escenario ni una sola vez.

- **Supuesto cuestionable — "flujo por gravedad en todas las etapas"**: asume desnivel >0,5 m sin confirmarlo. El user todavía no ha respondido a la pregunta del desnivel. El filtro vertical de pulido (etapa 3) necesita carga intermitente con sifón — ¿eso funciona por gravedad pura o requiere acumulación previa con cierta altura? Si no hay desnivel natural, el sistema entero cambia de concepto (bombeo = electricidad = dependencia = fallo si se va la luz).

- **Supuesto cuestionable — "sifón de descarga intermitente" en el filtro de pulido**: ¿quién ajusta el sifón si deja de funcionar? ¿Es un mecanismo que puede obstruirse? ¿Con qué frecuencia se revisa? No se menciona.

- **Señal de fallo ausente**: ¿cómo sabe el usuario que el filtro de pulido se está colmatando? "Rastrillar cada 3-6 meses" implica una acción preventiva. Pero si no hay señal visible de que ya toca, no se hará. ¿Hay encharcamiento superficial como indicador? No se describe.

---

## Crítica a `proposals/expert_biologo-fitodepuracion.md`

- **Punto fuerte**: el policultivo de tres especies con justificación de resiliencia es un argumento sólido frente al monocultivo. La dimensión estacional está bien abordada desde la biología.

- **Hueco crítico — dimensionado contradictorio con el ingeniero**: el biólogo propone 8-12 m² (4-6 m²/persona) citando García & Corzo 2008. El ingeniero propone 5-6 m² (2-3 m²/persona) citando UN-Habitat 2008. Es un factor ×2 de diferencia. ¿Cuál es? Esto no es un detalle: es el doble de excavación, el doble de grava, el doble de lámina EPDM, el doble de plantación. Si dos especialistas no se ponen de acuerdo en el dato más básico del dimensionado, ¿qué confianza puede tener el user en que el sistema funcionará?

- **Hueco — el sedimentador de 500-1000 L con 24-48h de retención**: esto es un tanque que acumula lodos. ¿Cada cuánto se vacían los lodos? El biólogo dice "vaciado de lodos del sedimentador cada 6-12 meses" en sus supuestos. ¿Cómo se vacía un tanque de lodos enterrado de 500-1000 L? ¿A mano con un cubo? ¿Con una bomba? ¿Dónde se depositan esos lodos? Esto es trabajo pesado y sucio. ¿El user sabe que esto es parte del paquete?

- **Supuesto cuestionable — "el pretratamiento se mantendrá"**: lo marca como supuesto, pero es más que eso. Es la condición de supervivencia del humedal. Si el pretratamiento no se mantiene, el humedal se colmata en 3-5 años (dato del propio biólogo). Pero "se mantendrá" no es un supuesto — es una esperanza. ¿Qué pasa si no se mantiene? ¿Cuál es la señal de alarma? ¿Hay forma de diseñar un pretratamiento que NO requiera atención y siga funcionando degradadamente?

- **Hueco — escenario de ausencia prolongada en verano**: con evapotranspiración que reduce volumen 30-50% y sin entrada de agua durante semanas (vacaciones), ¿qué pasa con las macrófitas? ¿Mueren? ¿Se secan? ¿Hay que replantar al volver? La propuesta no aborda este escenario aunque el user lo confirmó explícitamente.

- **Control de Arundo donax**: el biólogo advierte de esta invasora. Bien. Pero ¿cómo se controla en la práctica? ¿Revisión mensual? ¿Trimestral? ¿Qué implica eliminarla una vez que ha colonizado — requiere excavación del rizoma? Esto es mantenimiento pesado no cuantificado.

---

## Crítica a `proposals/expert_agronomo-riego.md`

- **Punto fuerte**: es la única propuesta que introduce parámetros cuantitativos de calidad objetivo (SS <50 mg/L, Na⁺ <3 meq/L, etc.) y que señala la gestión del sodio en origen como parte del diseño. Esto aterriza el problema.

- **Hueco — ¿quién mide esos parámetros?**: se definen umbrales de calidad, pero ¿cómo sabe el usuario que se están cumpliendo? ¿Compra un kit de análisis? ¿Cada cuánto mide? ¿O es un "confía en que el sistema funciona"? Si no hay forma práctica y barata de verificar que la DBO₅ está por debajo de 25 mg/L, el umbral es decorativo.

- **Hueco — la "especificación de insumos" es una restricción permanente de estilo de vida**: decirle al usuario "cambia tus detergentes" es fácil de escribir en un paper. En la práctica: ¿qué pasa cuando un invitado usa el jabón que trajo de su casa? ¿Cuando se compra "el que estaba de oferta"? ¿Cuando cambia la formulación de una marca? El control de insumos en un hogar no profesional es frágil. ¿Cuánto sodio de más rompe el sistema? ¿Hay margen de error o es un umbral duro?

- **Supuesto cuestionable — "goteo subsuperficial para frutales"**: exige SS <50 mg/L. ¿Qué garantiza que el efluente siempre estará por debajo? ¿Qué pasa el día que la trampa de grasas no se limpió en 2 meses y pasan más sólidos? ¿Se obturan los goteros permanentemente o se pueden lavar? ¿Cuánto cuesta reemplazarlos? Un gotero obturado no es una señal visible — el frutal simplemente se va estresando silenciosamente.

- **Hueco — modelo "uso casi continuo" vs almacenamiento**: la propuesta sugiere que en verano el desfase temporal es menor de lo esperado. Pero el user tiene 200-400 m² de riego y produce ~70-120 L/día de efluente. El riego de 200-400 m² en julio necesita 200-500 L/día (dato del técnico de almacenamiento). Es decir, el agua gris cubre un 15-50% de la demanda estival. ¿De dónde sale el resto? ¿Se riega menos? ¿Se complementa con otra fuente? Esto no se aborda.

---

## Crítica a `proposals/expert_tecnico-almacenamiento-hidrico.md`

- **Punto fuerte**: es la propuesta más realista sobre el problema del almacenamiento prolongado. La afirmación "sin conservación activa, el agua se degrada en ≤10 días" es la más honesta del council.

- **Hueco — "chimenea solar" sin datos de rendimiento**: se propone un tubo negro que genera tiro por convección. ¿Cuánto aire mueve realmente? ¿Es suficiente para mantener aerobio un depósito de 5000-10.000 L? ¿O es un concepto bonito que en la práctica mueve tan poco volumen de aire que el fondo del depósito sigue anóxico? Sin datos de caudal de aire vs. volumen de agua, esto es especulación.

- **Hueco — "entrada sumergida para evitar turbulencia" vs "necesidad de oxigenación"**: la propuesta dice que la entrada debe ser sumergida para no resuspender sedimentos, pero al mismo tiempo necesita oxigenación. Hay una contradicción funcional no resuelta: ¿quieres mover el agua (oxigenar) o no quieres moverla (evitar turbulencia)? El diseño intenta hacer las dos cosas a la vez sin explicar cómo se compatibilizan.

- **Supuesto cuestionable — "asumo que el sistema de filtrado previo logra DBO₅ <30 mg/L"**: este supuesto es la base de todo. Si no se cumple, el almacenamiento falla. Pero el ingeniero y el biólogo no coinciden en qué DBO₅ esperar del efluente (el ingeniero dice <15-20 con pulido+UV, el biólogo dice ~20-30 solo humedal). ¿Y si el pretratamiento se degrada y la DBO sube a 50 mg/L durante unas semanas? ¿El depósito aguanta o se vuelve anaerobio rápido?

- **Hueco — mantenimiento del propio depósito**: un depósito de 5000-10.000 L enterrado ¿acumula lodos en el fondo con el tiempo? ¿Cada cuánto hay que vaciarlo y limpiarlo? ¿Cómo se accede a un tanque enterrado de ese volumen? ¿Con qué frecuencia la bomba solar de recirculación necesita revisión/limpieza?

- **Hueco — rebosadero en invierno**: se propone derivar excedente a "zanja de infiltración o terreno arbolado". ¿El efluente que rebosa tiene calidad suficiente para verter al suelo directamente? ¿O estamos hablando de verter agua semi-tratada cuando el depósito está lleno? ¿Esto crea un charco permanente en invierno con olor?

---

## Posición (sin cambios)

Mi posición no cambia: las cuatro propuestas son técnicamente coherentes por separado, pero ninguna aborda de forma realista:

1. **El escenario de ausencia de varias semanas** (confirmado por el user) — ¿qué pasa con TODO el sistema cuando nadie está?
2. **La cadena de dependencias de mantenimiento** — el pretratamiento condiciona el humedal, el humedal condiciona el almacenamiento, el almacenamiento condiciona el riego. Si falla el eslabón más débil (limpieza de la trampa de grasas, que es el más aburrido y sucio), ¿cómo se propaga el fallo por el sistema?
3. **Las señales de fallo visibles** — ¿cómo sabe el usuario que algo va mal ANTES de que el agua del depósito apeste o los frutales se estresen?

El user dijo "las horas que hagan falta, pero lo menos dependiente posible". Esto NO es "acepto mantenimiento semanal". Es "quiero que si me olvido un mes, no se rompa todo". Ninguna propuesta cuantifica qué margen de negligencia tolera el sistema.

---

## Preguntas al user

- ¿En verano, cuando te vayas de vacaciones (3-6 semanas), hay alguien que pueda hacer un mantenimiento mínimo (ej: vaciar una cesta, comprobar que no hay desbordamiento)?
- ¿Tienes experiencia manejando lodos/residuos orgánicos (compostaje, fosa séptica, etc.) o te resulta algo que prefieres evitar?
- ¿Hay otra fuente de agua para riego (pozo, red municipal) que cubra la demanda en verano cuando el agua gris no sea suficiente?
