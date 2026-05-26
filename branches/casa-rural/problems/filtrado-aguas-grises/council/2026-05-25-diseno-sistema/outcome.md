# Outcome del council — casa-rural/filtrado-aguas-grises · 2026-05-25-diseno-sistema

---

## Arquitectura del sistema

Sistema de tres etapas en serie, por gravedad (aprovechando el bancal con ~1 m de desnivel disponible), sin partes móviles en el tratamiento biológico:

```
Desagüe aguas grises (casa)
        │  (gravedad)
        ▼
┌───────────────────────────────┐
│  1. PRETRATAMIENTO            │
│  Desgrasador (60-80 L)       │
│  + Decantador (500 L)        │
└───────────┬───────────────────┘
            │  (gravedad)
            ▼
┌───────────────────────────────────┐
│  2. HUMEDAL HFSS-H               │
│  Flujo subsuperficial horizontal  │
│  Grava 8-16 mm + policultivo      │
│  Superficie: 6-10 m² [ver nota]   │
└───────────┬───────────────────────┘
            │  (gravedad)
            ▼
┌───────────────────────────────────┐
│  3. DEPÓSITO PEAD 1.000 L        │
│  Entrada por cascada (cámara      │
│  ventilada + malla)              │
│  Aireación solar (bomba 5-10 W)  │
│  Rebosadero → zanja infiltración  │
└───────────┬───────────────────────┘
            │
            ▼
┌───────────────────────────────────┐
│  FILTRO MALLA 120-150 mesh        │
│  (antes de emisores de riego)     │
└───────────┬───────────────────────┘
            ▼
         RIEGO
   (goteo subsuperficial frutales/huerto
    + superficie para jardín ornamental)
```

**Nota sobre la tercera etapa**: la propuesta original del ingeniero incluía un filtro vertical de pulido (arena fina) entre el humedal y el depósito. Esta etapa refuerza la calidad del efluente (especialmente en invierno) y es recomendable, pero el panel no alcanzó consenso sobre si es imprescindible o si un humedal bien dimensionado (8-10 m²) la hace innecesaria. Si se opta por 5-6 m² de humedal, el filtro de pulido pasa a ser necesario. Si se opta por 8-10 m², puede omitirse inicialmente y añadirse más adelante si la calidad del efluente no es satisfactoria.

---

## Etapas de filtrado

### Etapa 1: Pretratamiento

| Elemento | Función | Especificación |
|----------|---------|----------------|
| Desgrasador | Retener grasas/aceites de cocina y detergentes flotantes | Arqueta de obra o bidón PE de 60-80 L con deflectores. Retención hidráulica: 15-20 min. |
| Decantador primario | Sedimentación de sólidos (fibras lavadora, restos orgánicos), flotación de grasas residuales | Tanque de 500 L con deflectores, relación largo/ancho ≥ 3:1. Retención hidráulica: est. 12-24 h `[verificar antes de decidir]`. |

**Red flags operacionales**: sin pretratamiento funcional, el humedal se colmata en 3-5 años. El pretratamiento es la condición de supervivencia del sistema completo.

### Etapa 2: Humedal construido HFSS-H

| Parámetro | Valor / Rango |
|-----------|---------------|
| Tipo de flujo | Subsuperficial horizontal |
| Medio filtrante principal (70% vol.) | Grava lavada 8-16 mm |
| Zona de salida (20% vol.) | Grava 6-12 mm |
| Base drenante (10 cm) | Grava gruesa 20-40 mm |
| Profundidad útil del lecho | 0,5-0,6 m |
| Superficie | **6-10 m²** (ver desacuerdo abierto abajo) |
| Pendiente de fondo | 1% |
| Impermeabilización | Lámina EPDM o PE de 1 mm sobre geotextil 200-300 g/m² |
| Distribución de entrada | Tubería perforada a lo ancho del lecho (evita caminos preferenciales) |
| Rebosadero alto | Mantiene nivel mínimo de 10-15 cm en sustrato durante ausencias |

**Desacuerdo abierto sobre superficie**:
- **Posición A (ingeniería + agronomía): 5-6 m²** — suficiente para aguas grises con filtro de pulido posterior; minimiza pérdida por evapotranspiración.
- **Posición B (biología; almacenamiento favorece): 8-10 m²** — necesario para rendimiento invernal con caída del 60-70% de actividad microbiana; prioriza calidad de efluente.
- **Resolución recomendada**: medir caudal real de aguas grises durante 1-2 semanas antes de construir. Si >200 L/día → dimensionar hacia 8-10 m². Si <150 L/día → 5-6 m² pueden ser suficientes con filtro de pulido.

**Plantación (policultivo mediterráneo obligatorio)**:

| Zona del lecho | Especies | Función |
|---|---|---|
| Entrada (primeros 2-3 m²) | *Phragmites australis* (carrizo) | Tolerante a carga alta, rizomas profundos (hasta 60 cm), máxima superficie de biofilm |
| Media (3-4 m²) | *Phragmites australis* + *Scirpus holoschoenus* (junco churrero) | Diversidad = resiliencia ante plagas y estacionalidad |
| Salida (2-3 m²) | *Iris pseudacorus* + *Carex riparia* | Pulido final, raíces finas que atrapan sólidos residuales |

Densidad de plantación inicial: 6-8 rizomas/m². Cobertura completa esperada en 12-18 meses. Todas las especies son nativas de la zona del litoral mediterráneo.

### Etapa 3 (opcional/condicional): Filtro vertical de pulido

| Parámetro | Valor / Rango |
|-----------|---------------|
| Medio | Arena silícea 0,5-2 mm (capa principal 60-80 cm) sobre grava drenante 20-40 mm (15-20 cm) |
| Superficie | est. 1-1,5 m² `[verificar antes de decidir]` |
| Modo de carga | Intermitente (sifón de descarga) para mantener aerobiosis |
| Función | Reducir DBO₅ residual y SS por filtración + biofilm aerobio |

**Recomendación**: incluir si el humedal es de 5-6 m². Puede omitirse inicialmente si el humedal es de 8-10 m², y añadirse posteriormente si la calidad del efluente no es satisfactoria.

---

## Materiales

| Material | Uso | Accesibilidad en zona mediterránea |
|----------|-----|-------------------------------|
| Grava lavada 8-16 mm | Medio filtrante principal del humedal | Cantera local — alta disponibilidad |
| Grava lavada 20-40 mm | Base drenante humedal + zona de entrada | Cantera local — alta disponibilidad |
| Arena silícea 0,5-2 mm | Filtro de pulido (si se incluye) | Proveedor de áridos |
| Lámina EPDM o PE 1 mm | Impermeabilización del humedal | Almacenes de construcción / riego |
| Geotextil 200-300 g/m² | Protección de la lámina impermeabilizante | Almacenes de construcción |
| Depósito PEAD agrícola opaco 1.000 L | Almacenamiento del efluente | Suministros agrícolas (tipo depósito negro de finca) |
| Tubería PVC Ø75-110 mm | Conducciones, distribución, rebosaderos, ventilación | Fontanería estándar |
| Bomba solar 5-10 W con difusor | Aireación activa del depósito | Tiendas de estanque / solar |
| Malla acero inox <1 mm | Anti-insectos en cámara de cascada y ventilación | Ferretería industrial |
| Bidones PE 60-80 L y 500 L | Desgrasador y decantador del pretratamiento | Reciclaje industrial / agrícola |
| Filtro de malla/disco 120-150 mesh | Protección de emisores de riego | Suministros de riego agrícola |

---

## Dimensionado

| Parámetro | Estimación | Notas |
|-----------|-----------|-------|
| Caudal diario aguas grises (2 personas) | est. 140-250 L/día `[verificar antes de decidir]` | Medir durante 1-2 semanas antes de construir |
| Superficie humedal HFSS-H | 6-10 m² (desacuerdo abierto) | Depende del caudal real medido |
| Profundidad útil humedal | 0,5-0,6 m | |
| Volumen desgrasador | 60-80 L | |
| Volumen decantador | 500 L | |
| Superficie filtro de pulido (si se incluye) | 1-1,5 m² | |
| Volumen depósito | 1.000 L | Confirmado por user |
| Superficie total del sistema (con accesos) | est. 15-25 m² | Según opción de dimensionado |
| Desnivel disponible (bancal) | ~1 m | Suficiente para gravedad en todas las etapas |
| Pérdida por evapotranspiración en verano | est. 40-60% del caudal de entrada | El efluente disponible para riego se reduce a 80-120 L/día en julio-agosto |

**Balance hídrico clave**: las aguas grises producen est. 80-150 L/día de efluente neto para riego (tras pérdidas del humedal). La demanda de riego para 200-400 m² en verano es de est. 140-340 L/día. **Las aguas grises son un complemento** — el user confirma disponer de otra fuente de agua para cubrir el déficit estival.

---

## Almacenamiento

### Diseño del depósito

| Componente | Especificación | Función |
|---|---|---|
| Depósito | PEAD agrícola opaco (negro), 1.000 L, semi-enterrado | Almacenamiento sin algas (oscuridad), estabilidad térmica |
| Entrada | Cascada/caída libre en cámara ventilada con malla inox <1 mm | Oxigenación del efluente (sale del humedal con O₂ a 1-3 mg/L) |
| Ventilación pasiva | Tubo PVC Ø75-110 mm con sombrero chino superior | Intercambio gaseoso permanente — **no negociable** |
| Aireación activa | Bomba solar 5-10 W con difusor en fondo | Mantiene aerobiosis durante horas de sol; inercia de O₂ cubre parcialmente la noche |
| Rebosadero | Tubo Ø75 mm a 5-10 cm del borde superior → zanja de infiltración o dispersión en parcela | Gestión de excedente (especialmente invernal). Se llena en ~5 días sin riego. |
| Toma de riego | Flotador con filtro grueso | Extrae agua de zona media-alta (mejor calidad) |
| Purga de fondo | Válvula en punto más bajo | Vaciado periódico de lodos residuales |

### Estrategia de uso

- **Verano**: flujo cuasi-continuo. El agua se consume en días (demanda > producción). El depósito actúa como pulmón de 3-5 días.
- **Invierno**: acumulación. Sin demanda de riego, el depósito se llena y el excedente se deriva por rebosadero a zanja de infiltración.
- **Vacaciones (3-6 semanas sin ocupación)**: con aireación solar funcionando, el agua ya almacenada se mantiene sin degradación significativa durante 3-4 semanas (DBO < 20 mg/L + aireación intermitente). Si la ausencia supera 4-5 semanas, una purga a la vuelta resuelve sin complejidad.

### Calidad objetivo del efluente a la entrada del depósito

| Parámetro | Umbral | Motivo |
|---|---|---|
| DBO₅ | < 25 mg/L | Evitar anaerobiosis en almacenamiento |
| SS | < 50 mg/L | Evitar sedimentación excesiva y obturación de goteros |
| Na⁺ | < 3 meq/L (~69 mg/L) | Prevenir sodificación del suelo — solo controlable en origen (detergentes) |
| pH | 6,5-8,5 | Rango tolerable para cultivos |

---

## Mantenimiento

### Tabla de mantenimiento mínimo

| Tarea | Frecuencia | Tiempo est. | Consecuencia de no hacerlo |
|---|---|---|---|
| Vaciar/limpiar desgrasador | Cada 2-4 semanas | 5-10 min | Grasas pasan al humedal → colmatación acelerada |
| Vaciar lodos del decantador | Cada 6-12 meses | 30-60 min | Sólidos pasan al humedal → colmatación |
| Inspección visual del humedal (encharcamiento superficial) | Mensual | 2 min | Colmatación no detectada → efluente de mala calidad |
| Poda de macrófitas (parte aérea a 15-20 cm) | Anual (feb-mar) | 1-2 h | Nutrientes se reliberan al agua; acidificación superficial |
| Vigilancia de invasoras (Arundo donax) | Trimestral | 5 min | Colonización que reduce eficiencia depuradora |
| Rastrillar superficie filtro de pulido (si existe) | Cada 3-6 meses | 15-20 min | Colmatación → encharcamiento |
| Verificar bomba solar de aireación | Trimestral | 5 min | Sin aireación → anoxia en depósito → olores |
| Purga de fondo del depósito | Cada 2-3 meses | 5 min | Acumulación de lodos → degradación de calidad |
| Limpieza filtro de malla de riego | Mensual en temporada de riego | 5 min | Obturación de goteros |

### Señales de fallo visibles (protocolo básico)

| Señal | Significado | Acción |
|---|---|---|
| Agua visible sobre el sustrato del humedal | Colmatación parcial | Revisar pretratamiento; considerar rastrillar zona de entrada |
| Amarillamiento de macrófitas fuera de ciclo estacional | Toxicidad en influente o sequía del sustrato | Verificar detergentes utilizados; comprobar nivel de agua |
| Cascada del depósito sin flujo (o flujo muy reducido) | Obstrucción aguas arriba | Revisar desgrasador y decantador |
| Olor sulfhídrico en depósito | Anoxia establecida | Verificar bomba solar; purgar fondo; si persiste, vaciar y recargar |
| Goteros obturados de forma generalizada | SS en efluente por encima del umbral | Revisar filtro de malla; verificar estado del pretratamiento |

### Protocolo de ausencia (>2 semanas)

**Antes de irse:**
1. Limpiar desgrasador y cesta de sólidos.
2. Verificar que la bomba solar funciona.
3. Comprobar que el rebosadero del depósito no está obstruido.

**Qué ocurre durante la ausencia:**
- Sin caudal de entrada, el humedal mantiene humedad residual en el sustrato. Phragmites entra en estrés hídrico a partir de la semana 4 pero no muere (rizomas resistentes). Typha es más vulnerable — el rebosadero alto del humedal mantiene un nivel mínimo que protege las raíces.
- El agua ya almacenada en el depósito se conserva 3-4 semanas con aireación solar intermitente.

**Al volver:**
- Inspección visual: ¿hay encharcamiento en humedal? ¿Olor en depósito?
- Si hay olor en el depósito tras >4 semanas: purgar fondo y dejar que se renueve con nueva producción.
- El sistema se recupera en 2-3 días de uso normal.

### Requisito operacional obligatorio: detergentes bajos en sodio

El sistema de biofiltración NO elimina sodio disuelto (Na⁺). El uso de detergentes convencionales (altos en sodio) producirá un efluente que, aplicado repetidamente al suelo, degrada su estructura en 2-4 años (especialmente en suelos arcillosos).

**Especificación**: usar jabones y detergentes de lavadora/lavavajillas basados en potasio (K⁺) en lugar de sodio. El potasio es un nutriente para las plantas — el efluente se convierte en fertilizante potásico gratuito. Coste adicional estimado: 20-30% sobre detergentes convencionales.

---

## Veredicto sobre la hipótesis del user

La hipótesis validada por el user establece:

> 1. Un sistema de filtrado natural construido de forma artesanal (no prefabricado), con materiales accesibles.
> 2. Enfoque ecológico como base, con posibilidad de incorporar tecnología puntual (ej: UV) si aporta valor para el almacenamiento prolongado sin olores.

### Partes confirmadas por el panel

- **Sistema artesanal natural con materiales accesibles**: confirmado plenamente. Todos los materiales (grava, EPDM, PVC, PEAD) son accesibles en la zona mediterránea sin necesidad de sistemas propietarios.
- **Enfoque ecológico como base**: confirmado. El humedal HFSS-H es el núcleo del sistema y opera sin electricidad ni químicos.
- **Posibilidad de tecnología puntual si aporta valor**: confirmado, pero la tecnología puntual que realmente aporta valor NO es la UV sino la **bomba solar de aireación** (5-10 W). Este es el componente que permite conservar el agua almacenada sin olores.

### Partes desafiadas o matizadas por el panel

1. **"Almacenamiento prolongado (semanas/meses) sin olores"**: el panel desafía la premisa de que se necesite almacenamiento de meses. Con las aguas grises como complemento de riego y un depósito de 1.000 L, el escenario real es flujo cuasi-continuo en verano (consumo en días) y acumulación invernal con rebosadero. El almacenamiento "de meses" no es necesario ni deseable — 3-4 semanas con aireación solar es el máximo práctico para un efluente biológico.

2. **UV como tecnología puntual**: el panel concluye que la UV es genuinamente opcional (no necesaria) si se adoptan aireación solar + depósito pequeño (1.000 L) + flujo cuasi-continuo. La UV solo sería relevante para almacenamiento >3-4 semanas sin aireación activa — un escenario que el diseño consensuado ya evita. La aireación solar sustituye funcionalmente a la UV con mayor efectividad (previene anoxia, que es la causa real de los olores).

3. **Omisión original de la calidad química del efluente**: la hipótesis asume que "filtrado natural" = agua adecuada para riego. El panel (agronomía) demuestra que esto es incompleto: el sistema biológico no elimina sodio ni sales disueltas. Sin control de insumos (detergentes bajos en Na⁺), el agua será biológicamente limpia pero agronómicamente dañina a medio plazo. Esto es un **requisito de diseño de entrada** que la hipótesis no contemplaba.

### Razones derivadas del debate

- La aireación solar fue convergencia unánime porque ataca la causa raíz (consumo de O₂ por DBO residual), mientras que la UV solo reduce carga bacteriana sin eliminar materia orgánica.
- El balance hídrico demuestra que en verano no hay excedente para almacenar (la demanda de riego supera la producción), haciendo innecesario el depósito grande y el almacenamiento de meses.
- El control de sodio en origen es la única intervención posible — ningún sistema biológico natural retira Na⁺ disuelto del agua.

---

## Audit post-síntesis

### 1. Mapeo de secciones

| Sección en `deliverable.md` | Sección en `outcome.md` | Estado |
|---|---|---|
| Arquitectura del sistema | ✅ Arquitectura del sistema | Completa |
| Etapas de filtrado | ✅ Etapas de filtrado | Completa (3 etapas + condicional) |
| Materiales | ✅ Materiales | Completa |
| Dimensionado | ✅ Dimensionado | Completa (con rangos y desacuerdo reflejado) |
| Almacenamiento | ✅ Almacenamiento | Completa |
| Mantenimiento | ✅ Mantenimiento | Completa (con protocolo de ausencia añadido) |

### 2. Disciplina de evidencia — cifras con fuente vs. estimaciones

| Cifra | Fuente / Estado |
|---|---|
| Caída 60-70% actividad microbiana en invierno | García & Corzo 2008, UPC / Kadlec & Wallace 2009 |
| 2-3 m²/persona para HFSS aguas grises | UN-Habitat 2008, Table 5.2 |
| 4-6 m²/persona para HFSS clima mediterráneo | García & Corzo 2008, UPC |
| Colmatación con sustrato <6 mm en 2-5 años | Nivala et al., 2012, *Ecological Engineering* |
| ET de Phragmites/Typha: 8-12 mm/día en verano | est. `[verificar antes de decidir]` — dato del biólogo sin fuente explícita citada |
| Caudal aguas grises 140-250 L/día (2 personas) | est. `[verificar antes de decidir]` — rango cruzado de estimaciones INE/CEDEX; medir antes de construir |
| Demanda riego 200-400 m² en verano: 140-340 L/día | est. `[verificar antes de decidir]` — cálculo del agrónomo basado en 4-6 L/m²/semana |
| O₂ disuelto efluente HFSS: 1-3 mg/L | Dato de ingeniería de tratamiento (sin fuente explícita; consistente con literatura general) |
| DBO₅ efluente HFSS: 15-30 mg/L según estación | García & Corzo 2008 / dato general de la literatura |
| Coste adicional detergentes K⁺: 20-30% | est. `[verificar antes de decidir]` |
