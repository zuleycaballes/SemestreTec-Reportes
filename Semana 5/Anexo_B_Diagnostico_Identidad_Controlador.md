Anexo B - Reporte de diagnóstico: desfase de identidad entre el registro central y un controlador de campo

Fecha del diagnóstico: 17 de septiembre de 2026
Origen: documento interno de diagnóstico, entregado al responsable del proyecto.

> Versión con datos omitidos. Este anexo reproduce el contenido técnico del reporte interno omitiendo identificadores de nodo, direcciones de red y detalles de configuración de los equipos, conforme al acuerdo de confidencialidad. Lo que se conserva es el problema, el razonamiento, los hallazgos y las opciones documentadas.

---

## Alcance

Validar que un controlador de campo real cumple al 100% el contrato de configuración, como parte de las pruebas previas a habilitar despliegues hacia hardware real. No se completó, se encontró un desfase de identidad real entre el sistema central y el controlador, que bloquea el envío de configuración hacia ese equipo mientras no se decida cómo resolverlo.

## Qué se confirmó

Los casos válidos del conjunto de pruebas, al enviarse directamente al controlador para validación, convergen correctamente al mismo resultado ejecutable. Los casos inválidos del mismo conjunto (variaciones de versión, huso horario, grupos, escenarios, programa y tiempos de ejecución) fallan todos antes de persistirse, con el código y la categoría exactos esperados, sin que ningún rechazo tocara el registro de configuración del controlador.

## Hallazgo: el sistema central y el controlador no coinciden en cómo identifican al mismo equipo

El sistema central identifica internamente a cada controlador registrado con una clave propia de registro. El controlador físico, en cambio, reporta y exige, para aceptar cualquier configuración nueva, su propia identidad nativa de hardware. El proceso que arma la configuración en el sistema central siempre usa la clave interna de registro, no la identidad nativa del equipo.

Reproducido en los tres casos válidos del conjunto de pruebas (tres tipos de programa distintos), con y sin una bandera de reasignación explícita, la inspección previa pasa incluso con reasignación, pero el envío final es rechazado por el controlador en los tres casos, porque la identidad que trae la configuración armada no coincide con la identidad activa del equipo.

**Esto ya había pasado antes con este mismo controlador**, en un incidente anterior no relacionado, donde dos registros distintos del sistema tenían identificadores distintos para el mismo equipo. Es la segunda vez que este controlador en particular tiene un desfase de identidad entre dos fuentes de verdad; vale la pena tratarlo como un patrón y no como un incidente aislado.

Existe en el sistema central un campo pensado para guardar la identidad física de cada controlador, pero está vacío en los dos controladores registrados; parece ser la columna pensada para esto, nunca conectada al proceso que arma la configuración.

## Opciones documentadas para el responsable del proyecto

| Opción | Qué implica |
|---|---|
| A. Poblar el campo de identidad física y usarlo en el proceso de armado | No toca nada ya desplegado; requiere completar ese campo para cada controlador registrado |
| B. Reconfigurar la identidad de los controladores para usar la clave interna del sistema central | Sí requiere tocar la configuración de cada equipo físico desplegado |

Escalado al responsable del proyecto con ambas opciones documentadas, en una versión directa y otra detallada. Sin resolver al cierre de este diagnóstico.

## Continuidad

- Decisión del responsable del proyecto sobre cuál opción tomar, antes de tocar el proceso que arma la configuración o cualquier equipo ya desplegado.
- Confirmar si el desfase de identidad detectado en este controlador se repite en otros equipos de campo, de cara a la instalación de nuevos nodos planeada más adelante.
