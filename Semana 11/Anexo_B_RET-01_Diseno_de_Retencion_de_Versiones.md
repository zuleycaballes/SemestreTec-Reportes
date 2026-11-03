Anexo B - Reporte de cierre y dictamen: RET-01, investigación y confirmación del diseño de la retención automática de versiones del firmware

Fecha del cierre: 28 de octubre de 2026
Origen: documentos internos de dictamen de diseño y de cierre, entregados al responsable del proyecto.

> Versión con datos omitidos. Este anexo reproduce el contenido técnico de los reportes internos omitiendo direcciones de red, puertos, nombres de servidores, de servicios y de equipos, rutas, versiones, huellas (hash), nombres de archivos y de herramientas internas, y nombres de personas ajenas a la residencia, conforme al acuerdo de confidencialidad. Los dos controladores revisados se identifican como equipo A y equipo B. Lo que se conserva es el problema, el razonamiento, los hallazgos y las verificaciones hechas.

---

## Resumen y decisión

**RET-01 cerrado.** El diseño de la retención automática de versiones quedó revisado y confirmado con correcciones, y la aprobación del responsable del proyecto se registró por escrito en la tarea el 28 de octubre. No se implementó, reinició ni desplegó nada: en los equipos y en el servidor solo se hicieron lecturas.

El objetivo de la serie es que el CSI respalde en el sistema central las versiones del firmware que dejan de ser necesarias y luego las elimine de su almacenamiento, sin retirar nunca una que todavía haga falta. Se divide en RET-01 (confirmar el diseño y registrar la aprobación), RET-02 (implementar y probar sin desplegar) y RET-03 (instalar, con autorización propia, primero en el equipo A y después, con aprobación aparte, en el equipo B).

```text
Diseño: CONSERVADO CON 8 CORRECCIONES (C1 a C8)
Aprobación: 28 de octubre de 2026, registrada en la tarea
Implementado o desplegado: NADA
Siguiente fase: RET-02 (implementar y probar sin desplegar)
```

## Verificaciones antes de trabajar

| Verificación | Resultado |
| --- | --- |
| Ramas de origen | Se revisaron los paquetes de la rama principal de los dos repositorios. El diseño se escribió sobre una rama que, según el equipo, ya está integrada a la principal |
| Código instalado | **No coincide con la rama principal.** Los dos equipos difieren del repositorio y entre sí |
| Cambios paralelos | Confirmado por el equipo: no hay trabajo paralelo sobre el actualizador |
| Limpieza manual | No tenía bloqueo, pausaba el actualizador y copiaba por SSH. Un criterio de "solo la actual y la anterior" habría borrado una versión en uso en el equipo B |
| Dónde se instala la extensión | En una ubicación que el paquete normal de actualización no cubre, con un procedimiento explícito. El archivo de sumas de verificación de la rama principal está desactualizado y hay que regenerarlo |

## Investigación de implementaciones comparables

Con código fuente y manuales oficiales se compararon cuatro ejes en OSTree, git-annex, dnf y el script de despliegue del sistema central, que se usó como contraejemplo.

| Eje | Qué hacen | Qué se adopta |
| --- | --- | --- |
| Protección de versiones | OSTree conserva la arrancada, la de fusión, las fijadas y las pendientes; dnf nunca elimina el kernel en ejecución. El despliegue del sistema central conserva 5 por fecha de modificación | Separar lo que corre de lo que está instalado, y pines. **No** usar conteo ni fechas |
| Exclusión mutua | OSTree usa un bloqueo exclusivo con una variante que espera y otra que solo intenta | La operación principal espera y la limpieza solo intenta |
| Confirmación del respaldo | git-annex no borra si no puede verificar que otro repositorio tiene el contenido | Verificar de nuevo justo antes de borrar, sin opción de forzar. Si el sistema central no responde, se conserva |
| Recuperación ante fallos | OSTree escribe el nuevo conjunto y limpia después; git-annex asegura un mínimo de copias | Guardar intención y comprobante antes de borrar y garantizar al menos una copia |

Mender y SWUpdate no se usaron como comparables porque trabajan con particiones A/B y no retienen N versiones.

## Correcciones al diseño

Se conserva del diseño el motor junto al actualizador sin tocar el control de luces, los conjuntos protegidos, los estados por trabajo, el manifiesto con huella y comprobante, la idempotencia, la base de datos propia, el bloqueo único, la simulación primero y la retención indefinida en el sistema central.

| # | Corrección | Motivo |
| --- | --- | --- |
| C1 | Una versión está en uso según el directorio de trabajo real de cada proceso, no por su línea de comando. Un proceso operativo sin atribuir bloquea el borrado | Borrar una versión en uso no tumba el proceso: **mezcla versiones en silencio** |
| C2 | "Pendiente" lo define la bitácora, no un archivo de preparación | Ese archivo nunca se borra y protegería todo para siempre |
| C3 | La espera de 24 h cuenta desde el último cambio observado de actual o anterior | El rollback manual no queda en la bitácora |
| C4 | No se reutilizan números de versión retirados | Hoy solo lo impide que exista el directorio |
| C5 | El actualizador espera el bloqueo con tope y reintenta; la retención solo intenta | Hoy un rechazo marca una activación programada como fallida |
| C6 | El CSI sube su respaldo por el canal seguro existente, con credencial por equipo | El servicio del actualizador no tiene autenticación; desaparece el proceso de fondo en el sistema central |
| C7 | Justo antes de borrar se verifica de nuevo que la copia sigue en el sistema central, fuera del bloqueo | Detecta que el almacén se perdió o se restauró después de confirmar |
| C8 | Se distinguen por huella una subida fallida, una versión dañada y una con fallas | Condición de la aprobación |

**Clasificación (C8).** Una subida fallida se detecta cuando el sistema central recalcula SHA-256 y tamaño y no coinciden con el manifiesto: se rechaza, el CSI reintenta y la versión no cambia ni se borra. Una versión dañada se detecta contra las sumas de instalación del paquete: queda "divergente", se archiva como evidencia y **no se borra en automático**. Una versión con fallas (activación fallida, revertida o salud en 0) se identifica por la bitácora: se archiva y retira con normalidad, pero nunca se elige como recuperación.

**Garantías de no bloqueo.** Ningún servicio operativo toma el bloqueo y la retención no reinicia ni detiene procesos; corre con prioridad mínima de CPU y disco, y comprime por flujo. Solo toma el bloqueo mientras borra una versión, con un tope duro de 30 s. La activación espera hasta 120 s y, si se vence, se reintenta en el siguiente ciclo de 5 min. La retención no arranca si hay una activación programada en los siguientes 60 min. Con un solo bloqueo no hay interbloqueo, y es un bloqueo del kernel que se libera solo si el proceso muere.

## Evidencia de los equipos

- **Hallazgo principal.** En el equipo B, un servicio auxiliar llevaba 12 días ejecutando código de una versión antigua, y no está en la lista de servicios que reinicia el actualizador. La regla simple de actual y anterior la habría borrado en uso, lo que confirma que la protección por proceso hace falta siempre.
- **Simulación esperada.** En el equipo B solo dos versiones antiguas serían candidatas; en el equipo A ninguna, hasta que el laboratorio haga dos activaciones más.
- **Espacio.** Cada versión retirada libera de 4 a 24 MB: la retención evita la acumulación, no recupera mucho espacio. El sistema central tiene más de 300 GB libres en un único disco.
- **Actualizador instalado contra repositorio.** De nueve archivos comparados por hash, cinco difieren en al menos un equipo. Los equipos A y B **no tienen el mismo actualizador** (difieren en cuatro archivos y uno falta en el B). RET-02 parte de lo instalado y no de la rama principal, y la primera instalación debe unificar los dos equipos.

## Condiciones que permitirían retirar una versión necesaria

Se identificaron 12 condiciones, cada una con su mitigación: proteger solo actual y anterior, atribuir procesos por línea de comando, procesos con directorio fuera de la versión, escaneo sin permisos, rollback manual sin registro, activación fallida que iguala anterior y actual, nombres reutilizados, un proceso que arranca entre la planificación y el borrado, comprobante válido pero almacén perdido, versión reescrita después de archivarse, preparación olvidada y versión dañada antes de archivarse. Las cubren las correcciones C1 a C8 y la revalidación con el bloqueo tomado.

## Hallazgos fuera de alcance, no atendidos

| Hallazgo | Severidad |
| --- | --- |
| El servicio de actualización no tiene autenticación en las rutas que activan, revierten, programan y preparan | Alta |
| Un servicio auxiliar fuera de la tabla del actualizador con código antiguo; rollback manual sin registro; cancelación que acepta corridas "activando"; preparación forzada que sobrescribe una versión posiblemente en uso | Media |
| Sumas de verificación desactualizadas (fallan 2 subpruebas); archivo de preparación que nunca se retira | Baja |

## Criterios de aceptación

**RET-02 (implementar y probar sin desplegar).** Con el bloqueo tomado por una activación, la retención no borra, y viceversa, sin dejar una activación programada como fallida. Ninguna versión se borra sin comprobante persistido, árbol igual al manifiesto, verificación remota en ese momento y escaneo sin referencias. En la simulación del equipo B, la versión usada por el servicio auxiliar sale protegida y solo salen candidatas las dos esperadas. Los tres casos de C8 deben ser distinguibles sin intervención manual, no debe haber regresiones (hoy 30 pruebas pasan y 2 subpruebas fallan) y ningún proceso espera más que el tope.

**RET-03 (instalar con autorización).** Antes y después en el equipo A deben coincidir los procesos operativos, el hash de la configuración, los destinos de los enlaces y los conteos de la bitácora, sin reinicios causados por la retención; el equipo B solo con aprobación aparte y en una ventana acordada.

## Decisiones y registro de aprobación

| Decisión | Resultado | Fecha |
| --- | --- | --- |
| Diseño ajustado, con política inicial de 24 h, revisión cada 6 h, una versión a la vez y hasta 3 por pasada | **Aprobado con condición:** distinguir por huella los tres casos de C8. Atendido | 28 de octubre |
| El CSI sube su respaldo con credencial propia por equipo | **Aprobado.** La doble verificación (C7) se evaluó y se hace siempre | 28 de octubre |
| La activación programada espera y se reintenta durante una limpieza | **Aprobado con condición:** que no se atore ningún proceso. Atendido; incluye el visto bueno vial | 28 de octubre |
| El sistema central como única copia del almacén | **Se acepta por ahora**, con pendiente documentado de almacenamiento segmentado | 28 de octubre |

La doble verificación aporta poco si el borrado ocurre segundos después de confirmar, pero es la única forma de detectar un almacén perdido o restaurado cuando el borrado se retrasa (versión en uso, bloqueo ocupado, reinicio o sistema central caído). Cuesta una consulta por versión, así que se hace siempre.

## Continuidad

- **Pendiente documentado:** mientras no haya discos nuevos, el almacén vive en el disco único de la máquina virtual del sistema central. Con prioridad: moverlo a un volumen nuevo cambiando solo la ruta, respaldarlo fuera de la máquina virtual y definir respaldo y rotación de los registros del actualizador en los CSI.
- **Para RET-02:** copia de solo lectura del actualizador instalado en cada equipo y la diferencia del módulo de enlace con el microcontrolador entre la versión antigua y la nueva en el equipo B (riesgo fuera de alcance). El dictamen y el cierre se publican en la tarea y no se suben al repositorio; solo se versiona el código de las herramientas operativas.
