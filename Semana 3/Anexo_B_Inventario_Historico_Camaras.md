Anexo B - Reporte de diagnóstico: inventario histórico de cámaras no proyectado

Fecha del diagnóstico: 2 de septiembre de 2026
Origen: documento interno de diagnóstico, entregado al responsable del proyecto para su decisión.

> Versión con datos omitidos. Este anexo reproduce el contenido técnico del reporte interno omitiendo el identificador del nodo, nombres de archivos y tablas internas, y rutas de servicio, conforme al acuerdo de confidencialidad. Lo que se conserva es el problema, el razonamiento, los hallazgos y las opciones documentadas.

---

## Qué se observó

En un nodo real usado para pruebas, el panel de detección actual muestra las cámaras con video en vivo, mientras que la ventana rediseñada reporta que ese nodo no tiene cámaras registradas. Mismo nodo, mismo momento, resultados opuestos.

## Por qué pasa

El inventario de cámaras vive en dos representaciones: un archivo maestro, que es la fuente de verdad histórica del sistema, y una tabla relacional nueva que la ventana rediseñada necesita para poder asignar cada cámara a una dirección semafórica. La segunda se llena proyectando la primera.

Antes de cierto punto de este año, esa proyección solo ocurría si alguien corría una reconciliación manual; dar de alta una cámara no la disparaba de forma automática. Las cámaras de este nodo se registraron en esa época y nunca se proyectaron: quedaron en el archivo maestro pero nunca llegaron a la tabla nueva.

## Lo que ya no es problema

Dar de alta una cámara hoy ya proyecta de forma automática a la tabla nueva, dentro del flujo normal de alta confirmada por el propio controlador. El hueco es solo con inventario histórico anterior a esa corrección, un conjunto acotado que no crece.

## Por qué importa para la decisión de producción

Un operador que abra la ventana nueva en un nodo con inventario histórico sin migrar verá "sin cámaras" y concluirá, con razón aparente, que el rediseño está roto. No lo está, pero la percepción sí sería esa, justo antes de habilitar esta pantalla para operadores reales.

## Opciones documentadas para el responsable del proyecto

| Opción | Qué implica |
|---|---|
| A. Reconciliar una vez, antes de habilitar | Ya existe una acción de reconciliación en el backend que reproyecta la tabla nueva desde el archivo maestro para todo el inventario. No toca hardware, solo actualiza la proyección relacional. Es la opción más limpia. |
| B. Habilitar solo en nodos ya proyectados | Sirve para una prueba controlada, pero deja al operador sin saber qué nodos sí tienen la información completa y cuáles no. No se recomienda como estado permanente. |
| C. No habilitar hasta resolver la migración | La más conservadora. El rediseño queda listo y probado, a la espera de que el inventario esté completo. |

**Recomendación entregada:** opción A, verificándola primero en el ambiente de pruebas para confirmar que pobla los nodos históricos sin efectos inesperados, y después corriéndola en producción en una ventana acordada con el equipo.

## Estado del rediseño, al margen de este hallazgo

La fase que expone esta pantalla en el flujo real ya está cerrada y probada de forma independiente: monta correctamente detrás de un interruptor apagado por defecto, y funciona de punta a punta en cualquier nodo cuyo inventario sí esté proyectado. Lo único que separa a la ventana de operar en cualquier nodo es que el inventario histórico se migre, que es justo lo que este documento pone a decisión.

## Nota de seguridad, aparte

El archivo maestro de inventario guarda credenciales de cámara sin cifrar. Es un pendiente ya reconocido en el propio sistema, no generado por este diagnóstico, pero se deja anotado junto con el resto de credenciales débiles ya señaladas en tareas anteriores.
