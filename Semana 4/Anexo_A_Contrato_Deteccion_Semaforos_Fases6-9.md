Anexo A - Reporte de cierre: contrato de comunicación entre el módulo de detección de vehículos y el módulo de luces (fases 6 a 9)

Fecha del cierre: 11 de septiembre de 2026
Origen: documento interno de cierre de contrato, entregado al responsable del proyecto.

> Versión con datos omitidos. Este anexo reproduce el contenido técnico del reporte interno omitiendo nombres de archivos, rutas internas, identificadores de nodo y detalles de configuración de los equipos, conforme al acuerdo de confidencialidad. Lo que se conserva es el problema, el razonamiento, las decisiones y las verificaciones hechas.

---

## Objetivo

Cerrar las cuatro fases restantes del contrato de comunicación entre el módulo de detección y el módulo de luces (continuación de las fases 1 a 5 cerradas la semana anterior), la regla que fija en qué momento se decide si el siguiente escenario tiene demanda, la política ante fallas de la cámara, la validación de que cada detector esté bien ligado a su escenario, y la observabilidad que expone el motivo real de cada decisión.

## Qué queda garantizado (resumen de las cuatro fases cerradas)

| Fase | Qué queda garantizado |
|---|---|
| 6 | La decisión sobre si el siguiente escenario tiene demanda se congela una sola vez, en el momento correcto, y no se puede alterar a media transición salvo por un rescate legítimo ante una detección real y verificada. |
| 7 | Cuatro políticas configurables para cuando la cámara falla o reporta un detector desconocido (servir normal, saltar, reintentar cada N ciclos, usar otra fuente), con los cruces peatonales bloqueados en dos capas para que nunca usen una política distinta a la normal. |
| 8 | Cada detector queda validado contra una lista de direcciones conocidas y no puede usarse en dos escenarios distintos dentro del mismo ciclo. |
| 9 | El sistema expone hacia afuera, por ambos lados del contrato, el motivo real de cada decisión y qué tan reciente es la información de la cámara, sin alterar ninguna decisión ya existente. |

## Hallazgo corregido en la fase 6

Al revisar el código real en vez de asumir que ya cumplía la regla documentada, se encontró que el mecanismo de rescate (la única excepción permitida para modificar una decisión ya congelada) usaba la misma función que decide qué hacer cuando la cámara está caída, una función que, por diseño, asume que sí hay demanda cuando no hay información, para no penalizar de más. El mecanismo de rescate usaba ese mismo resultado sin distinguir si venía de una detección real o solo de la caída de la cámara, así que bastaba con una caída de un par de segundos para otorgar una entrada que nunca tuvo evidencia real detrás.

Se corrigió separando las dos funciones, una para el comportamiento normal ante falta de información, y otra estricta que solo cuenta como rescate una detección confirmada. Confirmado con el responsable del proyecto antes de aplicar el cambio.

## Decisiones que requirieron autorización

Dos puntos del encargo de la fase 7 llegaron sin definición concreta, y se consultaron con el responsable del proyecto antes de implementar en vez de asumir:

- cuántos ciclos esperar antes de reintentar cuando la cámara falla — quedó en un ciclo, configurable;
- qué significa exactamente "usar otra fuente" cuando la cámara falla — resultó ser un mecanismo que corresponde al módulo de video, no al de luces, así que se dejó aceptado como valor de configuración pero sin implementar el comportamiento real, documentado explícitamente como pendiente y no como algo resuelto.

En la fase 8 se confirmó también con el responsable del proyecto que un mismo detector usado en dos escenarios distintos entre ciclos diferentes no es ambiguo, solo lo es dentro del mismo ciclo.

## Limitación documentada, no resuelta en esta fase

La fase 8 deja validado que el identificador de un detector tenga el formato correcto, pero no puede confirmar que ese detector exista de verdad configurado del lado de la cámara, porque son dos sistemas separados con configuraciones separadas. Cerrar esto de fondo requeriría una decisión de acoplamiento entre ambos sistemas que no correspondía a esta fase y no se tomó por cuenta propia.

## Verificación

| Fase | Suite completa | Pruebas nuevas |
|---|---|---|
| 6 | 95 correctas / 3 fallas preexistentes sin relación | 7 |
| 7 | 112 correctas / 3 fallas preexistentes sin relación | 17 |
| 8 | 119 correctas / 3 fallas preexistentes sin relación | 7 |
| 9 | 139 correctas / 2 fallas preexistentes sin relación (más 17 correctas / 1 falla preexistente sin relación, del lado de la cámara) | 14 (más 3 del lado de la cámara) |

Antes de agregar el campo nuevo que documenta el motivo de cada decisión (fase 9), se verificó de forma explícita que ese cambio no alterara ninguna decisión real —dado que la misma función ya había tenido el defecto corregido en la fase 6— comparando el comportamiento completo antes y después del cambio.

## Continuidad

- El mecanismo real de "usar otra fuente" cuando la cámara falla (cambiar de transmisión o de cámara redundante) sigue pendiente, es una pieza de trabajo distinta del lado de video.
- La matriz de conflictos entre grupos de semáforo sigue pendiente, confirmada por el responsable del proyecto como prioridad alta; el punto de enganche en el código ya quedó preparado.
- La verificación de que un detector exista de verdad del lado de la cámara (no solo que tenga formato válido) sigue sin resolverse, requiere una decisión de acoplamiento entre ambos sistemas.
- Ninguna entrega de este cierre modifica hardware ni toca el controlador de campo directamente; todo el trabajo vive en el software intermedio entre los dos módulos.
