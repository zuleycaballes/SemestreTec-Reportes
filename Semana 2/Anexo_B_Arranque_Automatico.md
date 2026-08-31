# Anexo B - Reporte de cierre: arranque automático del módulo de periféricos

**Fecha del cierre:** 27 de agosto de 2026
**Origen:** documento interno de cierre de subtarea, entregado al responsable del proyecto.

> **Versión con datos omitidos.** Este anexo reproduce el contenido técnico del reporte interno omitiendo identificadores de equipo, rutas internas, huellas de integridad y datos de arranque específicos del sistema, conforme al acuerdo de confidencialidad. Lo que se conserva es el problema, el razonamiento, las verificaciones hechas y sus resultados.

---

## Objetivo

Confirmar que la pantalla se recupera tras reiniciar el equipo sin depender de una sesión remota, un proceso manual ni archivos temporales, y dejar la evidencia registrada.

## Por qué se repitió la prueba

La tarea llegó describiendo un reinicio ya hecho, con datos que no correspondían al estado actual del equipo, reportaba cero reinicios no planeados y una unidad recién instalada, pero el tiempo de actividad del equipo mostraba un arranque de días antes, y la unidad activa en ese momento era la anterior a la corrección de portabilidad del entorno gráfico, la que fijaba el directorio de entorno de usuario a mano en vez de resolverlo de forma automática.

Documentar esa evidencia habría descrito una configuración que ya no está en producción. Se rehizo el reinicio con la versión vigente del componente y la unidad de servicio actual, que es además la prueba más relevante, confirma si el mecanismo resuelve el entorno gráfico cuando el sistema lo arranca automáticamente, sin que nadie haya iniciado sesión todavía.

## Evidencia del reinicio

| Dato | Valor |
|---|---|
| Versión activa | vigente |
| Arranque del servicio | segundo 0 |
| Entorno gráfico resuelto | mismo segundo |
| Pantalla abierta con datos | segundo 1 |
| Conectado al módulo de semáforos | segundo 1 |
| Reinicios no planeados tras el arranque | 0 |

Un segundo entre el arranque del servicio y tener pantalla con datos. El mecanismo resolvió el usuario y encontró el socket gráfico sin reintentos, la sesión ya estaba lista cuando el sistema lanzó la unidad.

## Comprobaciones antes y después del reinicio

Se guardó una snapshot del estado antes de reiniciar, en un directorio persistente, para comparar contra lo que sobreviviera al propio reinicio.

| Comprobación | Antes | Después |
|---|---|---|
| Huella de la configuración activa | valor de referencia | idéntico |
| Copia de prueba en directorio temporal | existía | desapareció, como corresponde a un archivo temporal |
| Copia en almacenamiento persistente | — | sobrevivió al reinicio |
| Instancias del proceso | 1 | 1, mismo identificador que reporta el sistema |
| Diagnóstico de salud del componente | sano | sano |
| Diagnóstico de runtime | sano | sano, con antigüedad de dato de 28 a 37 milisegundos |
| Confirmación visual | — | la pantalla volvió a mostrar el contenido sin intervención |

La configuración persistente sobrevivió intacta, y el archivo temporal se perdió como se esperaba de algo bajo un directorio temporal.

## Prueba adicional: dependencia tardía del módulo de semáforos

La tarea la dejaba como opcional, condicionada a que la situación se diera sola. No se dio durante el arranque, así que se provocó de forma controlada, deteniendo el módulo de semáforos y reiniciando el componente de periféricos.

Se observó un retroceso creciente en los reintentos de conexión mientras el módulo de semáforos estuvo caído, medio segundo, un segundo, dos segundos. Al restablecerlo, el componente se reconectó solo, sin que el componente de periféricos se reiniciara. Estado final, sano, con antigüedad de dato de 37 milisegundos.

## Lo que no quedó cubierto

Apagado físico o pérdida completa de energía. Se reportó que el equipo ya había pasado por apagados completos con recuperación exitosa de la pantalla, pero el registro de arranques del equipo solo conserva el arranque actual. No hay registro persistente de los apagados anteriores, así que esa evidencia, identificador de arranque, tiempos, resultado, no se puede documentar retroactivamente aunque el hecho haya ocurrido.

Se decidió no provocar un apagado físico adicional en esta sesión. Queda como la prueba de campo que la tarea contemplaba como más estricta que el reinicio controlado, con su propia autorización.

## Alcance de este cierre

Siguiendo el criterio que la propia tarea planteaba:

- **Si el criterio es un reinicio normal, completado.** Verificado con la configuración actual, no con la anterior.
- **Dependencia tardía del módulo de semáforos, completada**, provocada de forma controlada.
- **Pérdida total de energía, pendiente de evidencia documentable.** El hecho pudo haber ocurrido antes, pero no dejó registro que se pueda citar.

## Continuidad

No debe usarse la evidencia de un reinicio anterior a la corrección de portabilidad para dar por válido el arranque automático, la unidad que fijaba el usuario a mano ya no está en producción, y es precisamente el comportamiento que se corrigió.

Si se requiere evidencia formal de pérdida de energía, hace falta un apagado físico controlado, con autorización separada, en una ventana donde interrumpir el módulo de semáforos unos segundos sea aceptable. El procedimiento es el mismo que el de este reinicio, instantánea antes, identificador de arranque y tiempos después, confirmación visual.

El diagnóstico de salud en tiempo real y el endpoint público no se tocaron en esta validación, conforme a lo indicado, usan criterios ya cerrados en tareas separadas.

El equipo de campo sigue sin tocarse.
