# Panel — cliente-renovables/scada-general · 2026-05-26-propuesta

## Propósito
Evaluar y proponer una arquitectura SCADA distribuida (multinivel planta + central) para un portfolio de generación renovable, incluyendo selección de tecnologías de almacenamiento temporal, comunicaciones industriales, plataforma de microservicios y ciberseguridad.

## Domina
- Diseño de arquitecturas SCADA distribuidas edge/central
- Selección y dimensionado de bases de datos de series temporales
- Estrategia de comunicaciones industriales multi-protocolo
- Patrones cloud-native para entornos OT/IT convergentes

## Fuera de alcance
- Diseño eléctrico de las plantas (inversores, aerogeneradores, BOS)
- Negociación comercial o pricing del proyecto
- Infraestructura física de red (cableado, firewalls hardware — lo despliega otro equipo)
- Machine Learning / modelos predictivos (fase futura)

## Especialistas

- **arquitecto-scada-distribuido** (`role_category: Arquitectura de sistemas SCADA multinivel`) — Diseño de la topología edge/central, flujos de datos, resiliencia ante desconexiones
- **ingeniero-comunicaciones-industriales** (`role_category: Ingeniería de protocolos y comunicaciones OT`) — Protocolos de adquisición, gateways, normalización de datos en campo
- **ingeniero-datos-series-temporales** (`role_category: Ingeniería de datos temporales a escala`) — Selección de TSDB, modelado de datos, retención, store-and-forward
- **ingeniero-plataforma-cloud-native** (`role_category: Plataforma cloud-native en contenedores`) — Descomposición en servicios, orquestación K8s/OpenShift, comunicación inter-servicio
- **especialista-ciberseguridad-industrial** (`role_category: Ciberseguridad OT/IT convergente`) — Segmentación ISA-62443, compliance NIS2, identidad máquina-a-máquina

## Friction archetypes

- **arquetipo-esceptico-operacional** (`dimension: operacional`) — Dudas sobre lo que falla en campo real: conectividad, mantenimiento remoto, edge degradado

## Notas del panel

### Dueño por variable del entregable
- Arquitectura general → arquitecto-scada-distribuido
- Adquisición y comunicaciones → ingeniero-comunicaciones-industriales
- Datos y series temporales → ingeniero-datos-series-temporales
- Microservicios y despliegue → ingeniero-plataforma-cloud-native
- Seguridad y cumplimiento → especialista-ciberseguridad-industrial
- Plan de implantación y riesgos → todos + arquetipo-esceptico-operacional
- Comparativa de tecnologías → cada experto en su dominio

### Dimensiones del panel omitidas (y por qué)
- Se omite `adherencia` porque el equipo desarrolla y entrega el sistema; no se evalúa la sostenibilidad operativa a largo plazo por parte de un operador externo.
- Se omite `politico` porque es una decisión técnica interna sin stakeholders múltiples enfrentados.
- Se omite `economico` porque el proyecto es T&M y el presupuesto no es restricción de diseño en esta fase.

### Decisiones del user sobre composición
Ninguna — panel aceptado en primera propuesta.
