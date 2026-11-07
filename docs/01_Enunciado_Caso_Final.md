# Caso Final Power BI – Telemetría de Plantas Elevadoras de Agua Potable

Curso Power BI Esencial · Ejemplo base para el trabajo final


## 1. Contexto de negocio

Una empresa sanitaria opera 8 Plantas Elevadoras de Agua Potable (PEAP) en la Región Metropolitana. Cada planta bombea agua desde un estanque de aspiración hacia la red o hacia estanques en cotas más altas. Los datos son ficticios, pero el proceso es real.

Cada planta tiene una RTU (Unidad Remota) que envía al sistema SCADA de telecontrol las variables del proceso. Al cierre de cada día (23:59), el SCADA consolida un registro por planta con:

- **Volumen bombeado** (m³), medido por el caudalímetro de salida.
- **Energía consumida** (kWh), medida por el medidor eléctrico de la planta.
- **Horas de operación** de las bombas y **horas de detención** no disponible, con su **código de causa**.
- **Presión de salida** promedio (bar).

Para entender el proceso antes de analizar los datos, revisa en la carpeta `docs/` los esquemas [de una planta](esquema_1_proceso_peap.drawio.svg) y [de la red de plantas](esquema_2_red_plantas.drawio.svg), y el [diccionario de datos](diccionario_datos.md).

La Gerencia de Operaciones necesita un informe en Power BI para el primer trimestre de 2026 (enero a marzo) que responda tres preguntas:

1. ¿Qué plantas tienen **baja disponibilidad** y por qué se detienen?
2. ¿Qué plantas son **ineficientes energéticamente** (consumen más kWh por cada m³ bombeado)?
3. ¿Cómo evolucionan el volumen, la energía y el costo energético en el trimestre?

Los datos vienen tal como los exporta el SCADA: con filas de título, totales, duplicados, registros sin comunicación y errores de digitación. Parte del trabajo es dejarlos limpios antes de analizarlos.

**Tu rol:** tu equipo actúa como una firma consultora contratada por la Gerencia. No basta con describir los datos: debes diagnosticar el problema, demostrarlo con evidencia, **cuantificar cuánto cuesta** y proponer entre 2 y 3 medidas concretas.

## 2. Archivos fuente

Se entregan 5 archivos en la carpeta `01_Datos/datos_originales/`. Los tres archivos mensuales tienen exactamente la misma estructura y deben cargarse juntos desde la carpeta.

| Archivo | Formato | Contenido | Rol en el modelo |
| --- | --- | --- | --- |
| `Operacion/Operacion_2026_01.xlsx` | Excel, hoja *Telemetria* | Registro diario por planta, enero | Hechos |
| `Operacion/Operacion_2026_02.xlsx` | Excel, hoja *Telemetria* | Registro diario por planta, febrero | Hechos |
| `Operacion/Operacion_2026_03.xlsx` | Excel, hoja *Telemetria* | Registro diario por planta, marzo | Hechos |
| `Estaciones.xlsx` | Excel, hoja *Maestro* | Maestro de las 8 plantas | Dimensión |
| `Causas_Detencion.txt` | Texto delimitado por barra vertical (`\|`), UTF-8 | Catálogo de causas de detención | Dimensión |

**Columnas de Operación:** Fecha_Hora · Cod_Estacion · Volumen_m3 · Energia_kWh · Horas_Operacion · Horas_Detencion · Cod_Causa · Presion_Salida_bar.

**Columnas de Estaciones:** Estacion (código y nombre juntos) · Sector · Comuna · Tipo_Estacion · Capacidad_m3h · Potencia_kW · Anio_Puesta_Marcha.

**Columnas de Causas:** Codigo · Descripcion · Categoria (Eléctrica, Mecánica, Hidráulica, Comunicaciones, Programada, Sin detención).

## 3. Etapas a resolver

El trabajo se resuelve en 5 etapas, en este orden. Cada etapa se apoya en la anterior.

### Etapa 1 · Carga de datos

- [ ] Cargar los 3 archivos mensuales **desde la carpeta** `Operacion` (Obtener datos → Carpeta → Combinar y transformar), no uno por uno.
- [ ] Cargar `Estaciones.xlsx` (hoja *Maestro*).
- [ ] Cargar `Causas_Detencion.txt` como texto/CSV con delimitador `|` y codificación UTF-8.

### Etapa 2 · Transformación en Power Query

Los datos traen errores típicos de una exportación SCADA. Debes resolver, como mínimo:

- [ ] Quitar las filas de título del reporte y usar la fila correcta como encabezado.
- [ ] Eliminar la fila de **TOTAL** que trae cada mes.
- [ ] **Dividir** la columna Fecha_Hora y quedarte solo con la fecha, con tipo Fecha.
- [ ] Normalizar los códigos de estación y de causa (espacios sobrantes, mayúsculas/minúsculas).
- [ ] **Quitar duplicados** (el SCADA re-exportó algunos días).
- [ ] Eliminar los registros **sin comunicación** (volumen vacío) y los **errores de sensor** (volumen negativo).
- [ ] **Reemplazar nulos** de Cod_Causa: un día sin detención debe quedar con el código `NA`.
- [ ] En Estaciones: **dividir** la columna Estacion en código y nombre; corregir Sector y Comuna; quitar la fila vacía y el duplicado.
- [ ] En Estaciones: **combinar** Sector y Comuna en una columna *Ubicacion* (ejemplo: `Sur - Puente Alto`).
- [ ] En Causas: recortar espacios, quitar la línea vacía y el duplicado.
- [ ] Asignar el tipo de dato correcto a todas las columnas y dar nombres claros a las tablas.

**Reproducibilidad:** la limpieza debe poder repetirse paso a paso. Además de los pasos en Power Query, exporta la base depurada a `01_Datos/datos_limpios.xlsx` (con Power Query, con Python o con Excel avanzado; cualquiera de las tres herramientas es válida) y verifica que tenga las mismas filas que la tabla de hechos de Power BI.

### Etapa 3 · Modelo semántico

- [ ] Crear una tabla **Calendario** con DAX (enero a marzo 2026) con columnas Año, Mes, Nombre del mes y Día de la semana.
- [ ] Relacionar las tablas en **esquema estrella**: 1 tabla de hechos (Operacion) y 3 dimensiones (Estaciones, Causas, Calendario), relaciones 1 a muchos.
- [ ] Marcar Calendario como tabla de fechas.

### Etapa 4 · Medidas DAX

- [ ] Crear las medidas de la sección 4 en una tabla de medidas, con formato y nombre claro.

### Etapa 5 · Informe

- [ ] Construir 2 páginas: **Resumen operacional** y **Detenciones y eficiencia**.
- [ ] Incluir tarjetas KPI, al menos 3 tipos de gráficos distintos y segmentadores por mes, sector y estación.
- [ ] Escribir 3 conclusiones de negocio que respondan las preguntas del contexto.

## 4. Indicadores solicitados

Las metas de la Gerencia para el trimestre son: **disponibilidad ≥ 98,5 %** y **consumo específico ≤ 0,40 kWh/m³**. La tarifa eléctrica de referencia es **120 CLP/kWh**.

| Indicador (medida DAX) | Cómo se calcula | Control gráfico sugerido |
| --- | --- | --- |
| Volumen total (m³) | Suma de Volumen_m3 | Tarjeta + gráfico de líneas por día |
| Energía total (kWh) | Suma de Energia_kWh | Tarjeta |
| Consumo específico (kWh/m³) | Energía total ÷ Volumen total | Barras horizontales por estación con línea de meta 0,40 |
| Costo de energía (CLP) | Energía total × 120 | Tarjeta + columnas por mes |
| Disponibilidad (%) | 1 − Horas de detención ÷ (días registrados × 24) | Medidor (gauge) con meta 98,5 % |
| N° de detenciones | Días con Horas_Detencion mayor que 0 | Tarjeta |
| Horas de detención | Suma de Horas_Detencion | Columnas por categoría de causa |
| MTTR (h) | Horas de detención ÷ N° de detenciones | Tarjeta o columna en tabla |
| Volumen promedio diario (m³) | Promedio del volumen por día | Tarjeta |
| Cumple meta disponibilidad | Texto "Cumple" / "No cumple" según la meta | Tabla o matriz con formato condicional |

**Indicadores de impacto** (para la cuantificación del caso):

| Indicador (medida DAX) | Cómo se calcula | Control gráfico sugerido |
| --- | --- | --- |
| Sobrecosto energético (CLP) | Por estación: (energía − 0,40 × volumen) × 120, solo si es positivo | Barras por estación + tarjeta del total |
| Volumen no bombeado (m³) | Por estación: caudal medio real (volumen ÷ horas de operación) × horas de detención | Barras por estación |
| Horas de detención sobre la meta (h) | Por estación: horas de detención − 1,5 % de las horas registradas, solo si es positivo | Tarjeta o columna en matriz |

**Preguntas que el informe debe responder con un número:**

1. ¿Qué estación tiene la menor disponibilidad y cuál es su categoría de causa principal?
2. ¿Qué estaciones superan la meta de 0,40 kWh/m³? ¿Influye el tipo de estación (Impulsión o Rebombeo)?
3. ¿En qué mes hubo más horas de detención?
4. ¿Cuánto costó la energía del trimestre y qué estación aporta el mayor costo?

*Desafío opcional:* medida de variación porcentual del volumen respecto del mes anterior.

## 5. Estructura del caso de consultoría

El informe y el pitch deben seguir esta secuencia, no una descripción de gráficos uno tras otro.

1. **Diagnóstico y formulación del problema:** qué datos se analizaron (8 plantas, 90 días), por qué importa para una sanitaria (continuidad del servicio y costo eléctrico) y cuál es la brecha concreta frente a las metas de 98,5 % y 0,40 kWh/m³.
2. **Evidencia cuantitativa (hallazgos):** qué plantas no cumplen, qué causas explican las detenciones, si hay diferencias por tipo de estación, sector o mes. Cada hallazgo se respalda con una cifra del informe.
3. **Cuantificación del impacto:** cuánto cuesta el problema, usando los indicadores de impacto de la sección 4. Los supuestos deben quedar escritos (tarifa de 120 CLP/kWh; si se anualiza, se supone que el trimestre es representativo).
4. **Propuesta de solución:** entre 2 y 3 medidas concretas (por ejemplo, cambios en mantenimiento, renovación de equipos o alertas automáticas). Cada medida debe apuntar a un hallazgo y estimar cuánto del impacto podría recuperar.

## 6. Entregables

Se entrega un único archivo `Grupo_XX_Solemne2.zip` con esta estructura de 4 carpetas:

| Carpeta | Archivos | Contenido |
| --- | --- | --- |
| `01_Datos/` | `datos_originales/` (los 5 archivos entregados) y `datos_limpios.xlsx` | Datos crudos sin modificar y la base depurada (hojas Operacion, Estaciones, Causas) |
| `02_Programa/` | `script.py`, `notebook.ipynb` o `modelo_analisis.xlsx` | Limpieza y cálculo de indicadores, comentado paso a paso |
| `03_PowerBI/` | `dashboard_consultoria.pbix` | Modelo estrella, medidas DAX y al menos 2 vistas ejecutivas con segmentadores |
| `04_Presentacion/` | `pitch_consultoria.pptx` o `.pdf` | Pitch de 5 minutos, **máximo 7 láminas** |

**Guion sugerido del pitch (7 láminas):**

1. El problema: 8 plantas, metas de disponibilidad y consumo específico.
2. Los datos: fuente SCADA, período y cómo se limpiaron (filas antes y después).
3. Hallazgo 1: plantas que no cumplen disponibilidad y sus causas.
4. Hallazgo 2: plantas ineficientes en energía.
5. Impacto: sobrecosto en CLP, volumen no bombeado y horas sobre la meta.
6. Propuestas: 2 a 3 medidas, cada una con el impacto que ataca.
7. Próximos pasos y cómo se monitoreará con el dashboard.

## 7. Evaluación

Se usan los mismos criterios y pesos de la pauta. La columna de la derecha dice qué mirar en este caso para alcanzar el nivel sobresaliente.

| Criterio | Peso | En este caso, el nivel sobresaliente exige |
| --- | --- | --- |
| Procesamiento de datos | 25 % del entregable | Los 14 errores intencionales corregidos; tabla de hechos con 701 filas; fechas y tipos correctos; resultado verificado contra `datos_limpios.xlsx` |
| Modelado y Power BI | 25 % del entregable | Estrella con 1 hecho y 3 dimensiones; medidas DAX explícitas en tabla de medidas; 2 vistas ejecutivas con segmentadores funcionando |
| Caso de consultoría | 30 % del entregable | Brecha frente a las metas bien definida; impacto en CLP y m³ con supuestos escritos; 2–3 propuestas ligadas a plantas y causas concretas |
| Reproducibilidad y orden | 20 % del entregable | ZIP con las 4 carpetas; script o libro Excel comentado; el PBIX actualiza sin errores desde la carpeta de datos |
| Storytelling y tiempo | 50 % de la defensa | Pitch de 5 minutos y máximo 7 láminas que va de problema a impacto y acción |
| Dominio técnico individual | 50 % de la defensa | Cada integrante explica cualquier paso de Power Query, la relación entre tablas y cualquier medida DAX del informe |

El entregable y la defensa pesan 50 % cada uno.

**Chequeo rápido:** la tabla de hechos limpia debe quedar con **701 filas**. Si tienes otra cifra, revisa los pasos de la Etapa 2.
