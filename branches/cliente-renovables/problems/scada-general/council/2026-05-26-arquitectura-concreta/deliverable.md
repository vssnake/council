# Deliverable — cliente-renovables/scada-general · 2026-05-26-arquitectura-concreta

## Tipo de salida

Propuesta arquitectónica concreta: catálogo de servicios + diagrama de flujo de datos + stack tecnológico + interfaces entre componentes. Orientado a entregable para comité técnico del cliente.

## Secciones esperadas en outcome.md

1. `## Catálogo de Servicios en Planta (Edge)`
   - Tabla: nombre del servicio/contenedor, responsabilidad, tecnología principal, dependencias.

2. `## Catálogo de Servicios en Central`
   - Tabla: nombre del servicio/contenedor, responsabilidad, tecnología principal, dependencias.

3. `## Flujo de Datos End-to-End`
   - Descripción narrativa + diagrama ASCII/Mermaid del recorrido de un dato desde el equipo de campo hasta su consumo en central.

4. `## Comunicación Edge ↔ Central`
   - Topología lógica, protocolo, patrones (push, store-and-forward), puertos/endpoints lógicos.

5. `## Stack Tecnológico Completo`
   - Tabla consolidada: capa/función → tecnología concreta (framework, librería, runtime, versión si relevante).

6. `## Interfaces entre Servicios`
   - Para cada par de servicios que se comunican: qué expone, cómo lo consume el otro, protocolo (AMQP, gRPC, REST, en-memoria). Nivel propuesta, no swagger.

## Nivel de detalle

Medio-alto — suficiente para que un responsable técnico del cliente entienda qué se construirá. Nombres concretos de tecnologías, no solo categorías. Pero no un diseño de implementación con schemas o contratos formales.
