## Posición
No valido todavía la hipótesis como base operativa sólida. La veo razonable en PowerPoint, pero demasiado optimista para campo real: concentra demasiado riesgo en edge, asume que la degradación de red será gestionable sin demostrarlo y da por hecho que la complejidad de microservicios y Kubernetes en planta no se convertirá en deuda operacional.

## Razones
- **Comunicación planta ↔ central**: el problema ya admite WAN best-effort, cortes y ancho de banda desconocido. En ese contexto, discutir REST vs. colas como si fuera solo una decisión de integración me parece insuficiente. La objeción no es el protocolo; es qué ocurre cuando hay desconexión prolongada, reintentos encadenados, duplicados, desfases temporales y backlog local creciendo sin que nadie pueda entrar desde central.
- **Edge con Kubernetes pequeño**: la hipótesis mete extracción, procesado, persistencia, alertas y reporter en un entorno local donde el equipo no controla la infraestructura y donde el soporte de campo previsiblemente será limitado. Mi duda no es si “cabe”; es quién diagnostica un pod inestable, un nodo degradado, un volumen corrupto o una rotación de certificados fallida cuando el parque está aislado o solo responde a ratos.
- **Módulo único de extracción**: se justifica por la restricción de conexión única de ciertos instrumentos, pero operativamente eso también concentra fallo. Si ese módulo se bloquea, consume recursos de más o queda enganchado con un driver/protocolo problemático, el impacto se propaga a todo el parque. No veo todavía cómo se limita el radio de explosión de ese punto único.
- **TSDB**: la comparativa InfluxDB vs. TimescaleDB puede acabar siendo demasiado académica si antes no se responde qué pasa con disco lleno, retención mal ajustada, corrupción tras apagado brusco, replay de históricos tras reconexión o mezcla de datos tardíos con datos en tiempo casi real. El riesgo operacional está antes que el benchmark.
- **Microservicios como opción principal**: veo red flag clara. En edge industrial, más fronteras de proceso suelen significar más puntos de fallo, más observabilidad que mantener y más despliegues que romper. Si además la comunicación entre servicios todavía está abierta (REST o colas), la hipótesis está apilando incertidumbres una encima de otra.
- **Alarmas y reporter**: la hipótesis los coloca como módulos, pero no aclara su comportamiento degradado. ¿Qué pasa con una alarma generada en planta cuando no hay enlace con central? ¿Se pierde, se duplica, llega tarde, cambia de severidad al recomputarse? ¿Y qué versión de la verdad consume el reporter cuando hay retrasos o reconciliación posterior?
- **OpenShift en central + parques heterogéneos**: la separación planta/central tiene sentido conceptual, pero también puede esconder una asimetría operativa peligrosa: central muy ordenada y edge muy frágil. Si la propuesta técnica brilla en central pero depende de disciplina perfecta en cada parque, el problema real seguirá estando donde menos visibilidad y menos manos hay.

### Red flags
- La hipótesis sigue sin convertir la degradación de conectividad en escenario primario de diseño.
- Se habla de módulos y microservicios antes de cerrar el modelo de fallo y recuperación.
- Se asume operabilidad remota suficiente en parques a los que central no puede acceder directamente.
- No aparece todavía el caso de certificados expirados, relojes desalineados, buffers locales saturados o reinicios no limpios.
- La frontera entre dato bruto, dato procesado, alarma y reporte parece limpia en diseño, pero no está claro cómo se comporta bajo replay y reconciliación.

## Supuestos
- Que la latencia aceptable no se está interpretando erróneamente como tolerancia a pérdida de contexto operativo.
- Que “Kubernetes pequeño” en planta no implica soporte humano local con herramientas y experiencia también pequeñas.
- Que el módulo único de extracción convivirá con dispositivos y drivers con comportamientos no homogéneos y a veces no documentados.
- Que el equipo deberá operar durante degradaciones de red, no solo recuperarse después.
- est. [verificar antes de decidir]: que la propuesta para cliente va a comprometer expectativas de mantenibilidad y soporte, no solo de arquitectura lógica.

## Preguntas al user
- ¿Qué pasa operativamente si un parque queda aislado bastante tiempo y el almacenamiento local empieza a acercarse a saturación?
- ¿Quién resuelve una incidencia rutinaria en planta cuando requiere tocar Kubernetes, certificados o volúmenes persistentes?
- ¿Qué nivel de intervención manual es aceptable por parque y por incidencia antes de que el modelo deje de escalar?
- ¿Cuál es la fuente de verdad cuando central y planta discrepan tras una reconexión con replay de datos?
- ¿Qué impacto de negocio es peor para el cliente: perder datos, duplicarlos, entregarlos tarde o generar alarmas inconsistentes?
- ¿Hasta qué punto se está asumiendo hardware/firmware “razonablemente sano” en dispositivos que quizá no lo esté?