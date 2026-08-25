# Anexo A - Reporte de cierre: activación y reversión verificada

**Fecha del cierre:** 19 de agosto de 2026
**Origen:** documento interno de cierre de subtarea, entregado al responsable del proyecto.

> **Versión con datos omitidos.** Este anexo reproduce el contenido técnico del reporte interno omitiendo rutas de despliegue, nombres de servicios, huellas de integridad de los paquetes y detalles de configuración de los equipos, conforme al acuerdo de confidencialidad. Lo que se conserva es el problema, el razonamiento, los defectos encontrados y las verificaciones hechas.

---

## Objetivo

Que una actualización no se registre como exitosa hasta comprobar que el proceso quedó corriendo la versión correcta.

## El problema que resuelve

El gestor de servicios del sistema operativo devuelve código de éxito en cuanto el comando de reinicio arrancó, no cuando el proceso quedó funcionando. Un componente que muere a los 500 milisegundos produce un reinicio "exitoso".

Ocurrió al revertir a una versión anterior, el gestor aceptó el reinicio, la bitácora lo registró como correcto, y el proceso murió porque esa versión no incluía el punto de entrada que la unidad de servicio invoca. Nada indicaba que algo estuviera mal.

## Cómo se resolvió

### El proceso publica su identidad

Al arrancar, el componente escribe de forma atómica un archivo de estado con:

| Campo | Para qué sirve |
|---|---|
| Identificador de proceso | Comprobar que es el proceso que el servicio administra |
| Versión | Comprobar que cargó la que se activó |
| Ruta | Directorio del que se importó el paquete |
| Marca de arranque | Cuándo empezó |
| Latido | Se refresca cada 5 segundos; un latido viejo significa que dejó de avanzar |
| Dispositivos | Cuántos controladores de hardware quedaron activos |

El archivo se retira al cerrar, de modo que su ausencia significa que no hay proceso corriendo. Vive en un directorio compartido que sobrevive a actualizaciones y reversiones.

### La herramienta verifica cuatro condiciones

Tras el reinicio, sondea hasta 20 segundos:

1. El servicio sigue activo.
2. Su identificador de proceso es distinto del anterior. Si no cambió, el reinicio no tomó efecto.
3. Existe el archivo de estado y su identificador coincide con el que reporta el gestor de servicios.
4. La versión publicada es la que se acaba de activar.

Más una quinta implícita, el latido no puede tener más de 20 segundos de antigüedad.

Si alguna falla, la herramienta restaura el estado anterior por sí sola y anota el resultado en la bitácora.

### Procesos fuera del servicio

Se detectan y se reportan, pero no se cierran. Matar un proceso ajeno sin saber quién lo lanzó es peor que avisar. Hay una prueba automatizada que falla si el código llega a contener cualquier instrucción capaz de terminar un proceso.

El caso real fue una instancia lanzada a mano que quedó 22 horas corriendo junto al servicio. Las dos abrían la misma pantalla física, así que lo que se veía podía no ser lo que el gestor de servicios administraba.

Se validó en el equipo lanzando a propósito una segunda instancia. La herramienta reportó el proceso duplicado con su aviso, omitió la operación, y el proceso siguió vivo hasta que se cerró a mano, que es el comportamiento pedido.

---

## Defecto encontrado al validar en el equipo

La reversión automática restauraba la versión anterior sin comprobar que esa versión sí pudiera verificarse.

Se probo en laboratorio que revertir de la versión nueva a la inmediata anterior dejó la referencia movida y el servicio muerto, porque esa versión anterior no publica el archivo de estado. El sistema quedó peor que antes de intentar.

Al corregir se rechaza activar o revertir hacia una versión que no publique, antes de mover algo, con un mensaje que explica por qué.

Con una sola versión instalada del mecanismo nuevo, no hay reversión verificable. Las versiones anteriores no están rotas, son de antes del mecanismo, pero no sirven como destino de reversión.

## Defecto en las propias pruebas

Varias pruebas fijaban un número de versión escrito a mano y fallaron al subir la versión del proyecto. Estaban atadas a la versión del repositorio sin que nada lo indicara, y el mensaje de fallo no sugería la causa. Corregido en tres archivos, la versión se lee del contrato del componente en lugar de suponerse.

## Defecto corregido: el estado mentía sobre la reversión

El comando de consulta calculaba si había reversión disponible mirando solo las referencias en disco, sin considerar si la versión anterior podía verificarse. Reportaba que sí se podía revertir mientras el comando de reversión lo rechazaba, quien consultara el estado creía tener una red de seguridad que no tenía.

Al corregir, el campo aplica las mismas guardas que la operación real, y cuando dice que no, explica el motivo.

Se agregaron ocho pruebas que comparan lo que dice el estado contra lo que la reversión realmente acepta, ejecutándola en modo de simulación. Si divergen, fallan. Ninguna prueba anterior verificaba esa coherencia, que es por lo que el defecto pasó desapercibido.

---

## Validación

| Prueba | Resultado |
|---|---|
| Suite de herramientas | 367/367 correctas |
| Suite del componente | 251/251 correctas |
| Guarda de alcance del reinicio | Conservada. Rechaza los servicios ajenos al componente y los intentos de inyección en el nombre |
| No cerrar procesos ajenos | Verificado por análisis del código |
| Pruebas negativas | Quitar la verificación rompe 6 pruebas; quitar la guarda de verificabilidad rompe 2; volver a calcular la disponibilidad de reversión por referencias en disco rompe 3 |

### Verificado sobre el equipo de laboratorio

| Caso | Resultado |
|---|---|
| Activar la versión nueva | Proceso vivo, versión correcta, latido de hace 0 segundos |
| Archivo de estado contra lo que reporta el gestor de servicios | Coinciden |
| Revertir a una versión que no publica, forzando | Lo detectó y lo reportó |
| Activar una versión que no arranca, forzando | Lo detectó y revirtió sola |
| Revertir con la guarda puesta | Rechazado antes de mover nada |
| Estado tras el rechazo | La versión buena sigue activa y el servicio corriendo |
| Coherencia entre el estado consultado y la reversión real | Correcta |
| Proceso duplicado real | Reportado con su aviso, sin cerrarlo |

### Reinicio completo del equipo

Era la última verificación pendiente del criterio de aceptación original, que el servicio arranque junto con el equipo.

Se reinició el equipo de laboratorio y, sin intervención de ninguna clase, el servicio quedó activo. Sin reintentos, arrancó a la primera.

La línea más informativa de la bitácora de arranque fue una desconexión inicial con error de conexión, el módulo del que depende todavía no estaba listo cuando el componente arrancó, y el controlador se reconectó solo medio segundo después. Es el comportamiento que se pedía, esta vez en un arranque real del equipo y no en una prueba provocada.

Confirma además que la unidad de servicio hizo bien en no declarar una dependencia dura sobre el otro módulo, con esa dependencia habría fallado en vez de reintentar.

El riesgo que se había anticipado (que la sesión gráfica no existiera al momento de arrancar) no se materializó, porque el equipo tiene inicio de sesión automático configurado. Conviene tenerlo presente, si un equipo no lo tuviera, este arranque podría fallar.

La revisión de salud general del sistema reportó estado sano tras el reinicio.

---

## Interpretación de un requisito

El criterio original pedía confirmar que "el estado de la pantalla vuelve a actualizarse". Se resolvió con el latido del archivo de estado del proceso y no con el archivo de estado del controlador de pantalla. 

Debido a que ese segundo archivo solo existe si hay un dispositivo habilitado, así que no sirve como criterio de fallo cuando la configuración está apagada. El latido del proceso se refresca siempre que esté avanzando, con o sin dispositivos.

Es equivalente en la práctica, pero no es literalmente lo que dice el texto y conviene que quede asentado.

---

## Continuidad

**Una bandera permite dejar la referencia y el proceso desalineados.** No es un defecto, la bandera existe justamente para mover la referencia sin tocar el servicio, pero después nada lo reporta. Se observó durante las pruebas. El estado podría advertirlo comparando la versión referenciada contra la publicada por el proceso. Queda a decisión del equipo.

**La reversión verificable empieza en la versión nueva.** Hasta que haya dos versiones instaladas que publiquen su estado, no hay reversión posible con garantías.

**El equipo de campo sigue sin tocarse**, por decisión del equipo de trabajo.
