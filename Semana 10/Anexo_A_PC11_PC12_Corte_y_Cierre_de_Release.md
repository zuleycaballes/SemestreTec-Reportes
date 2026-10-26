Anexo A - Reporte de avance y cierre: corte controlado (PC-11) y cierre del release (PC-12) en laboratorio

Fecha del cierre: 19 de octubre de 2026
Origen: documentos internos de avance del corte controlado y de cierre del release, entregados al responsable del proyecto.

> Versión con datos omitidos. Este anexo reproduce el contenido técnico de los reportes internos omitiendo direcciones de red, nombres de servidores, identificadores de nodo y de controlador, versiones y huellas de código, huellas (hash) de configuración, nombres de archivos, rutas y comandos internos, y nombres de personas ajenas a la residencia, conforme al acuerdo de confidencialidad. Lo que se conserva es el problema, el razonamiento, los hallazgos y las verificaciones hechas.

---

## Resumen y decisión

La versión nueva del firmware (canario) se desplegó en el controlador de laboratorio, pasó sus validaciones y el rollback a la versión anterior se ensayó con éxito, con regreso a la versión nueva sin perder identidad ni configuración. El respaldo de la base de datos es restaurable. La PC-09 se repitió sobre el canario y sobre el release activo del sistema central y quedó aprobada. La PC-10 quedó aprobada con un parcial aceptado, y la PC-11 quedó cerrada en laboratorio con una excepción documentada.

**Decisión: promover en el controlador de laboratorio y observar en el controlador de calle y en cualquier equipo en campo.** Con eso quedó desbloqueado el plan de configuración por controlador, solo para laboratorio.

```text
PC-09: APROBADA (laboratorio, dos versiones del firmware, 15 y 19 de octubre)
PC-10: APROBADA CON PARCIAL (dos pasos esperan un nodo con varios controladores)
PC-11: APROBADA EN LABORATORIO CON EXCEPCIONES
Rollback disponible: firmware ensayado el 19 de octubre; sistema central, release anterior conservado y sin ensayo
Decisión: PROMOVER (laboratorio) / OBSERVAR (calle)
```

## Alcance

Toda la evidencia de PC-09, PC-10 y PC-11 se ejecutó en el controlador de laboratorio. El controlador de calle solo se leyó, sigue en la versión anterior, no ha recibido ningún envío de configuración desde el sistema central, y se respaldó su identidad y configuración como requisito previo. Esta conclusión deja de valer en cuanto algo de esta versión se active en un equipo en campo.

## Cronología del canario y del rollback (19 de octubre, hora local)

| Hora | Paso | Resultado |
| --- | --- | --- |
| 13:24 | Respaldo previo al canario | Nueve archivos de identidad y configuración, con sumas de verificación |
| ~13:27 | Subir y activar el canario desde el panel de actualización | Paquete de 320 archivos; cambian solo los servicios de control de luces y de visión |
| ~13:31 | Validación posterior | Chequeo de salud "sano", cinco servicios activos, contrato de tiempos correcto, configuración en la versión 1 con la misma huella en dos lecturas e igual al respaldo, registro legible |
| 13:33 | Rollback a la versión anterior | Sano en la versión anterior; identidad, referencia a la instancia retirada, salud y huella de la configuración iguales |
| ~13:35 | Activar de nuevo el canario | Activación exitosa; sin advertencias en la bitácora del sistema |
| 13:49 | Respaldo de la base de datos de producción | Restaurable; coincide con producción en controladores y migraciones |
| 14:03 | PC-09 sobre el canario | Aprobada |
| 14:13 | Despliegue del sistema central | Release activo nuevo |
| ~15:10 | PC-09 sobre el release activo | Rechazada, 48 de 51 (ver más abajo) |
| ~15:15 | PC-09 repetida | Aprobada, 57 de 57 |
| 15:23 | Envío de configuración de prueba desde Sincronía, modo sin destello | Correcto; el sistema central y el controlador quedaron en sincronía |
| 15:25 | Cierre del release | Decisión de promover en laboratorio y observar en calle |

## Estado contra el checklist del corte

| Punto | Estado | Detalle |
| --- | --- | --- |
| Respaldo restaurable de la base | Hecho | Restaurado en una base temporal, con las migraciones aplicadas iguales a producción; la temporal se borró |
| Paquete del canario y sumas de verificación | Hecho | Armado con el empaquetador oficial |
| Responsables técnico y vial | Hecho, sin firma | Confirmación comunicada de forma verbal, sin documento |
| Detener escrituras de Sincronía | Excepción | No se detuvieron formalmente; el canario fue un solo controlador de laboratorio con un solo operador |
| Backend y frontend del sistema central | Hecho | Parche de la referencia a la instancia retirada verificado en la base; frontend desplegado el mismo día |
| Un solo canario | Hecho | Laboratorio en la versión nueva; el de calle sin cambio |
| Chequeo de salud | Hecho, con límite | Reporta "sano" tras activar y tras el rollback, pero solo revisa el código HTTP |
| Contrato, versión y huellas | Hecho | Iguales antes, durante y después |
| Smoke del sistema central al CSI y de regreso | Hecho | Enrolamiento, sincronización de identidad y envío de prueba |
| Ventana de observación | Parcial | Unos 6 minutos sin advertencias |
| Conservar el release anterior | Hecho | Firmware y sistema central |
| Respaldo de la configuración de los controladores | Hecho | Laboratorio y calle, más el del canario |
| Ensayo del rollback | Hecho | Con regreso a la versión nueva, sin pérdida de identidad ni de configuración |

## Hallazgos

| Hallazgo | Impacto | Estado |
| --- | --- | --- |
| Una versión candidata anterior tenía cientos de pruebas fallando, clasificadas por el equipo como expectativas viejas y un defecto real, ya corregido; en la rama principal nueva el conjunto de pruebas pasa completo | No activar en calle sin poner en 0 los ajustes de rojo de 1 s: compilaría un ciclo de 62 s y no de 60 s | Resuelto en pruebas; falta validación física, ventana autorizada y paquete definitivo |
| Certificado del sistema central autofirmado y sin nombre alternativo | El enrolamiento depende de una solución provisional de laboratorio | Runbook y script probados en un entorno de pruebas, autorizados, sin ejecutar |
| Se perdió la línea del archivo de nombres que apunta al sistema central en el controlador de laboratorio | Dos advertencias en la bitácora | Resuelto con una línea provisional |
| El sistema central ignoraba la referencia a la instancia retirada que devuelve el controlador | La fila activa no enlazaba a la retirada | Resuelto con un parche desplegado |
| El chequeo de salud solo revisa el código HTTP | Reporta "sano" aunque el cuerpo diga lo contrario | Abierto |
| El controlador de laboratorio llegó a reportar fallo en las escrituras de luces | Parecía falla de software y era del equipo | Hoy funciona; falta confirmar qué cambió |
| La evidencia de PC-09 declaraba una versión del sistema central distinta a la que corría | La evidencia no cubría el código en producción | Resuelto repitiendo PC-09 sobre el release activo |
| El primer intento de PC-09 sobre el release activo falló en 3 de 51 verificaciones | Las tres inspecciones del sistema central no tuvieron respuesta, probablemente por el certificado interno | Repetido con la verificación del certificado desactivada, aprobado 57 de 57; deuda de seguridad menor |

## Aprobaciones

| Rol | Decisión | Forma |
| --- | --- | --- |
| Sistema central y CSI | Aprobado | Declarado por escrito en el chat de trabajo |
| Decisión sobre las rutas de traducción temporales | Aprobada por el líder técnico del sistema central | Declarada por escrito en el chat de trabajo |
| Seguridad vial (laboratorio) | Acuerdo verbal | Sin documento |
| Responsable técnico y vial | Confirmación escrita recibida | Falta anexar la captura |

## Excepciones y pendientes

- Observación del canario de unos 6 minutos; ampliarla si el responsable lo pide.
- Pasos de PC-10 que esperan un nodo con varios controladores, autorizados por la jefatura.
- Revisión del parche del compilador de tiempos por su dueño, antes de activar en calle.
- Rollback del sistema central sin ensayo.
- Once discrepancias de documentación por corregir, que no bloquean el cierre.
- Fecha límite global propuesta para firmas y pendientes de laboratorio: 30 de octubre (propuesta, no acordada).

## Cómo revertir

Ningún cambio tocó la configuración local del controlador ni su identidad. Los cambios provisionales se deshacen así:

| Cambio | Reversión |
| --- | --- |
| Línea provisional del archivo de nombres | Restaurar la copia hecha justo antes del cambio |
| Certificado nuevo del sistema central, cuando se ejecute | Ejecutar el rollback del script, que verifica la suma de verificación del respaldo y recarga el servidor web |
| Autoridad certificadora instalada en un controlador, cuando se reparta | Quitarla de la lista del sistema y refrescar |
| Parche de la referencia a la instancia retirada | Revertir el cambio; el valor ya guardado es inofensivo |
| Firmware del controlador de laboratorio | Rollback a la versión anterior, ya ensayado |
