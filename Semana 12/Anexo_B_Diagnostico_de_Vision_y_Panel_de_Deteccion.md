Anexo B - Reporte de diagnóstico: visión y panel de detección

Origen: documento interno de diagnóstico, entregado al responsable del proyecto.

> Versión con datos omitidos. Se omiten direcciones de red, puertos, identificadores de nodo y de controlador, nombres de archivos, de módulos, de tablas y de servicios, referencias de línea de código y cualquier detalle de cómo aprovechar los hallazgos de seguridad, conforme al acuerdo de confidencialidad.

---

## Resumen ejecutivo

La cadena de detección funciona de punta a punta y sus pruebas pasan, pero tiene **3 fallas críticas**. Alcance: copias de la rama principal del firmware del CSI y del sistema central, revisadas por lectura de código y pruebas locales. **No se probó contra un controlador real.**

| Capa | Estado | Pruebas |
| --- | --- | --- |
| Firmware de visión (CSI) | Flujo completo; ante una cámara caída reporta "desconocido" y nunca "ocupado" | 484 aprobadas, 2 omitidas, 0 fallas |
| Backend del sistema central | Configuración deseada y observada con revisión y hash; cinco defectos conocidos ya corregidos | 1,203 más 118 con base de datos aprobadas; 1 falla y 2 errores por la forma de importar de las pruebas |
| Panel de detección | Es el único panel; ya no existe la ventana anterior | 387 de 387; compilación sin errores |

**Críticos, en orden de atención:**

1. **Credenciales de las cámaras expuestas** por dos caminos: un canal de configuración del CSI sin la protección debida y una consulta del sistema central que las devuelve sin redactar a cualquier usuario con sesión. Además quedan repetidas sin cifrar en tablas.
2. **Presencia que nunca expira en el panel.** Si la conexión en vivo cae, el croquis sigue mostrando "Ocupado" o "Libre" sin aviso. Se suma un congelamiento silencioso si el reloj del CSI retrocede.
3. **Falsos "libre" en el firmware.** La supresión de detecciones solapadas no distingue clases y corre antes del filtro de vehículos, así que una persona en una moto puede suprimir la moto, y solo cuenta el centro de la caja dentro de la zona. El control de luces lo recibe como un "falso" conocido y salta la fase.

**Decisión pendiente del responsable:** el significado de los códigos de dirección está escrito de dos formas opuestas en el código (A6).

## Cómo funciona la visión

Un servicio del CSI con reinicio automático captura cada cámara (flujo secundario, con respaldo al principal), infiere con un modelo de detección en CPU (de 1 a 5 cuadros por segundo por cámara) y convierte la caja detectada en presencia si su centro cae dentro de una zona. La presencia se activa a los 0.3 s y se apaga a los 0.7 s, y cada detector es la unión de sus zonas. Pasa a "desconocido" de inmediato si caen todas sus cámaras, o a los 3 s si cae cualquiera. El estado se envía al control de luces hasta 4 veces por segundo, y este marca la visión desconectada a los 2 s. En campo, cada inferencia toma de 231 a 240 ms, con unos 2.95 cuadros por segundo reales y la CPU cerca del 89 %.

**Modo de fallo.** Una cámara caída produce "falso" más "desconocido", nunca "verdadero". En cambio, un "falso" **conocido** el control de luces lo toma como "sin demanda", por eso los falsos negativos con la cámara sana son el riesgo operativo principal.

El sistema central guarda una configuración **deseada** y una **observada** por nodo y controlador, y decide la sincronía por hash de contenido. Para enviar, resuelve la operación previa, reserva la operación con control de revisión y, si hay tiempo agotado, lee la configuración del CSI y compara por hash. Una operación ambigua bloquea las escrituras. El panel combina la configuración confirmada con un borrador, la conexión en vivo con el CSI y un catálogo de demanda por detector.

## Hallazgos priorizados

Son 23: 3 críticos, 4 altos, 13 medios y 3 bajos. Cada uno se verificó por lectura a mano, por script o por el análisis de la capa.

| ID | Sev. | Hallazgo |
| --- | --- | --- |
| A1 | Crítica | Credenciales de las cámaras expuestas por dos caminos y repetidas sin cifrar en tablas |
| A2 | Crítica | Presencia que nunca expira: si visión se cuelga o la conexión cae, el croquis sigue en "Ocupado" o "Libre" y la imagen dice "en vivo" sobre un cuadro congelado |
| A3 | Crítica | Falsos "libre" con cámara sana (persona más moto, solo el centro de la caja, recorte de zona). Impacto en campo sin medir |
| A4 | Alta | Las rutas de escritura solo exigen sesión: un rol de operador podría enviar configuración a un controlador de campo |
| A5 | Alta | Tormenta de reconexión: cerca de 1 intento por segundo por pestaña, la misma clase del incidente anterior de 32 mil |
| A6 | Alta | Significado contradictorio de la dirección (tráfico que se aproxima desde esa dirección o que circula hacia ella). El responsable confirmó "calle de llegada", y una zona de apoyo norte o sur puede quedar invertida |
| A7 | Alta | El relé empareja respuestas por tipo y no por identificador de petición; un envío aplicado puede quedar fallido o ambiguo |
| A8 | Media | Si el reloj del CSI retrocede, se descarta el estado de cámara más viejo y la presencia queda congelada en silencio |
| A9 | Media | El latido reenvía el último estado sin revisar su edad: si el bucle se bloquea, el control de luces no ve a visión caída |
| A10 | Media | Tras un cambio de configuración o recuperación de cámara se publican unos 0.5 s de ausencia con un vehículo presente |
| A11 | Media | El croquis usa solo la primera zona primaria y puede decir "Libre" mientras el control de luces ve el detector activo |
| A12 | Media | Los errores de carga de configuración o borrador se tragan; editar guarda un borrador construido desde vacío |
| A13 | Media | Una herramienta cruda de sincronía manda "visión fuera de línea" por defecto y sin confirmación |
| A14 | Media | El prototipo dice "nada se guarda" pero guarda asignaciones y borradores reales |
| A15 | Media | Cámaras de otras marcas infieren sobre el flujo principal; si excede 1080p, el detector queda en desconocido permanente |
| A16 | Media | Los puntos fuera de rango se recortan sin aviso; un polígono degenerado hace que el firmware rechace toda la configuración |
| A17 | Media | Dos clientes pueden pisarse la configuración sin aviso, y la escritura a disco bloquea el ciclo de eventos |
| A18 | Media | Si cae al flujo principal y falla al abrir, el reintento del secundario nunca vuelve |
| A19 | Media | Con una operación ambigua, ningún detector es seleccionable a los 600 s hasta un nuevo envío |
| A20 | Media | El envío no maneja varios códigos de conflicto ni ofrece recargar; cerrar el modal pierde el resultado |
| A21 | Baja | La interfaz muestra un umbral nocturno de 0.45 y el firmware usa 0.35 |
| A22 | Baja | Error 500 en lugar de 4xx en tres casos de entrada inválida |
| A23 | Baja | Archivos de respaldo y código antiguo versionados, un archivo de cámaras con credenciales en claro y los 8 códigos de dirección repetidos en 3 lugares |

**Sospechas a vigilar en campo (no confirmadas):** una cascada de reinicios de inferencia con la CPU al 89 %, el bucle principal cerca de 200 Hz con envíos grandes, y una configuración inválida al arrancar combinada con variables de entorno que sobrescriben el archivo compartido.

## Pruebas y defectos conocidos

Ninguna suite tiene fallas atribuibles al código: las vistas fueron dependencias faltantes o configuración de las pruebas. Los defectos conocidos del inventario de cámaras, el cambio de identificadores al renombrar, la reconciliación sin consulta previa y el cierre de la conexión en vivo ya están resueltos, y el interruptor del rediseño se retiró. Sigue abierto un hook de zonas sin recarga compartida.

## Continuidad

- Atender los hallazgos por prioridad, empezando por los tres críticos.
- Decisión del responsable sobre el significado de los códigos de dirección (A6).
- Siguen abiertos para cerrar la integración visión y luces de septiembre: cámara de laboratorio autenticada, detector con zona real, prueba física del modo seguro y revisión del PR 24. El diagnóstico no encontró evidencia de que se hayan cerrado.
- Confirmar en campo las sospechas y medir el impacto real de los falsos "libre".
