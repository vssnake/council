## Posición
Valido la hipótesis en su dirección general, pero la ajusto en un punto clave: en planta sí recomiendo un concentrador único de adquisición multi-protocolo con normalización temprana; para planta ↔ central no recomiendo depender solo de HTTP REST si el enlace puede cortarse. Mi posición es **store-and-forward con cola persistente como patrón principal**, dejando **HTTP REST** para configuración, consulta puntual y exposición API, no como canal único de ingesta OT.

## Razones
- El `problem.md` encaja muy bien con el patrón de **gateway/concentrador OT**: hay equipos que pueden soportar una sola conexión y además se quiere un módulo único de extracción. Desde comunicaciones industriales, eso no es un detalle de implementación; es una restricción de diseño.
- Desafío parcialmente el anchor en el punto 3: **no conviene mezclar la decisión de planta ↔ central con la comunicación entre microservicios**. Son problemas distintos. Hacia campo prima compatibilidad con protocolos industriales, calidad de dato y tolerancia a cortes; entre servicios prima desacoplamiento funcional y operabilidad.
- Para adquisición, propondría esta jerarquía práctica: **drivers de campo → modelo canónico interno → exposición normalizada aguas arriba**. Si el equipo dispone de OPC UA en origen, úsese como interfaz preferente de normalización; si no, el concentrador debe absorber Modbus/OPC DA/MQTT y publicar un modelo homogéneo.
- **HTTP REST** como canal único de subida desde planta es débil ante cortes y ráfagas de volumen: obliga a reintentos, idempotencia, ordenación y buffering en varios puntos. Se puede hacer, pero termina reimplementando una cola de forma ad hoc.
- Un **bus/cola persistente** entre planta y central encaja mejor con la restricción explícita de ancho de banda desconocido y desconexiones: desacopla el ritmo de adquisición local del ritmo de entrega a central y permite reenvío ordenado cuando vuelve el enlace.
- En OT, el dato sin **timestamp de origen**, **quality flag** y **identidad de activo/señal** pierde mucho valor operativo. Mi recomendación es que el concentrador normalice y propague esos metadatos antes de persistir o reenviar.
- Respecto a IEC 61850 para terceros en central: lo veo razonable como **interfaz norte** si el cliente realmente necesita exponer un modelo eléctrico interoperable hacia afuera; no lo usaría como protocolo interno de adquisición multi-fabricante en planta salvo que los equipos ya nazcan en ese ecosistema.
- Sobre microservicios: desde mi especialidad no veo valor en trocear la **adquisición OT** en muchos servicios pequeños. Ahí prefiero un **servicio/módulo de adquisición bien encapsulado por planta**; si luego el resto de capacidades en central se resuelven con microservicios, perfecto, pero no multipliquemos drivers ni conexiones al campo.

## Supuestos
- Que la mayoría de señales de operación del parque toleran latencias de segundos o minutos y no requieren hard real-time, tal como indica `problem.md`.
- Que la VPN permite tráfico saliente fiable desde planta hacia central, aunque no acceso entrante desde central.
- Que el sistema local en planta puede persistir buffer suficiente durante caídas del enlace y reenviar después sin pérdida semántica.
- est. [verificar antes de decidir]: que no habrá un porcentaje alto de dispositivos legacy que obligue a mantener OPC DA más tiempo del deseable.
- est. [verificar antes de decidir]: que el requisito de terceros sobre IEC 61850 está en la capa de integración central y no exige exposición directa desde cada parque.

## Preguntas al user
- ¿La comunicación planta → central debe salir siempre iniciada desde planta, o existe algún caso real en que central necesite invocar activamente a planta?
- ¿Esperáis conservar en central el **timestamp de origen** y la **calidad del dato** como campos de primer nivel, o solo el valor ya procesado?
- ¿Qué porcentaje aproximado de equipos relevantes habla OPC UA nativo frente a Modbus, OPC DA o interfaces propietarias?
- ¿La necesidad de IEC 61850 para terceros es de lectura near-real-time, intercambio por lotes, o integración contractual con otro SCADA/EMS?