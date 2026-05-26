# Hipótesis tentativa — cliente-renovables/scada-general · 2026-05-26-propuesta

[auto-generated · edited by user]

## Hipótesis

1. **Arquitectura modular en planta**: un módulo único de extracción multi-protocolo + módulo de procesado/TSDB + alertas + reporter + datos bruto para ML.
2. **TSDB**: evaluación entre InfluxDB y TimescaleDB (sin inclinación clara aún).
3. **Comunicación**: elegir entre HTTP REST o sistema de colas — tanto para la comunicación planta ↔ central como para la comunicación entre microservicios.
4. **Arquitectura de servicios**: microservicios como opción principal a validar.
