# Reporte Semanal

| Campo | Dato |
| --- | --- |
| **Nombre** | Zuleyca Guadalupe Balles Soto |
| **Matrícula** | A01741687 |
| **Programa Académico** | ITC |
| **Empresa** | Dirección de Vialidad y Semaforización del Ayuntamiento de Hermosillo |
| **Tutor Académico** | Mario Durán Vega |
| **Fecha** | 4 de octubre de 2026 |
| **Periodo reportado** | Semana 7 - 28 de septiembre al 2 de octubre de 2026 |

> **Nota sobre confidencialidad.** El proyecto está sujeto a un acuerdo de confidencialidad. Este reporte describe metodología, decisiones técnicas y aprendizajes obtenidos, omitiendo identificadores de infraestructura, rutas de despliegue, credenciales y cualquier dato que permita ubicar equipos o instalaciones específicas.

---

## 1. Descripción de la fase u objetivos desarrollados en la semana declarados en el cronograma

### 1.1 Contexto del proyecto

Los términos **nodo**, **ventana de detección** y **contrato de comunicación** ya quedaron definidos en reportes anteriores. Se agregan dos para esta semana. **Sincronía** es el módulo con el que se edita el programa de ciclos de un controlador (tiempos de verde por fase, modo de operación) y se envía al equipo. **Identidad de controlador**, ya introducida la semana 5, es la forma en que el sistema central y el equipo físico reconocen que una configuración le pertenece; esta semana quedó separada en tres piezas (registro interno, puesto lógico e identidad aplicada en el equipo). La semana giró en torno a la puesta al día del despliegue a producción tras la reconstrucción de la semana anterior, el diagnóstico de por qué las cámaras no llegaban a la ventana de detección tras el reinicio de fábrica, la corrección de la presencia de vehículos en el croquis de la ventana de detección, y la prueba de punta a punta del flujo de Sincronía (PC-10).

### 1.2 Actividades declaradas en el cronograma para la Semana 7

Según el cronograma, la Semana 7 corresponde a las mismas Actividades 5, 6 y 8 de la semana anterior:

- **Actividad 5.** Supervisar la integración entre hardware y software, verificando la correcta comunicación con controladores, dispositivos en campo y sistemas embebidos.
- **Actividad 6.** Proponer e implementar mejoras en la arquitectura del sistema, orientadas a incrementar la estabilidad, escalabilidad y facilidad de mantenimiento.
- **Actividad 8.** Optimizar la lógica de control semafórico, ajustando ciclos, transiciones y sincronización entre intersecciones para mejorar el flujo vehicular.

### 1.3 Objetivos efectivamente perseguidos

**Actividad 6 - cubierta.** El despliegue automático a producción fallaba en el paso que compara los scripts de operación del repositorio contra los instalados en el servidor, porque el instalado era de una versión previa a un cambio ya integrado. Se reinstaló y, antes de tocar producción, se ensayaron sobre una copia de la base de datos las tres migraciones pendientes, se aplicaron en producción y el sistema quedó con el release nuevo. Esas migraciones además separan la identidad de controlador en registro interno, puesto lógico e identidad aplicada en el equipo, lo que atiende el desfase de identidad documentado en la semana 5 y fue validado en laboratorio con la PC-10. En paralelo se diagnosticó por qué las cámaras no aparecían en la ventana de detección tras el reinicio, las causas estaban encadenadas al estado de datos, que se corrigieron con respaldo previo, más una consulta del backend que hacía reaparecer configuraciones de controladores dados de baja y que se parchó. Detección volvió a mostrar video.

**Actividad 5 - cubierta.** Se ejecutó la prueba PC-10 en el controlador de laboratorio, cargar la configuración desde el equipo, editarla y guardarla sin enviar, exportarla e importarla, enviarla, releerla y comparar su huella (hash) para confirmar que ambos lados convergen. Se probó también que los rechazos (controlador inactivo, no asociado, inexistente o sin conexión) fallan sin escribir nada y sin registrarse como envíos exitosos, y que al enviar solo cambia el controlador destino, el controlador de calle quedó intacto. Todos los pasos pasaron salvo el 8e y el 14 (aislamiento entre controladores del mismo nodo), que requieren hardware multicontrolador que aún no llega. Resultado: **aprobado con parcial**. En la misma línea de integración, se logró que el croquis de la ventana de detección refleje la presencia real de vehículos con la configuración enviada al controlador de laboratorio.

**Actividad 8 - cubierta parcialmente.** La PC-10 validó el canal por el que se programan ciclos, el programa guardado vuelve idéntico, el PDF exportado no suma los tiempos de transición y el envío al controlador converge. *Justificación de cobertura parcial:* no se ajustó ningún ciclo en equipos de calle; la validación fue sobre el controlador de laboratorio y el de calle no se tocó. El ajuste real de ciclos y de sincronía entre intersecciones sigue condicionado a la instalación de los controladores del corredor.

### 1.4 Relación entre el cronograma y el trabajo de la semana

Esta semana el cronograma y el trabajo realizado coincidieron en las tres actividades declaradas (5, 6 y 8), aunque la Actividad 8 quedó en fase de validación y no de optimización, condicionada a que el hardware del corredor todavía no está instalado.

**Actividad no programada para esta semana.** El diagnóstico de las cámaras y la falla del despliegue son seguimiento a incidencias (Actividad 7), que el cronograma programó hasta la semana 5. Se atendieron porque bloqueaban la operación del sistema y de la ventana de detección.

---

## 2. Descripción del desarrollo de las actividades semanales (metodología, recursos, procedimientos)

### 2.1 Metodología

El criterio de la semana fue no tocar producción sin ensayo previo y no dar una causa por buena sin ver los datos reales. Los cambios riesgosos se ensayaron primero sobre una copia (migraciones) o se respaldaron antes de ejecutarse (borrado de registros). Los diagnósticos se hicieron por capas, cada hipótesis se comparó con una consulta a la base de datos, y las que resultaron falsas se descartaron de forma explícita en el reporte. Donde el cambio afectaba equipos en calle o lógica de firmware, no se ejecutó, se dejó propuesto para aprobación.

### 2.2 Recursos y herramientas

- **Infraestructura:** servidor de producción Linux, base de datos relacional con su runner de migraciones versionado, integración continua con runner propio y reversión automática, controlador de laboratorio con firmware propio.
- **Aplicación:** backend en Python, frontend en JavaScript con biblioteca de interfaz basada en componentes, WebSocket de visión que transmite cuadros y detecciones por zona (ROI) del controlador al SGSI.
- **Verificación:** respaldos con `pg_dump` y consultas de solo lectura.

### 2.3 Procedimientos seguidos y actividades realizadas

**Despliegue a producción.** El workflow compara el script de despliegue del repositorio con la copia instalada en el servidor y falla a propósito si difieren. Se reinstaló desde la copia del runner. Al destrabarlo apareció un segundo bloqueo, tres migraciones pendientes. Se revisó qué hace cada una, se hizo un dump, se restauró en una base temporal y ahí se corrió simulación y aplicación, sin errores. Solo entonces se aplicaron en producción y se relanzó el job. Después del despliegue, el endpoint respondió sano con todas sus dependencias y el registro del backend no mostró errores en los minutos siguientes. Se actualizó además el playbook de despliegue.

**Cámaras ausentes en Detección.** El rediseño de Detección lee de una tabla, que se llena proyectando desde el inventario de cámaras. Esa proyección es de todo o nada y, por diseño, su error es silencioso para que el alta de cámara no falle, por lo que nadie veía una causa. En el diagnóstico se identificó que el reinicio de fábrica recreó los controladores con identificadores nuevos y que "borrar" un controlador es un soft delete, de modo que las cuatro cámaras quedaron ligadas a un controlador inactivo con el mismo nombre que el activo. Corregir solo eso no bastó, pues se encontraron tres fuentes huérfanas que bloqueaban la proyección completa, y una consulta del backend que no filtraba controladores inactivos. Se re-ligaron las cuatro cámaras al controlador activo, se hizo `pg_dump` y se borraron las tres huérfanas, y se parchó la consulta. La proyección corrió sin error y Detección mostró video. Detalle completo en **Anexo A**.

**Presencia de ROI en el croquis.** Los ROI estaban dibujados y con cámara asignada, pero el croquis mostraba "Sin datos" con vehículos presentes; tras un primer arreglo, alternaba entre Ocupado y "Sin datos". El análisis encontró tres causas, el frontend no cruzaba las detecciones con las direcciones del croquis, el borrador de ROI no se había enviado al controlador (el firmware no conocía los ROI y no emitía detecciones) y los mensajes de estado, que llegan cada 0.5 s sin detecciones, pisaban las detecciones previas. Se corrigieron dos archivos del frontend y se verificó sintaxis con esbuild.

**PC-10, flujo completo de Sincronía.** Se ejecutó la prueba de 14 pasos sobre el controlador de laboratorio, usando el de calle solo como identidad de texto en pruebas negativas. El primer día salió un bug bloqueante, al abrir Sincronía, la app leía el equipo y sobrescribía en la base de datos el borrador guardado sin enviar. Ahora no se lee automáticamente si hay borrador pendiente y se pregunta antes de reescribir el equipo. Se corrigió en el frontend con pruebas y quedó integrado a la rama principal. El segundo hallazgo fue de identidad, el controlador de laboratorio reportaba la identidad del ambiente de desarrollo. Las migraciones aplicadas separan ahora el registro interno, el puesto lógico y la identidad aplicada, y esta última se llenó sola en la primera lectura y coincide con lo que reporta el equipo. Las pruebas negativas se hicieron con códigos esperados (409, 404 y 502), y la de controlador sin conexión con una regla temporal de firewall. Detalle completo en **Anexo B**.

### 2.4 Evidencia

El respaldo de lo descrito son los reportes técnicos redactados durante la semana, despliegue y migraciones, cámaras en Detección, presencia de ROI en el croquis y resultado de la PC-10. Son documentos internos del proyecto, así que no se anexan completos por el acuerdo de confidencialidad.

Se anexan dos de ellos en versión con la información sensible omitida:

- **Anexo A.** Cámaras ausentes en Detección tras el reinicio de fábrica: causas, correcciones y criterios verificados.
- **Anexo B.** Prueba PC-10 del flujo de Sincronía: resultados por paso, observaciones y pendientes.

Los demás pueden entregarse en el mismo formato si el asesor los requiere.

---

## 3. Listado de las actividades siguientes o pendientes no resueltas, con justificación

### 3.1 Estado de las actividades declaradas para la semana

**Actividad 8, optimizar la lógica de control semafórico.** Cubierta parcialmente. *Justificación:* esta semana se validó el canal de programación de ciclos en el controlador de laboratorio, pero no hay ajustes reales de ciclo que aplicar mientras los controladores del corredor no estén instalados. *Plan de acción:* retomar el ajuste de ciclos y de sincronía, con el modelado de la semana 6 como referencia, en cuanto el hardware del corredor esté instalado. *Resultado esperado:* primer ajuste real de ciclos sobre un corredor coordinado. Las Actividades 5 y 6 se cubrieron; la 5 conserva dos pasos de prueba pendientes (punto 1 de abajo).

### 3.2 Pendientes generados en la semana

**1. Pasos 8e y 14 de la PC-10 (aislamiento entre controladores del mismo nodo).** *Justificación:* no existe aún un nodo con más de un controlador. *Plan de acción:* ejecutarlos cuando llegue el hardware multicontrolador.. *Resultado esperado:* cerrar la PC-10 como aprobada completa.

**2. Protección contra cuadros viejos en el croquis.** *Justificación:* con el parche, si una cámara deja de mandar cuadros el croquis conserva el último estado en lugar de volver a "Sin datos"; antes, los mensajes de estado borraban las detecciones cada 0.5 s y actuaban como una expiración accidental. *Plan de acción:* marcar el momento de recepción de cada cuadro y tratar como "Sin datos" cualquier cámara que pase unos 3 s sin mandar uno. *Resultado esperado:* croquis confiable antes de usar esa señal en un cruce real.

**3. Cierre del diagnóstico de cámaras.** *Justificación:* quedan un par de puntos que dependen de otra persona. Primero, el origen de las tres fuentes huérfanas es solo una hipótesis (un script de importación de inventario). Segundo, hay tres mejoras propuestas sin implementar (registrar en log el error silencioso, filtrar controladores inactivos en las listas de selección y que la proyección omita huérfanas con aviso en vez de bloquear todo). *Plan de acción:* confirmar el origen, pedir la aprobación y presentar las tres mejoras con su justificación. *Resultado esperado:* inventario de calle limpio y un fallo de proyección que deje rastro visible.

**5. Blindaje del backend.** *Justificación:* La protección del borrador está solo en la interfaz, si otro cliente llama al backend directamente, aún podría sobrescribirlo. *Plan de acción:* corregirlas en orden de impacto y blindar el backend antes de que otra pantalla use esa lectura. *Resultado esperado:* Sincronía sin falsos avisos y con el borrador protegido también del lado del servidor.

### 3.3 Actividades cercanas

Según el cronograma, la Semana 8 repite las mismas Actividades 5, 6 y 8. Lo concreto para la semana es dar seguimiento a los pendientes de arriba y continuar la validación de integración con controladores conforme llegue el hardware.

---

## 4. Autorreflexión personal: problemas enfrentados, aprendizajes y desempeño

### 4.1 El patrón que más se repitió esta semana

La primera explicación de un problema casi nunca fue la completa. El primer diagnóstico de cámaras (controladores con identificadores nuevos y soft delete) era correcto pero insuficiente, corregir solo eso no hacía que llegaran a la ventana de detección, y las causas restantes solo aparecieron al volver a ejecutar la proyección y leer el error real. Pasó igual con el croquis, donde arreglar el cruce de datos destapó que el borrador de ROI no estaba enviado al controlador, y después un parpadeo. En los tres casos, lo que avanzó fue probar la corrección contra el dato real y seguir hasta que el síntoma desapareciera, no detenerse en la primera causa.

### 4.2 Errores propios y lo que aprendí de cada uno

En el diagnóstico de cámaras se actualizaron llaves de un mapa interno que resultó no influir en el comportamiento. No causó daño, pero se hizo sin confirmar antes qué leía el código. Además, el respaldo con `pg_dump` quedó confirmado antes del borrado, pero no se confirmó uno previo a los primeros `UPDATE`, y el que existe está en una carpeta temporal que se limpia al reiniciar. El aprendizaje es respaldar antes de cualquier escritura, no solo antes de las destructivas, y guardar el respaldo fuera de un directorio temporal. Por último, el parche del croquis resolvió el parpadeo pero eliminó una expiración que ocurría por accidente; fue la revisión posterior, y no la primera lectura del código, la que lo detectó, y quedó registrado como pendiente.

### 4.3 Aprendizajes profesionales y consideraciones éticas

- Se evitó actuar sobre lo que correspondía a otras personas o a equipos de calle. No se borró la configuración que el controlador vuelve a subir, y los cambios a la lógica de salud de cámara y a la configuración de las cámaras de calle se dejaron como propuestas para aprobación.
- Se probó con el controlador de laboratorio y con una simulación de desconexión reversible, usando el controlador de calle solo como identidad de texto, para no afectar la operación real.
- El hallazgo de la PC-10 sobre el borrador perdido se trató como bloqueante y se corrigió antes de continuar, en vez de documentarlo como observación menor.

### 4.4 Sobre el cronograma

Esta semana el cronograma coincidió con el trabajo real en las Actividades 5, 6 y 8, con la salvedad de que la 8 avanzó en validación y no en optimización por depender de hardware que aún no está instalado. El seguimiento a incidencias (Actividad 7) volvió a ocupar parte de la semana aunque el cronograma lo programó hasta la semana 5.

### 4.5 Desempeño, bienestar y acciones que voy a mantener

La semana combinó cuatro frentes en cinco días. Lo que ayudó a sostener el ritmo fue cerrar cada frente con un reporte escrito y dejar explícito qué quedaba pendiente y de quién dependía, lo que evitó cargar mentalmente puntos abiertos.

Acciones que se van a sostener:

- Respaldar antes de cualquier escritura en datos compartidos y guardar el respaldo fuera de un directorio temporal.
- Ensayar en una copia antes de aplicar cambios a producción, y registrar qué quedó sin verificar.
- Confirmar qué lee el código antes de modificar un dato, y revisar los efectos secundarios de un parche después de aplicarlo.
- Escalar con la propuesta documentada todo cambio que toque equipos de calle o decisiones de diseño de otra persona.
