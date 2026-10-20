Anexo A - Reporte de incidente: migraciones a producción y despliegue fallido

Fecha del cierre: 13 de octubre de 2026
Origen: documento interno de reporte de incidente, entregado al responsable del proyecto.

> Versión con datos omitidos. Este anexo reproduce el contenido técnico del reporte interno omitiendo nombres de servidores, de bases de datos y de archivos, identificadores de controlador, versiones y huellas de código, y rutas internas, conforme al acuerdo de confidencialidad. Lo que se conserva es el problema, el razonamiento, los hallazgos y las verificaciones hechas.

---

## Resumen

Se aplicaron seis migraciones a la base de producción y el registro quedó en 41 aplicadas y 0 pendientes. El primer despliegue de la rama principal falló en la verificación de salud y se revirtió automáticamente al release del 2 de octubre. La causa fue un error de importación por un archivo mal nombrado en la rama principal, y no las migraciones. El fix se mergeó y, con posterioridad al cierre del reporte interno, el despliegue posterior al fix se verificó completo (ver "Actualización posterior al cierre").

## Cronología (hora local, UTC-07:00)

| Hora | Evento |
| --- | --- |
| 13:14 | Respaldo de la base de producción (350 KB), verificado listando su contenido con `pg_restore` |
| 13:14 a 13:17 | Copia de ensayo en una base temporal. La primera restauración falló por permisos de un directorio y se repitió por entrada estándar. La copia quedó con 35 migraciones aplicadas y 0 pendientes |
| 13:23 | Clon de la rama principal en un directorio temporal. Revisión del SQL: sin `DROP TABLE` ni `DELETE`; solo la eliminación de una vista (una migración), tablas temporales y quitar el valor por defecto de una columna de identidad de controlador (dos migraciones) |
| 13:24 | Ensayo en la copia: seis migraciones aplicadas, 41 aplicadas, 0 pendientes, 0 divergentes |
| 13:25 | Segundo respaldo. Aplicación en producción: seis migraciones sin error (entre 14 y 54 ms cada una). Registro: 41 aplicadas, 0 pendientes |
| 13:25 | El despliegue de la rama principal crea el release nuevo. Pasos 1 a 9 correctos; el paso 10 (verificación de salud) falla 10 de 10 intentos y ejecuta la reversión al release del 2 de octubre, que termina bien |
| 13:30 | En el registro del sistema, los procesos del release nuevo caen al arrancar con un error de importación |
| 13:30 a 13:36 | Con el release anterior sirviendo sobre el esquema 41, aparece un error de atributo inexistente al leer la configuración de dos controladores: 24 apariciones hasta las 13:36 y ninguna antes de las 13:25 |
| 13:36 a 13:46 | Diagnóstico: la rama principal contiene el archivo con un sufijo de más en el nombre, y dos módulos (de auditoría y de detección de eventos) lo importan sin ese sufijo. El mismo nombre está en las otras ramas de desarrollo |
| 13:59 | Fix mergeado a la rama principal (reportado por el desarrollador) |

## Causas

1. **Despliegue fallido.** El archivo del servicio de contratos de datos no coincide con el nombre que importan los dos módulos mencionados. El backend no arranca.
2. **Error de atributo.** El servicio de configuración de controladores del release del 2 de octubre usa un campo de identidad que el modelo de controlador de ese release no define. El release nuevo sí lo define. El motivo exacto de que el error aparezca solo después de las 13:25 no se determinó.
3. **Esquema adelantado al código.** El script de despliegue no aplica migraciones por diseño, así que el flujo fue migrar y luego desplegar. Al fallar el despliegue, quedó el código viejo sobre el esquema nuevo.

## Criterios

| Criterio | Estado | Evidencia |
| --- | --- | --- |
| Respaldo previo a migrar | Cumplido | Dos respaldos de la base de producción en la propia máquina |
| Ensayo en copia restaurada | Cumplido | De 35 a 41 aplicadas, 0 divergentes, 0 fallidas |
| Migraciones aplicadas en producción | Cumplido | Registro de producción: 41 aplicadas, 0 pendientes |
| Despliegue de la rama principal en producción | No cumplido en el intento de las 13:25 | Reversión automática por falla de verificación de salud |
| Fix mergeado en la rama principal | Cumplido | Reportado en el cierre; verificado después (ver actualización) |
| Despliegue posterior al fix | Cumplido | Sin verificar al cierre del reporte interno; verificado después, con salud y disponibilidad en verde |
| Lectura de configuración de los dos controladores | Falló con el código anterior; resuelta con el release nuevo | Error de atributo en el registro del sistema con el código anterior; verificada después la lectura sin el error |

## Hallazgos

- El chequeo de disponibilidad solo exige migraciones hasta la 35, así que seguía en verde con el esquema en 41 y con el error de atributo activo.
- El ensayo cubrió solo el SQL. No hubo prueba de importación del código nuevo antes del despliegue.
- El paso 7 del script de despliegue listó las seis migraciones como nuevas aun con el registro en 41; es un aviso y no un error.
- La revisión de operaciones `DROP` no mostró operaciones destructivas sobre datos. No se comparó el contenido de las tablas antes y después más allá del estado del runner de migraciones.

## Continuidad

- Producción: la base en el esquema 41. Tras la reversión, el release activo era el del 2 de octubre; después del fix, el despliegue posterior se completó y quedó verificado.
- Respaldos: los dos respaldos están en la propia máquina y siguen sin copia fuera de ella.
- Residuos a limpiar cuando la ventana esté cerrada: la base de ensayo, tres directorios temporales y el release fallido.
- Verificaciones que estaban pendientes al cierre y que ya se cumplieron: la rama principal trae el archivo con el nombre correcto; el despliegue posterior terminó con salud y disponibilidad en verde; los dos controladores con error leen su configuración sin el error de atributo.
- Mejoras identificadas, sin implementar: que el chequeo de disponibilidad considere el registro de migraciones completo, y una prueba de importación del backend nuevo antes de activar el release.

## Actualización posterior al cierre

Después de entregado el reporte interno, se verificó el despliegue posterior al fix. Se comprobó la rama principal, el resultado del despliegue con salud y disponibilidad en verde, y la lectura de configuración de los dos controladores sin el error de atributo. Con eso se cierran los tres puntos que el reporte interno dejaba sin verificar. Siguen abiertos los respaldos sin copia externa, los residuos por limpiar y las dos mejoras sin implementar.
