Anexo A - Plan de ejecución: corte controlado y rollback (PC-11)

Fecha del documento: 5 de octubre de 2026
Origen: documento interno de preparación, entregado al responsable del proyecto.

> Versión con datos omitidos. Este anexo reproduce el contenido técnico del plan interno omitiendo direcciones de red, puertos, identificadores de nodo y de controlador, versiones y huellas (hash) de despliegue, números de migración, nombres de archivos y rutas internas, conforme al acuerdo de confidencialidad. Lo que se conserva es el problema, el razonamiento, los hallazgos y las verificaciones previstas.

---

## Objetivo

Convertir el checklist de la PC-11 (desplegar el parche de forma controlada, validar el controlador canario y demostrar que el sistema puede recuperarse) en un procedimiento ejecutable, con fases, criterios de paso, condiciones de reversa y evidencia requerida, contrastándolo antes contra el estado real de los repositorios y de los scripts de despliegue.

## Estado

**No ejecutada.** Este documento prepara la ventana; no sustituye la evidencia de una corrida real. El corte solo puede hacerse en la ventana autorizada, con los responsables presentes.

## Punto de partida

Un checklist de 18 puntos (respaldo restaurable, registro de paquetes y huellas, responsables técnico y vial, detener escrituras de Sincronía, migración antes del backend, despliegue de backend y frontend, un solo controlador canario, verificación de salud y de identidad, prueba de punta a punta, observación y conservación de las versiones anteriores), 7 disparadores de reversa y 4 criterios de terminado.

## Discrepancias entre el checklist y la realidad

| # | Hallazgo | Acción |
|---|---|---|
| 1 | El checklist indica aplicar una migración de base de datos por su número, pero esa migración ya estaba aplicada en producción según la auditoría documentada del 10 de septiembre. | No aplicar por número. Obtener la lista real de pendientes con la herramienta de migraciones, en modo de solo lectura, contra producción. |
| 2 | El repositorio contiene migraciones posteriores a las del plan original y varias no son aditivas: eliminan una restricción, eliminan datos sembrados y eliminan un índice. | Revertir la aplicación no revierte el esquema. Se arranca la versión anterior de la aplicación contra una copia restaurada con el esquema nuevo para saber si es compatible; el rollback de datos es restaurar el respaldo tomado justo antes del corte. |
| 3 | La revisión de salud del controlador valida servicios, zona horaria, estado de los módulos de iluminación y supervisión, interfaz y versión. No compara la identidad del controlador, ni sus metadatos, ni la configuración vigente, ni su registro. | Esas comprobaciones del checklist se hacen manualmente. Que la revisión de salud pase no prueba la identidad ni el formato de configuración. |
| 4 | El flujo de identidad cambió: ya no se crea un archivo de identidad a mano; la identidad se sincroniza desde el panel y se verifica consultándola en el propio equipo. Algunos pasos del plan original son históricos. | Usar el flujo nuevo y confirmar que las ramas que lo implementan están en la versión que se despliega. |
| 5 | El checklist pide a la vez un controlador canario y ensayar el rollback en el controlador de laboratorio, pero existe un solo controlador de laboratorio y uno de calle. | Decidir y anotar cuál es el canario. Si es el de laboratorio, el ensayo ocurre en el mismo equipo y el de calle sigue con la versión anterior. Si es el de calle, el ensayo previo en laboratorio es obligatorio y distinto. |

Sin cerrar los puntos 1, 2 y 5 no se abre la ventana.

## Estructura del plan

**Hoja de ejecución.** Se llena antes de empezar, identificador de la versión, huellas de los dos repositorios con árbol de trabajo limpio, controlador canario, ubicación y huella del respaldo, ventana autorizada, responsables de despliegue, del controlador, del operador de Sincronía y de seguridad vial (quien autoriza cualquier envío o reinicio que afecte luces), aprobador y decisor del rollback con su tiempo máximo de decisión.

**Fase A, preparación previa a la ventana.** Pruebas y compilación sobre los mismos artefactos que se van a desplegar; sumas de verificación de cada paquete; respaldo completo y prueba de que es restaurable, siempre en una base temporal y nunca sobre producción; migraciones ensayadas en esa copia con conteos antes y después; arranque de la versión anterior contra la copia ya migrada; respaldo de la configuración del canario; confirmación de que la versión anterior del sistema central y la del controlador se conservan y no serán limpiadas; confirmación de los responsables. Si falla la restauración, las migraciones en copia o el arranque de la versión anterior, no se abre la ventana.

**Fase B, corte del sistema central.** Confirmar permiso vial y responsables; detener las escrituras de Sincronía; tomar un respaldo final inmediato, que es el punto de restauración real; aplicar las migraciones antes del backend; activar la versión nueva con reversión automática si falla la verificación de salud; validar backend y frontend; reanudar las escrituras solo si ambos pasaron.

**Fase C, controlador canario (solo uno).** Los demás controladores siguen con la versión anterior hasta cerrar la observación. Se activa la versión nueva en el canario, se corre la revisión de salud y se comprueban a mano la identidad, los metadatos, la configuración vigente (leída dos veces, con la misma huella) y el registro. La identidad debe coincidir en cuatro fuentes, archivo del controlador, inventario del sistema central, estado de salud y metadatos. Cualquier diferencia activa la reversa.

**Fase D, prueba de punta a punta y observación.** Validación de una configuración de prueba sin aplicarla a las luces; envío real solo con el responsable vial presente; relectura y comparación de huellas; verificación de que el registro de envíos solo cambió para el controlador canario; confirmación de que toda entrada se persiste en el formato vigente; ejecución de las pruebas PC-09 y PC-10 sobre el canario; y una ventana de observación con revisiones periódicas de salud, huellas y registros de los servicios.

## Condiciones de reversa

Cualquiera de estas activa la decisión de revertir, falla la salud de los servicios principales; la identidad no coincide entre las cuatro fuentes; la configuración vigente deja de ser válida; el ciclo de ida y vuelta modifica la semántica o la huella; un envío al controlador A modifica al B; se pierde configuración o quedan pendientes ilegibles; no puede comprobarse un estado físico seguro de las luces. Se conserva toda la evidencia del fallo y no se intenta un segundo despliegue hasta explicar la causa.

## Reversa por componente

- **Controlador.** Regreso a la versión anterior con el mecanismo de actualización, revisión de salud en esa versión y comparación de la configuración contra el respaldo previo. El script de pruebas de cierre no se ejecuta en calle, porque uno de sus escenarios rompe el módulo de iluminación a propósito.
- **Aplicación del sistema central.** Se vuelve a enlazar la versión anterior y se reinicia, lo mismo que hace el despliegue automático cuando falla su verificación.
- **Datos.** Restauración del respaldo final de la Fase B, con ventana controlada. Se pierden las escrituras posteriores al respaldo, y por eso las escrituras de Sincronía están detenidas desde el inicio del corte. No se elimina la tabla de configuraciones por controlador como parte de esta ventana.

## Ensayo obligatorio en laboratorio

Con el cruce fuera de servicio, activar la versión nueva en el controlador de laboratorio y revertirla; confirmar que la revisión de salud queda sana y que la configuración está intacta; repetir el rollback de la aplicación contra la base restaurada; y registrar los tiempos de cada rollback, que alimentan el tiempo máximo de decisión.

## Criterios de terminado

| Criterio | Estado |
|---|---|
| El canario completa PC-09 y PC-10 | Pendiente |
| Salud, registros y huellas estables durante la observación | Pendiente |
| Rollback ensayado en laboratorio, con tiempos | Pendiente |
| Versión anterior del sistema central y del controlador conservadas y protegidas | Pendiente |
| Los demás controladores se pueden desplegar de uno en uno sin perder configuración | Pendiente |

## Continuidad

- Llenar la hoja de ejecución, huellas, identidad del canario, responsables y ventana.
- Correr la fase A en copia restaurada y traer la lista real de migraciones pendientes; con ella se ajustan el plan y la plantilla de cierre.
- El estado de la PC-11 se mantiene como **no ejecutada** hasta contar con una corrida real; la plantilla de cierre (estado, huellas, migraciones aplicadas, canario, rollback ensayado, incidentes, decisión y aprobadores) no se llena antes.

## Evidencia

El plan, con sus comandos y salidas esperadas, está en el documento interno. No hay evidencia de ejecución porque el corte no se ha ejecutado.
