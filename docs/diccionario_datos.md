# Diccionario de datos

Referencia rápida de cada columna de los archivos de `01_Datos/datos_originales/`, el instrumento o sistema que la origina y su unidad. El esquema [`esquema_1_proceso_peap.drawio.svg`](esquema_1_proceso_peap.drawio.svg) muestra dónde está cada instrumento en la planta.

## Operacion (`Operacion/Operacion_2026_MM.xlsx`, hoja *Telemetria*)

Un registro por planta y por día, generado por el SCADA al cierre del día (23:59).

| Columna | Tipo esperado | Unidad | Origen | Significado |
| --- | --- | --- | --- | --- |
| Fecha_Hora | Fecha y hora (viene como texto) | dd-mm-aaaa hh:mm | SCADA | Momento del cierre diario |
| Cod_Estacion | Texto | — | SCADA | Código de la planta (PEAP-01 a PEAP-08); clave hacia Estaciones |
| Volumen_m3 | Número entero | m³ | Caudalímetro FIT-01 | Volumen bombeado en el día |
| Energia_kWh | Número entero | kWh | Medidor de energía JT-01 | Energía eléctrica consumida en el día |
| Horas_Operacion | Número decimal | h | Estado de bombas YS-01 | Horas en que al menos una bomba estuvo funcionando |
| Horas_Detencion | Número decimal | h | Estado y alarmas YS-01 | Horas en que la planta no estuvo disponible para bombear |
| Cod_Causa | Texto | — | Alarmas registradas en el SCADA | Causa de la detención; clave hacia Causas. Vacío cuando no hubo detención |
| Presion_Salida_bar | Número decimal | bar | Transmisor de presión PIT-01 | Presión promedio en la línea de impulsión |

## Estaciones (`Estaciones.xlsx`, hoja *Maestro*)

Un registro por planta.

| Columna | Tipo esperado | Unidad | Significado |
| --- | --- | --- | --- |
| Estacion | Texto | — | Código y nombre de la planta en una sola columna (`PEAP-01 \| Los Dominicos`) |
| Sector | Texto | — | Sector de la red: Oriente, Sur o Poniente |
| Comuna | Texto | — | Comuna donde está la planta |
| Tipo_Estacion | Texto | — | **Impulsión**: eleva agua desde la matriz troncal. **Rebombeo**: eleva agua desde un estanque a una cota mayor |
| Capacidad_m3h | Número entero | m³/h | Caudal máximo de diseño |
| Potencia_kW | Número entero | kW | Potencia eléctrica instalada |
| Anio_Puesta_Marcha | Número entero | año | Año en que la planta entró en operación |

## Causas (`Causas_Detencion.txt`, texto delimitado por `|`, UTF-8)

Catálogo de causas de detención.

| Columna | Tipo esperado | Significado |
| --- | --- | --- |
| Codigo | Texto | Código de la causa (`NA` = sin detención) |
| Descripcion | Texto | Descripción de la causa |
| Categoria | Texto | Agrupación: Eléctrica, Mecánica, Hidráulica, Comunicaciones, Programada o Sin detención |

## Glosario

| Término | Significado |
| --- | --- |
| PEAP | Planta Elevadora de Agua Potable: estación que bombea agua a una cota mayor |
| Cota | Altura del terreno o del agua sobre un nivel de referencia |
| RTU | Unidad Remota de Telecontrol: equipo en terreno que lee los instrumentos y los envía al SCADA |
| SCADA | Sistema de supervisión y control que centraliza la telemetría de todas las plantas |
| VFD | Variador de frecuencia: regula la velocidad de los motores de las bombas |
| FIT · PIT · JT · YS | Nomenclatura de instrumentos: caudal (Flow), presión (Pressure), energía (J) y estado (Y); la T final indica transmisor |
| Disponibilidad | Porcentaje del tiempo en que la planta pudo bombear: 1 − horas de detención ÷ horas del período |
| Consumo específico | Energía usada por cada m³ bombeado (kWh/m³); mide la eficiencia energética |
