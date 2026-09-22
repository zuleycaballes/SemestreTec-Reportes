Anexo A - Reporte de cierre: contrato de comunicación entre el módulo de detección de vehículos y el módulo de luces (fase 10 - pruebas, y resolución de un conflicto de integración)

Fecha del cierre: 16 de septiembre de 2026
Origen: documento interno de cierre de contrato, entregado al responsable del proyecto.

> Versión con datos omitidos. Este anexo reproduce el contenido técnico del reporte interno omitiendo nombres de archivos, rutas internas, identificadores de nodo y detalles de configuración de los equipos, conforme al acuerdo de confidencialidad. Lo que se conserva es el problema, el razonamiento, las decisiones y las verificaciones hechas.

---

## Objetivo

Cerrar la última fase pendiente del contrato de comunicación entre el módulo de detección de vehículos y el módulo de luces (fase 10, continuación directa de las fases 1 a 9 cerradas en semanas anteriores), verificando que los 25 criterios de aceptación acordados para el contrato completo estén realmente cubiertos por pruebas, y no solo documentados como cubiertos.

## Metodología de cierre

Antes de escribir cualquier prueba nueva se auditó cada uno de los 25 criterios contra el código y las pruebas ya existentes de fases anteriores, verificando cada uno contra el comportamiento real. De los 25, 22 ya estaban cubiertos. Los 3 huecos se cerraron con pruebas nuevas, incluyendo uno sobre la función geométrica más básica de todo el módulo de detección, que nunca había tenido una prueba directa en toda la historia del proyecto.

Una prueba adicional recorre la cadena completa del contrato paso por paso, verificando en cada uno no solo el resultado sino el motivo real que lo produce, para confirmar que la razón coincide con la esperada y no solo la conclusión.

## Resolución del conflicto de integración

Al fusionar esta fase con la rama principal del repositorio, se encontró que otra línea de trabajo había integrado, en paralelo y desde un punto anterior, una parte distinta del mismo contrato, junto con un sistema separado de configuración construido por otro equipo. Las dos historias de desarrollo chocaron.

**Alcance:** 9 archivos marcados por la herramienta de control de versiones como en conflicto, y 4 archivos más tocados por la fusión automática sin ningún marcador, porque el problema no era textual sino de comportamiento.

**Decisiones de arquitectura confirmadas con el responsable del proyecto antes de resolver:**
- Uso de una variante propia frente a una alternativa del otro lado para representar la identidad de un detector, adaptando la fase correspondiente a esa decisión.
- Una fase completa, ausente del otro lado, se replica íntegra en vez de descartarse.

**Corrupción de contenido introducida por el merge automático, sin marcador de conflicto, reparada a mano:** dos definiciones completas duplicadas por el merge (se eliminó la copia sobrante); el límite entre dos definiciones relacionadas quedó corrompido y una de ellas desapareció por completo (reconstruida); un bloque de lógica completo desapareció de una función central (restaurado); dos pruebas de una fase anterior se cortaron a la mitad durante el merge (restauradas y verificadas de nuevo). Ninguno de estos cuatro casos traía marcador de conflicto, si alguien solo hubiera resuelto lo marcado, se habrían colado a producción sin que nadie lo notara.

**Mejoras legítimas adoptadas del otro lado de la fusión, sin perder nada propio:** códigos de error estructurados junto al mensaje en las respuestas; códigos de estado HTTP diferenciados según el tipo de rechazo, en vez de un único código genérico; validación más flexible de la forma del dato entrante, para que un dato malformado pase por la validación propia en vez de por un rechazo genérico previo; una verificación de que el dato realmente cambió antes de contarlo como una actualización; manejo de errores más específico al interpretar una confirmación entrante; una verificación adicional sobre un mapa de estado que ya existía; registro de qué origen mandó la última actualización; y una suite de pruebas de extremo a extremo contra el servicio real, que no se había construido antes.

## Los 3 puntos que necesitan confirmación explícita del responsable del proyecto

1. **Se quitó una validación que hacía imposible una fase ya aprobada.** Dos partes distintas del sistema traían, cada una, un chequeo que exigía una condición que la otra parte rechazaba explícitamente; no existe ningún dato que satisfaga ambos chequeos a la vez, así que la fase quedaba inutilizable bajo cualquier circunstancia. Dado que el responsable del proyecto ya había confirmado que esa fase debía replicarse, se quitó el chequeo que lo impedía.
2. **Un archivo de prueba compartido se renombró y sus verificaciones de integridad se recalcularon**, porque usaba nombres de detector inválidos para el nuevo esquema. El archivo se describe a sí mismo como un contrato compartido con otro sistema. Es local a este repositorio, así que se procedió, pero ese lenguaje sugiere que podría representar algo que otro equipo espera ver sin cambios, y no hay forma de confirmar eso desde aquí.
3. **Se eliminaron 3 pruebas que verificaban exactamente la validación removida en el punto 1**, porque, con la validación quitada, esas pruebas afirmaban lo contrario de la decisión ya confirmada. Una cuarta prueba del mismo archivo, no relacionada con ese mecanismo, se dejó intacta.

## Verificación

| Prueba | Resultado |
|---|---|
| Suite completa tras la fusión | 281 correctas / 0 fallidas / 13 omitidas por diseño |
| Falla intermitente preexistente, no relacionada | Confirmada como preexistente, pasa consistente en aislado |

## Continuidad

- Confirmar los tres puntos señalados con el responsable del proyecto.
- Si el archivo de prueba renombrado en el punto 2 resulta ser consumido por otro sistema o equipo, sus nuevas verificaciones de integridad deben propagarse ahí también.
