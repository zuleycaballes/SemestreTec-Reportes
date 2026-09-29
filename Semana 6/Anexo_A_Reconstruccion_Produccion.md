Anexo A - Reporte de cierre: reconstrucción del ambiente de producción tras un reinicio de fábrica del servidor

Fecha del cierre: 22 de septiembre de 2026
Origen: documento interno de cierre, entregado al responsable del proyecto.

> Versión con datos omitidos. Este anexo reproduce el contenido técnico del reporte interno omitiendo direcciones de red, nombres de host, rutas internas, huellas de certificados y detalles de configuración de los equipos, conforme al acuerdo de confidencialidad. Lo que se conserva es el problema, el razonamiento, las decisiones y las verificaciones hechas.

---

## Objetivo

Reconstruir desde cero el ambiente de producción del sistema, después de que el servidor se reiniciara de fábrica y migrara, sin que nadie lo planeara, a una máquina distinta a la original.

## Diagnóstico inicial

Los primeros síntomas parecían un reinicio ordinario, no un cambio de máquina. Tres señales lo aclararon, el gestor de servicios del sistema operativo no reconocía ninguna de las unidades ya configuradas, la versión del sistema operativo no coincidía con la documentada, y una consulta de identificación de hardware reportó un tipo de equipo distinto (máquina virtual en vez de equipo físico).

## Alcance de la pérdida

Todo lo que no estaba versionado en el repositorio se perdió: la configuración de los servicios del sistema, los certificados de seguridad, la llave de acceso usada para el despliegue automático, el ejecutor de integración continua propio, y la base de datos completa.

## Qué se reconstruyó

- Un usuario de servicio dedicado y su estructura de directorios de trabajo, con una llave de acceso nueva registrada en el repositorio (la anterior se dio de baja; su parte privada se perdió con el disco).
- El conjunto de servicios de apoyo necesarios para el sistema (video, coordinación entre procesos, y el entorno de construcción del frontend).
- La base de datos, recreada aplicando el conjunto completo de migraciones versionadas mediante la herramienta propia del proyecto, en vez de cargar un respaldo suelto. Se creó además un rol de aplicación de solo datos, sin permiso de modificar la estructura.
- Los servicios permanentes del sistema (servidor web con HTTPS por certificado nuevo, el backend bajo el gestor de servicios del sistema operativo, y el servicio de video), todos configurados para arrancar automáticamente.
- El mecanismo de despliegue manual, verificado de punta a punta con reversión automática si algo falla.
- El ejecutor de integración continua propio, reinstalado como servicio con permisos acotados solo a los scripts de despliegue.

## Hallazgo: un bug preexistente en la rama principal

El indicador de "listo" del backend fallaba de forma permanente, aunque el de "salud básica" respondía bien. La causa, el código mantiene a mano, en una lista fija, cuáles migraciones deben estar aplicadas para considerar el esquema completo; esa lista hacía referencia a una migración con un número que ya había sido reasignado a una migración distinta en el repositorio. El error existía desde la renumeración, sin que nadie lo notara, porque la prueba que ya lo habría detectado nunca se corrió antes de integrar el cambio a la rama principalm no hay verificación de pruebas obligatoria en las contribuciones.

**Corregido:** la lista fija se reemplazó por una verificación que compara contra los archivos de migración reales en disco, se agregó una prueba dedicada a esa comparación, y se instaló una verificación automática que corre antes de permitir cualquier cambio nuevo, para que el mismo tipo de error no vuelva a pasar inadvertido.

## Otros hallazgos durante la reconstrucción

- El nombre del paquete de un servicio de coordinación entre procesos cambió de proveedor en la versión del sistema operativo usada en la máquina nueva; el procedimiento de arranque automático no lo reconocía por su nombre anterior. Corregido detectándolo por su comportamiento real en vez de por su nombre de paquete.
- El nombre de la unidad de servicio de la base de datos cambió respecto al que usa la distribución nativa del sistema operativo, lo que afecta cualquier referencia a esa unidad en otros servicios.
- Un servicio de administración de base de datos, ajeno al proyecto, ocupaba el puerto que necesitaba el servidor web; se reubicó a un puerto distinto.
- Al intentar aplicar las migraciones con el rol de aplicación de solo datos, el proceso falló porque ese primer paso siempre necesita permiso para crear estructura, permiso que el rol de aplicación correctamente no tiene. Se resolvió creando un rol dedicado, distinto tanto del rol de aplicación como del superusuario de la base, con permiso acotado solo para administrar la estructura de las migraciones.

## Trabajo en paralelo sin coordinación

Durante la sesión, otra persona del equipo responsable de la base de datos trabajaba sobre la misma base sin saber que ya se había recreado, porque la encontró vacía. Se verificó que solo existe una base y un rol de aplicación, y que las credenciales vigentes siguen siendo válidas. La lección es avisar antes de recrear un recurso compartido, aunque parezca una operación de rutina.

## Estado al cierre

Todos los servicios permanentes quedaron activos y configurados para arrancar en el reinicio del equipo. El despliegue manual y automático quedaron verificados, con reversión disponible. La base de datos quedó con el esquema completo y sin datos de operación previos (controladores registrados, historial de auditoría); no se pudo recuperar el estado de operación anterior al reinicio.

## Pendientes de mayor prioridad

- Rotar las credenciales que quedaron expuestas durante la reconstrucción, incluida una credencial temporal usada para destrabar las migraciones.

## Lecciones

Lo que no está versionado, no existe, se perdió por completo lo que solo vivía en el disco del servidor anterior, mientras que lo versionado se reconstruyó en minutos. Un despliegue limpio es la única prueba real de que el sistema funciona desde cero, el bug de migraciones llevaba días en la rama principal y solo apareció al montar una base nueva. Las listas mantenidas a mano se desincronizan con el tiempo; la solución no fue corregir el valor, sino dejar de mantenerla a mano.
