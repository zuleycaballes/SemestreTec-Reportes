Anexo A - Reporte de cierre: cámaras ausentes en la ventana de detección tras el reinicio de fábrica

Fecha del cierre: 30 de septiembre de 2026
Origen: documento interno de diagnóstico y cierre, entregado al responsable del proyecto.

> Versión con datos omitidos. Este anexo reproduce el contenido técnico del reporte interno omitiendo nombres de nodo, identificadores de controlador, direcciones de red, nombres de archivos y rutas internas, conforme al acuerdo de confidencialidad. Lo que se conserva es el problema, el razonamiento, los hallazgos y las verificaciones hechas.

---

## Objetivo

Encontrar por qué las cámaras de un nodo no aparecían en la ventana de detección rediseñada después del reinicio de fábrica, aunque sí estaban dadas de alta en el inventario y el panel anterior sí mostraba su video, y dejarlas funcionando sin alterar el diseño de la ventana.

## Resultado

Las cuatro cámaras del nodo ya están en la tabla que consume la ventana de detección, activas y ligadas al controlador activo del nodo, y la ventana muestra video (confirmado por el equipo). El rediseño de la ventana no tenía defecto, el problema era de estado de datos.

## Corrección al diagnóstico de la primera sesión

El diagnóstico del 29 de septiembre era correcto pero incompleto: corregir solo lo que describía no bastaba para que las cámaras llegaran a la tabla de visión. La sesión del 30 de septiembre encontró las causas restantes, y este anexo sustituye el arreglo propuesto en la primera.

## Cadena técnica confirmada

La ventana de detección lee de una tabla de cámaras de visión, que se llena proyectando desde el inventario de cámaras del sistema. Esa proyección es de todo o nada, si una sola fuente apunta a un controlador que no existe, rechaza proyectar todas. Además, el alta de cámara llama a la proyección y traga su error a propósito, para que el alta hacia el equipo de campo no falle. Por eso ninguna de las causas dejaba un error visible para el operador, solo "la cámara no aparece".

## Causas encontradas

| # | Causa | Cómo se confirmó |
|---|---|---|
| 1 | El reinicio de fábrica recreó los controladores con identificadores nuevos. | Había dos controladores con el mismo nombre en cada nodo: uno activo (el nuevo) y uno inactivo (el anterior). |
| 2 | Borrar un controlador es un borrado lógico: la fila solo se marca como inactiva y permanece. Como el alta de cámara ofrecía dos controladores con nombre idéntico, las cuatro cámaras quedaron ligadas al inactivo. | Consulta al inventario: las cuatro cámaras apuntaban al controlador inactivo. |
| 3 | La proyección es todo o nada, y había tres fuentes huérfanas (apuntaban a controladores que ya no existen) que bloqueaban a todas. | Tras re-ligar las cuatro cámaras, la proyección siguió fallando por dos identificadores de controlador inexistentes; al borrar las tres huérfanas, pasó. |
| 4 | La consulta que arma el inventario de detección de un nodo cruzaba con la tabla de controladores sin filtrar los inactivos, por lo que las configuraciones de controladores dados de baja reaparecían como "No registrada en SGSI". | Las configuraciones de detección de los dos controladores inactivos (4 cámaras y 2 cámaras) seguían presentes y aparecían en el inventario. |

Las tres fuentes huérfanas eran restos de un inventario viejo, su última sincronización era de junio, estaban en una red distinta a la de los nodos actuales y las tres tenían la misma fecha y hora de creación. No eran cámaras reales del nodo donde se les buscó.

## Cambios aplicados

| # | Cambio | Resultado |
|---|---|---|
| 1 | Se ligaron las cuatro cámaras al controlador activo de su nodo. | 4 filas actualizadas, verificado con consulta. |
| 2 | Se actualizaron las llaves de un mapa interno de cámaras (previo ensayo con reversión). | 4 filas. No influye en el comportamiento (ver correcciones a diagnósticos intermedios); es inofensivo. |
| 3 | Se re-ligaron dos de las fuentes huérfanas a un controlador activo de otro nodo. | Sin efecto práctico: se borraron en el cambio siguiente. |
| 4 | Respaldo (`pg_dump`) de las tablas de controladores, fuentes de cámara, credenciales de fuentes y cámaras de visión, seguido del borrado de las tres fuentes huérfanas. | 3 filas borradas; quedan 4 fuentes. Las credenciales asociadas se limpian en cascada. |
| 5 | Proyección del inventario completo. | Correcta; las cuatro cámaras quedan activas en la tabla de visión. |
| 6 | Parche de backend: se agrega el filtro de controladores activos al cruce de la consulta de inventario de detección, con una nota en su documentación interna. Compila. | Aplicado por un compañero del equipo. El inventario del nodo de calle muestra 2 filas no registradas en lugar de las 4 esperadas sin el parche. |

## Correcciones a diagnósticos intermedios

- Las llaves del mapa interno de cámaras no influyen en la lista de "no registradas", esa lista compara el controlador de la fuente contra los valores del mapa, no contra sus llaves. El cambio 2 no era necesario.
- Una de las fuentes huérfanas no era duplicado de una cámara real; era una cámara distinta (otro equipo, otro nodo, otro controlador).
- Las fuentes huérfanas no eran cámaras reales del nodo de calle al que se re-ligaron temporalmente.
- Las filas "No registrada en SGSI" que aparecieron en el nodo de laboratorio no salen del inventario de cámaras, las arma el backend desde las configuraciones de detección. Aparecieron al re-ligar las cámaras, que hasta entonces tapaban la configuración del controlador inactivo, por lo que son consecuencia del arreglo y no un daño.

## Criterios verificados

| Criterio | Estado |
|---|---|
| La proyección corre sin error de catálogo inválido | Cumple |
| Las cuatro cámaras están activas en la tabla de visión | Cumple |
| Ninguna fuente del inventario apunta a un controlador inexistente o inactivo | Cumple (4 filas, todas en el controlador activo) |
| Video en la ventana de detección del nodo | Cumple, confirmado por el equipo |
| Las cámaras son visibles en dispositivos de campo del nodo | Cumple, con captura del inventario |
| Configuraciones de controladores inactivos excluidas del inventario | Aplicado |
| Origen de las tres fuentes huérfanas | No confirmado, solo hipótesis |

## Continuidad

- **Origen de las tres filas viejas.** Hipótesis: alguien corrió un script de importación de inventario de cámaras el 23 de septiembre. Debe confirmarlo el responsable de bases de datos. El despliegue no las resucita.
- **Mejoras propuestas, sin implementar.** Registrar en log el descarte silencioso del error de proyección; filtrar los controladores inactivos en la lista de selección del alta de cámara; y que la proyección omita las fuentes huérfanas con un aviso y proyecte el resto. La última cambia una decisión de diseño y queda a criterio del responsable del backend.
- **Purga de controladores dados de baja**: no se ejecutó. No es necesaria para el síntoma, y un borrado real puede afectar relaciones e historial.
- **Respaldos.** El respaldo del cambio 4 está en una carpeta temporal que se limpia al reiniciar. No se confirmó un respaldo previo a los cambios 1 a 3.
