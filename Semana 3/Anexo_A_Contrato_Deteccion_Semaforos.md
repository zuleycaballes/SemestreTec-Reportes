Anexo A - Reporte de cierre: contrato de comunicación entre el módulo de detección de vehículos y el módulo de luces

Fecha del cierre: 3 de septiembre de 2026
Origen: documento interno de cierre de contrato, entregado al responsable del proyecto.

> Versión con datos omitidos. Este anexo reproduce el contenido técnico del reporte interno omitiendo nombres de archivos, rutas internas, identificadores de nodo y detalles de configuración de los equipos, conforme al acuerdo de confidencialidad. Lo que se conserva es el problema, el razonamiento, las decisiones y las verificaciones hechas.

---

## Objetivo

Formalizar y endurecer el contrato de datos entre el módulo que detecta vehículos por cámara y el módulo que decide las fases del semáforo, de modo que ninguno de los dos lados tenga que confiar en que el otro se comporta bien, sino que lo verifique.

## El problema que resuelve

Antes de este trabajo, el canal entre ambos módulos no tenía garantías explícitas, no había un límite estricto de qué tan seguido se publica un cambio, no había validación de lo que llega antes de aplicarlo, y la memoria de corto plazo que recuerda una demanda pendiente no dejaba rastro de por qué desaparecía. Un dato corrupto o incompleto podía llegar a mutar el estado real de un semáforo sin que nada lo detuviera.

## Qué queda garantizado (resumen de las cinco fases cerradas)

| Fase | Qué queda garantizado |
|---|---|
| 1 | Semántica exacta del "estado en línea" del módulo de detección y de los tres estados válidos que puede tener un detector; ninguna combinación inválida puede salir del lado emisor. |
| 2 | Una detección de un único cuadro de video ya no puede disparar ni apagar una demanda; se exige presencia (o ausencia) sostenida un tiempo mínimo antes de reaccionar. |
| 3 | El lado emisor respeta siempre un límite estricto de frecuencia de publicación, manda una señal de "sigo vivo" aunque no haya cambios, y reintenta con espera creciente si falla el envío. |
| 4 | El lado receptor valida por completo cualquier paquete antes de aplicar cualquier cambio; si algo no cumple, se rechaza entero, no se aplica a medias. |
| 5 | La memoria de corto plazo de una demanda pendiente queda formalizada: quién la activó, en qué momento, qué la puede resolver, si ya se atendió, y el motivo exacto si se elimina sin haberse atendido. |

## Decisiones que requirieron autorización

Un par de puntos de este trabajo no eran técnicamente ambiguos, pero sí cambiaban comportamiento de un sistema en operación real, así que se confirmaron con el responsable del proyecto antes de implementarse:

- corregir un error de agrupación en cómo se combina la señal de varias cámaras de un mismo detector (una cámara degradada ya no invalida a sus cámaras hermanas sanas del mismo detector);
- exigir que cada paquete de datos represente siempre el estado completo de todos los detectores configurados, rechazando cualquier envío parcial en vez de tolerarlo como excepción.

## Verificación

| Suite corrida | Resultado |
|---|---|
| Suite completa del lado receptor | 91 correctas / 2 fallas preexistentes sin relación con este trabajo |
| Suite dedicada al endurecimiento del punto de recepción | 19 de 19 correctas |
| Prueba manual: rechazo de combinación inválida de estado | Confirmado, el receptor la corrige incluso si llegara |
| Prueba manual: reemplazo atómico del estado completo | Confirmado, un detector ausente del paquete pasa a "sin información", no se queda con su último valor conocido |
| Prueba manual: registro de diagnóstico | Cuatro llamadas de prueba generaron exactamente las líneas de registro esperadas, ni una de más |

## Continuidad

- Falta calibrar en campo un umbral opcional relacionado con la calidad de la señal de video; hoy está desactivado por default.
- Un envío de datos adicionales que hoy viaja en el mismo canal que la decisión sigue pendiente de moverse a un canal aparte de telemetría; no bloquea nada y ya quedó documentado como pendiente explícito.
- Ninguna entrega de este cierre modifica hardware ni toca el controlador de campo directamente; todo el trabajo vive en el software intermedio entre los dos módulos.
