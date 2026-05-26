## Posición final
Tras la mediación, sostengo una arquitectura de datos temporales con **RabbitMQ como buffer de sincronización separado** y **TimescaleDB como TSDB preferente única** en planta y central, con la siguiente acotación: esta unificación debe quedar **condicionada a una validación operativa explícita en edge** (footprint, compactaciones, operabilidad y rendimiento sostenido en el clúster de 3 nodos). Si esa validación falla, deberá reabrirse la decisión para planta.

Además, mantengo tres exigencias de diseño que no considero opcionales: **orden por partición/señal durante replay**, **deduplicación determinista extremo a extremo**, y **política de retención/downsampling definida desde el día 1**. El hecho de que el user quiera enviar toda la telemetría a central no elimina la necesidad de agregar por horizontes temporales; solo desplaza esa necesidad a la estrategia de conservación histórica.

## Resumen del recorrido
- Posición inicial (Ronda A): prioricé que la decisión crítica no era solo la TSDB, sino el trípode buffer duradero + replay correcto + retención multinivel.
- Crítica recibida (Ronda B): se me empujó a aterrizar mejor la convivencia entre una única TSDB para edge/central y el papel real del broker frente al replay masivo.
- Mediación (Ronda C): cedí en converger a **TimescaleDB como elección preferente única** y en separar claramente **TSDB local** de **cola de sincronización**; mantuve como condición firme la PoC en edge y la disciplina de modelo temporal (orden, deduplicación, cardinalidad y retención).

## Lo que aporto al moderador
- Punto de acuerdo con el panel: en planta no convienen microservicios finos; el máximo razonable son **tres unidades desplegables**: adquisición/procesado, TimescaleDB y RabbitMQ, exportando hacia central un **canónico técnico** con timestamps de origen, quality flags, IDs estables y unidades normalizadas.
- Punto de disenso firme (si lo hay): no doy por cerrada la elección única de TimescaleDB por conveniencia de ecosistema; solo la respaldo si supera validación realista de operación edge con backlog, replay y retención local de días sin WAN.
