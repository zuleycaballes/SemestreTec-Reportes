Anexo A - Reporte de cierre y dictamen: retención automática de versiones del firmware, RET-02 (implementación) y RET-03 (validación en laboratorio)

Origen: documentos internos de cierre de RET-02 y de reporte y dictamen de RET-03, entregados al responsable del proyecto.

> Versión con datos omitidos. Se omiten direcciones de red, nombres de servidores, de servicios y de equipos, rutas, versiones, huellas (hash), nombres de archivos y de personas ajenas a la residencia, credenciales y comandos, conforme al acuerdo de confidencialidad. El controlador donde se validó se identifica como equipo de laboratorio y el de calle como equipo de calle.

---

## Resumen y dictamen

**RET-02 cerrado:** la retención quedó implementada y probada sin desplegar. **RET-03 aprobado con observaciones** para el equipo de laboratorio. El equipo de calle queda sin autorizar por el dictamen hasta cumplir cuatro condiciones (al final).

El objetivo es que el controlador respalde en el sistema central las releases del firmware que dejan de hacer falta y luego las elimine de su almacenamiento, sin retirar nunca una que todavía se necesite. La implementación no modificó el servicio de control de luces ni nada fuera del actualizador.

## RET-02. Qué se implementó

| Requisito | Implementación |
| --- | --- |
| Proteger release actual, anterior, de recuperación, en uso, fijadas y pendientes | El escaneo de procesos revisa directorio de trabajo, mapas de memoria, descriptores y línea de comandos. Un proceso operativo sin atribuir, o una inspección incompleta, bloquean toda la pasada |
| No atorar al actualizador | Bloqueo común: el actualizador espera hasta 120 s y la retención no espera. El bloqueo solo dura la revalidación y el renombrado (tope de 30 s); el borrado físico va fuera |
| Archivo y comprobante | Archivo comprimido determinista con manifiesto y copia consistente de las bases. El sistema central verifica tamaño, huella, manifiesto y cada miembro antes de emitir el comprobante, ligado al controlador y al contenido exacto |
| Borrar solo con respaldo vigente | Antes de retirar, reverifica en el sistema central, vuelve a escanear procesos y compara la huella del árbol. Registra la intención, y al arrancar se reconcilia cualquier estado intermedio |
| Distinguir subida fallida, release corrupta y release con fallas | La integridad y el estado operativo viajan en el manifiesto, y cada intento de subida queda registrado con su causa |
| Sin worker en el sistema central | Por diseño aprobado, el controlador sube el respaldo y recibe el comprobante en la misma conexión; el sistema central nunca consulta a los controladores |
| Impacto en ejecución | Servicio con prioridad mínima, 25 % de CPU y tope de 256 MB. Nada se comprime ni se borra dentro de la activación |

Modos: simulación (inicial), archivo (respalda sin borrar), habilitado y apagado.

## RET-02. Pruebas y fallas encontradas

| Suite | Antes | Con RET-02 |
| --- | --- | --- |
| Actualizador del controlador | 30 pasan, 2 fallan (sumas de verificación desactualizadas) | **89 pasan, 0 fallan** |
| Backend del sistema central | 5,241 pasan, 1 falla previa, 265 omitidas | **5,256 pasan**, la misma falla previa. Sin regresiones |
| Fallas inyectadas a propósito | n/a | **9 de 9 detectadas** |

Casos cubiertos: concurrencia con la activación, versiones en uso, respaldos corruptos o alterados, comprobantes falsos o repetidos, disco lleno, reintentos, reinicios en cualquier punto, saltos de reloj y eliminación parcial. **Sin comprobante válido no se elimina ninguna versión**: se demostró con seis variantes inválidas, de extremo a extremo contra el backend real y con fallas inyectadas.

| Falla encontrada | Cómo se encontró | Corrección |
| --- | --- | --- |
| Un bloqueo seguía tomado hasta 20 s más si se mataba el actualizador a mitad de una activación | Prueba de terminación forzada | Cerrar el descriptor en las esperas y procesos hijos |
| El estado calculaba hashes de todas las releases en cada consulta | Revisión del flujo | Caché de integridad, que además se recalcula desde cero antes de archivar |
| **Hueco de permisos:** el sistema central no comparaba los permisos de cada miembro del respaldo con el manifiesto | Una prueba falló sin privilegios de administrador | Un permiso alterado se rechaza con 422 |

**Correcciones al dictamen de RET-01.** Las releases existentes no se contrastan contra un hash, porque el registro lo guarda siempre vacío (solo se comprueba que estén sus archivos). El canario necesita una activación, no dos. Y un proceso que ejecuta a través del enlace de la release sin directorio dentro de una release bloquea la pasada, variante más estricta que comparar tiempos de arranque.

**Hallazgo sobre lo instalado.** El chequeo de salud de la rama principal lee una ruta por defecto que no existe donde el servicio corre con otro usuario, así que cada activación se revertiría. No se incluyó en el paquete.

**Limitaciones.** No se probó en el entorno real el servidor web ni la política de seguridad del sistema operativo, el escaneo de procesos se probó con un directorio simulado y la carga en el controlador se midió en RET-03. El almacén del sistema central es la única copia (riesgo aceptado).

## RET-03. Validación en el equipo de laboratorio

Con la instalación real, el sistema central real y los servicios operando. La retención se instaló sin cambiar nada operativo; solo se reinició el servicio del actualizador.

| Prueba | Resultado |
| --- | --- |
| Instalación sin cambios operativos | La comparación contra la línea base quedó vacía antes y después |
| Archivo de una release | 2.07 MB comprimidos a partir de 22 MB; pasada de 3 s con 0.78 s de CPU y **0 bytes escritos** en el almacenamiento del controlador |
| Verificación y restauración en un área de prueba | Aceptada en el sistema central; árbol restaurado igual al original (417 líneas): **idéntico** |
| Retiro real de una release | Solo esa release; pasada de 4 s con 0.74 s de CPU; 21.5 MB más libres |
| Sistema central caído al subir y al reverificar | **Nada borrado**; reintento diferido 15 min y reverificación pospuesta |
| Reinicio del backend del sistema central | Saludable a los 15 s, misma huella en la verificación y el controlador se autentica solo |
| Diagnóstico final contra la línea base | Vacío: procesos, configuración, enlaces y bitácora iguales |

| Métrica | Base (600 s, 2,399 muestras) | Retiro (900 s, 3,599 muestras) |
| --- | --- | --- |
| Edad de control de luces, mediana | 0.04796 s | 0.04733 s |
| Edad de control, máximo | 0.10131 s | 0.10146 s |
| Ciclos por segundo | 9.5906 | 9.5881 |
| Fallas o falta de respuesta | 0 | 0 |
| Carga a 1 minuto, mediana / máximo | 1.51 / 2.21 | 1.52 / 2.65 |

La herramienta concluyó **sin afectación medible**. La carga máxima subió con la misma mediana y una pasada de 4 s con 0.74 s de CPU no la explica, así que se atribuye a otra actividad del equipo y queda como observación.

## Observaciones y pendientes del dictamen

**Condiciones antes de autorizar el equipo de calle:** (1) el código probado debe estar en el repositorio y el paquete debe salir de ahí; (2) aprobación escrita del responsable, con visto bueno vial y una ventana de día lejos de la activación programada; (3) alcance de red desde calle hasta el sistema central, comprobado antes de instalar; (4) credencial propia de calle, entregada de archivo a archivo.

**Recomendadas, no bloquean:**

- **Gracia por release.** La gracia global se reinicia cuando cambia la release activa o la anterior, y con actualizaciones frecuentes nunca habría candidatas.
- **Estado más claro.** Muestra "candidata" aunque la release ya esté respaldada y no muestra el reintento diferido.
- **Disco al 87 %.** La retención libera unos 23 MB por release y el consumo grande es la base de eventos del control de luces (1.28 GB).
- **Memoria no medida directamente,** porque el kernel no reporta el pico; el tope de 256 MB no tuvo eventos de falta de memoria.

**Hechos del canario:** el controlador de laboratorio se reinició antes de instalar, por una causa desconocida y no relacionada, y se tomó una base nueva. Una credencial se pegó una vez en un chat y se rotó. Una línea del servidor web se agregó antes de crear su archivo y la verificación falló.

**Fuera del alcance de la retención:** las rutas del servicio de actualización no tienen autenticación, una prueba del sistema central falla desde antes y el nombre de la serie colisiona con otro servicio del backend.

## Continuidad

- El equipo de calle requiere su propia autorización, ventana y credencial, y cada cambio de fase se avisa antes y se cierra con estabilidad verificada.
- Mientras no haya discos nuevos, el almacén vive en una sola máquina virtual, con el pendiente documentado de segmentar el almacenamiento y respaldarlo fuera de ella.
