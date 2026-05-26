# scada-general

## Contexto

Sistema SCADA para el cliente que consume datos de varios centros de generación renovable:
- varios centros solares fotovoltaicos.
- varios parques eólicos.
- En cada parque hay tres máquinas en las que se puede instalar el sistema de monitoreo.

Arquitectura prevista en dos niveles:
- **En los parques**: sistema de consumo de datos + base de datos temporal.
- **Central**: sobre OpenShift, con los datos ya procesados listos para consumir por el cliente y por terceros.

**Volumetría**: la frecuencia de muestreo varía (desde 1s hasta minutos). Se espera mucho volumen de información, especialmente en la central que agrega todos los parques.

## Objetivo y criterio de éxito

Criterios de éxito:
- Poder obtener datos de las diferentes máquinas que hay en el parque.
- Que los datos estén siendo procesados y parametrizados — tanto métricas temporales como métricas de los dispositivos.
- Que haya un sistema para informar de las alertas del sistema (tanto en planta para operarios como en central con todos los parques).
- En el futuro, un sistema de predicción (ML).

Módulos previstos:
1. **Adquisición de datos multi-protocolo**: módulo único de extracción que se conecta a periféricos vía Modbus, OPC UA, MQTT, OPC DA. Razón de módulo único: bastantes instrumentos solo soportan una conexión, se centraliza.
2. **Procesado y persistencia**: módulo de procesado y guardado en base de datos de series temporales.
3. **Alarmas**: sistema de alertas para métricas de dispositivo — evaluación tanto en planta (local, para operarios) como en central (vista agregada de todos los parques).
4. **Reporter**: dos tipos de reportes (internos y externos), ambos por definir.
5. **Datos en bruto para predicción**: datos disponibles para análisis predictivo futuro.

**Central** (lógica adicional):
- Dashboards propios.
- API para terceros (REST).
- Protocolo industrial IEC 61850 para terceros.
- En el futuro: ML/predicción.

## Restricciones

- **Modalidad**: Time & Materials.
- **Seguridad / red**: los parques no se pueden acceder desde la central. Están dentro de una VPN (no la controlan ellos, la facilitan).
- **Plazo de ejecución**: 2 años, ampliable.
- **Despliegue de infraestructura**: NO está en alcance del equipo (lo hace otro equipo).
- **Tecnología de módulos**: .NET (versión LTS).
- **Despliegue de código**: Azure DevOps pipelines.
- **Infraestructura en parque**: 3 máquinas por parque → Kubernetes pequeño.
- **Conectividad**: ancho de banda desconocido; puede interrumpirse. Requiere mecanismos store-and-forward. Se baraja un sistema de colas para tolerar latencia.
- **Despliegue en central**: OpenShift.
- **Latencia aceptable**: sí puede haber latencia entre planta y central (no es hard real-time).

[SKIP: presupuesto — el usuario prefiere centrarse en la presentación técnica]
[SKIP: normativa concreta aplicable — el usuario no la conoce; la infraestructura no la despliegan ellos]

## Decisiones tomadas y exploraciones previas

**Arquitectura modular en planta** (a desarrollar):
- Módulo único de extracción (centraliza conexiones a instrumentos que solo soportan una).
- Módulo de procesado y guardado en TSDB.
- Sistema de alertas para métricas de dispositivo.
- Reporter (internos + externos).
- Datos en bruto disponibles para análisis predictivo.

**BD de series temporales**: pendiente de decidir. Opciones en evaluación: **InfluxDB** y **TimescaleDB**.

**Stack tecnológico**: .NET para los módulos.

**Comunicación planta ↔ central**: la VPN no la controlan ellos. Hipótesis a validar: **HTTP REST** sería suficiente. Se baraja también un **sistema de colas** para manejar la latencia y las desconexiones.

**Arquitectura de servicios**: se está investigando si hacerlo con microservicios (alternativas por definir: microservicios puros vs. módulos en contenedores vs. híbrido).

**Reporter**: dos tipos de reportes (internos y externos), ambos por definir.

## Decisión concreta que necesitas

- Diseñar / validar una **arquitectura modular** para el SCADA (módulos en planta + central).
- Validar la elección de **base de datos de series temporales** entre **InfluxDB** y **TimescaleDB**.
- Validar la hipótesis de que **HTTP REST** sea suficiente para la comunicación planta ↔ central a través de la VPN (vs. sistema de colas u otra alternativa).
- Recomendar **microservicios vs. otra arquitectura de servicios** para los módulos.
