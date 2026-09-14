Anexo B - Reporte de cierre: sincronización de firmware e incidencias resueltas en un controlador de campo

Fecha del cierre: 9 de septiembre de 2026
Origen: documento interno de cierre de subtarea, entregado al responsable del proyecto.

> Versión con datos omitidos. Este anexo reproduce el contenido técnico del reporte interno omitiendo identificadores de equipo, rutas internas, direcciones de red y huellas de integridad de los paquetes, conforme al acuerdo de confidencialidad. Lo que se conserva es el problema, el razonamiento, los defectos encontrados y las verificaciones hechas.

---

## Objetivo

Sincronizar el controlador de un equipo de campo con la versión vigente de laboratorio antes de una ventana operativa programada para el mismo día. El equipo de campo estaba diez versiones atrás, y su propio mecanismo de actualización no reconocía un módulo nuevo agregado recientemente en laboratorio.

## Resultado

Equipo de campo actualizado a la versión vigente, con los mismos servicios activos que el de laboratorio, confirmado contra el estado real del sistema tras la actualización.

## Diagnóstico

El mecanismo de actualización del equipo de campo llevaba meses sin tocarse y no contemplaba el módulo nuevo en ninguno de sus tres puntos de verificación (armado del paquete, aplicación del cambio, chequeo de salud). Cualquier paquete que se le subiera perdería ese módulo en silencio, antes incluso de compararse contra lo ya instalado.

## Incidentes durante el despliegue

**1. Fallo de arranque tras el primer envío.** Un archivo del que dependía el mecanismo de actualización no se incluyó en el primer lote transferido. Se resolvió subiendo el archivo faltante.

**2. Primer intento de activación rechazado por el chequeo de salud.** El nuevo módulo no arrancó por dos causas combinadas, una definición de servicio copiada del release traía un usuario de sistema que no existe en el equipo de campo (que usa un usuario distinto para todos sus servicios), y al entorno del equipo le faltaba una dependencia. El sistema detectó la falla y revirtió solo, sin tiempo real de caída. Se corrigieron ambas causas, se probó el arranque manualmente, y el segundo intento de activación quedó sano.

## Decisiones tomadas

| Decisión | Razón |
|---|---|
| Usar el código ya resuelto por el equipo en el repositorio, en vez de un parche manual | Ya tenía pruebas cubriendo el escenario exacto; menor riesgo que reinventar la lógica bajo presión de tiempo |
| Activar antes de la ventana de mantenimiento habitual | Autorizado explícitamente por el responsable del proyecto, dado que había una ventana operativa programada ese mismo día que no se podía mover |
| Corregir el usuario de sistema solo en el equipo ya desplegado, no en el repositorio | Se priorizó cerrar la ventana operativa; la corrección al repositorio queda pendiente |

## Verificación

Confirmado contra el estado real del sistema tras la actualización, versión activa correcta, mismos servicios que el equipo de laboratorio, todos en ejecución.