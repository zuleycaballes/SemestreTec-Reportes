# Reporte Semanal

| Campo | Dato |
| --- | --- |
| **Nombre** | Zuleyca Guadalupe Balles Soto |
| **Matrícula** | A01741687 |
| **Programa Académico** | ITC |
| **Empresa** | Dirección de Vialidad y Semaforización del Ayuntamiento de Hermosillo |
| **Tutor Académico** | Mario Durán Vega |
| **Fecha** | 8 de noviembre de 2026 |
| **Periodo reportado** | Semana 12 - 03 al 06 de noviembre de 2026 |

> **Nota sobre confidencialidad.** El proyecto está sujeto a un acuerdo de confidencialidad. Este reporte describe metodología, decisiones técnicas y aprendizajes obtenidos, omitiendo identificadores de infraestructura, rutas de despliegue, credenciales y cualquier dato que permita ubicar equipos o instalaciones específicas.

---

## 1. Descripción de la fase u objetivos desarrollados en la semana declarados en el cronograma

### 1.1 Contexto del proyecto

Los términos de reportes anteriores (**nodo**, **ventana de detección**, **escenario**, **castigo por ausencia**, **latch**, **Auditoría**, **ventana de cierre**, **cambio tardío**, **rescate** y la serie de **retención**) se mantienen. Se agregan cinco. La **gracia** es el tiempo que una release debe esperar sin protección antes de poder retirarse. El **comprobante** es la constancia del sistema central de que recibió la copia exacta de un respaldo; sin él no se borra nada. El **modo simulación** calcula qué retiraría la retención, pero no archiva ni borra. Un **falso "libre"** es una detección que reporta que no hay vehículo cuando sí lo hay.

La semana tuvo tres frentes, cerrar la retención automática de versiones, instalar la versión nueva del firmware en el laboratorio desde el repositorio y repetir sobre ella las validaciones de detección, y hacer un diagnóstico de punta a punta de la visión y del panel de detección.

### 1.2 Actividades declaradas en el cronograma para la Semana 12

La Semana 12 es la última de las Actividades 9, 10 y 11, que iniciaron en la semana 10:

- **Actividad 9.** Validar cambios mediante pruebas controladas y en campo, asegurando que las modificaciones implementadas no afecten la operación del sistema antes de su despliegue.
- **Actividad 10.** Implementar herramientas de monitoreo y diagnóstico, como registros de eventos (logs), trazas del sistema y métricas de desempeño que faciliten el análisis operativo.
- **Actividad 11.** Coordinar el despliegue de nuevas versiones del sistema, asegurando una implementación controlada en los nodos activos y minimizando riesgos durante la actualización.

### 1.3 Objetivos efectivamente perseguidos

**Actividad 9 - cerrada en laboratorio.** Con la versión nueva instalada se corrieron cuatro validaciones de detección seguidas en el controlador de laboratorio, y las cuatro terminaron aprobadas. En la de cambios tardíos y pérdida de visión pasaron 7 pruebas con 2 ciclos válidos cada una y 14 seguimientos, la detección tardía se rescató, la admisión no se revocó cuando la detección desapareció, la cámara caída aplicó la política configurada sin inventar una detección, y la misma falla con la decisión ya congelada castigó el escenario. En la de salidas, 3 casos por 2 ciclos válidos, contrato de entrada 13 y los 6 ciclos coincidentes con la Auditoría; en los 6,015 cuadros medidos las salidas ordenadas y las leídas del controlador coincidieron siempre. En 204 escenarios, el error máximo de duración fue de 0.16 s. *Justificación de cobertura parcial:* solo se validó en el controlador de laboratorio, con un programa que no entra en el perfil de la regla nueva del firmware (que sigue probada solo en simulación), una configuración y sin observar las lámparas físicas. Tampoco se cubrieron otras políticas ni el castigo "extender el anterior" en el equipo real.

**Actividad 10 - avance parcial.** No se construyó una herramienta nueva de monitoreo. La retención quedó instrumentada para diagnosticarla (estados registrados por trabajo, bitácora de cada intento de subida con su causa, estado de solo lectura y contabilidad de CPU, memoria y escritura), y con eso se midió su efecto en el laboratorio durante un retiro real, sin afectación medible. Además, el diagnóstico de visión y panel de detección revisó la ruta de monitoreo en vivo y encontró que el panel puede mostrar presencia congelada como si fuera actual. *Justificación:* el diagnóstico define qué construir, pero no lo implementa, y el chequeo de salud del CSI sigue pendiente.

**Actividad 11 - en curso, con el mayor avance.** La retención quedó implementada, el controlador respalda en el sistema central las releases que dejan de hacer falta, el sistema verifica la copia y emite el comprobante, y solo entonces se retira la copia local. Pasaron 89 pruebas del actualizador (antes 30, con 2 fallas) y 5,256 del sistema central sin regresiones, y se detectaron las 9 fallas inyectadas a propósito. En el laboratorio, el dictamen fue **aprobado con observaciones**, el respaldo se restauró idéntico, archivo por archivo; retirar una release no afectó la ejecución del semáforo; con el sistema central caído no se borró nada. La release nueva ya se instala desde el repositorio y se verifica archivo por archivo contra el código, algo que antes se indicaba a mano. Con autorización del responsable del proyecto, la retención se instaló además en un controlador de calle en modo simulación. *Justificación:* en calle solo calcula, no archiva ni borra, y su revisión queda para el inicio de la próxima semana.

### 1.4 Relación entre el cronograma y el trabajo de la semana

El trabajo coincidió con las tres actividades declaradas, que cierran su ventana, la 11 con el mayor avance, la 9 sólida en laboratorio y sin campo, y la 10 como la más baja. Sin programar, la semana tocó la Actividad 12 (seguridad) con un hueco de permisos corregido en la custodia de respaldos y tres hallazgos críticos documentados en el diagnóstico; la Actividad 6 (arquitectura) con el bloqueo común y los estados recuperables que la retención agregó al actualizador; y la Actividad 7 (incidencias) con cuatro casos, un reinicio inexplicado del controlador de laboratorio antes de instalar, un error de orden en la configuración del servidor web, una prueba que falló sin privilegios de administrador y un script de validación que marcaba como falla un comportamiento nuevo del firmware. Al final de la semana inició además la preparación de un sistema operativo propio para los controladores.

---

## 2. Descripción del desarrollo de las actividades semanales (metodología, recursos, procedimientos)

### 2.1 Metodología

El criterio fue que una garantía solo cuenta si hay una prueba capaz de romperla, y que un resultado solo vale si se contrastó con algo distinto del propio script. Por eso "sin comprobante válido no se borra nada" se demostró de tres maneras (variantes inválidas, extremo a extremo contra el backend real y fallas inyectadas a propósito), el efecto de la retención se midió contra una base tomada con la misma herramienta, las validaciones se repitieron sobre código verificado archivo por archivo y lo calculado por el script se recalculó aparte sobre la captura cruda. Lo que no se probó se dejó escrito como límite.

### 2.2 Recursos y herramientas

- **Infraestructura:** controlador de laboratorio con el firmware propio, servidor del sistema central y un controlador de calle.
- **Aplicación:** firmware del CSI y su actualizador en Python, backend del sistema central, panel web en JavaScript y Git.
- **Verificación:** pruebas automáticas, un script de validación sobre el flujo de estado del controlador, un script que compara una release contra el código y copias del motor con fallas inyectadas.

### 2.3 Procedimientos seguidos y actividades realizadas

**Cierre de la retención.** Se implementó la protección de releases en uso (el escaneo de procesos revisa directorio de trabajo, memoria y descriptores, y un proceso sin atribuir bloquea la pasada), un bloqueo común con el actualizador, el archivo con manifiesto, el comprobante y la revalidación antes de retirar. Se corrigieron tres fallas propias del diseño y, al correr las pruebas como usuario normal, una falló y destapó un hueco, el sistema central no comparaba los permisos de cada archivo del respaldo con los del manifiesto. Ahora un permiso alterado se rechaza.

**Validación en el controlador de laboratorio.** Con una línea base tomada antes, se instaló la retención sin tocar nada operativo y se probaron el archivo, la verificación, la restauración, el retiro real, el sistema central caído y el reinicio del backend. El efecto en ejecución se midió 900 s contra una base de 600 s, la edad de control de las luces (mediana 0.048 s) y los ciclos por segundo (9.59) fueron iguales y no hubo fallas. La carga máxima subió de 2.21 a 2.65 y se atribuyó a otra actividad del equipo, no a una pasada de 4 s con 0.74 s de CPU.

**Versión nueva y validaciones de detección.** Un script comparó lo instalado contra el código, los cinco componentes (223 archivos) resultaron idénticos. Las cuatro validaciones se corrieron dentro del propio controlador y con el puente de visión pausado. Como el firmware agregó una regla nueva para programas de 40 y 20 s, que se comprobó en simulación, se corrigió el script para reconocer ese perfil y exigir ahí, y solo ahí, el comportamiento nuevo.

**Diagnóstico de visión y panel de detección.** Se revisaron por lectura de código y pruebas locales el firmware de visión, el backend del sistema central y el panel. Cada hallazgo se etiquetó con su verificación y su severidad. Las suites existentes no tuvieron fallas atribuibles al código, y no se probó contra un controlador real. Salieron 23 hallazgos, 3 críticos, 4 altos, 13 medios y 3 bajos.

### 2.4 Evidencia

El respaldo son cinco documentos internos y los paquetes de evidencia que los acompañan (cierre de la retención, dictamen del laboratorio con 56 archivos de evidencia, validación de cambios tardíos, validación de salidas con su captura y diagnóstico de visión). No se anexan completos por el acuerdo de confidencialidad. Se anexan dos en versión con datos omitidos:

- **Anexo A.** Retención automática de versiones: cierre y dictamen en el controlador de laboratorio.
- **Anexo B.** Diagnóstico de visión y panel de detección.

---

## 3. Listado de las actividades siguientes o pendientes no resueltas, con justificación

### 3.1 Estado de las actividades declaradas para la semana

**Actividad 9.** Cubierta en laboratorio, pendiente en campo. *Plan de acción:* repetir con un programa de 40 y 20 s, con "extender el anterior" y con las políticas faltantes. *Resultado esperado:* el comportamiento ante detección tardía y falla de cámara validado con varias configuraciones y evidencia física.

**Actividad 10.** Avance parcial. *Plan de acción:* corregir el chequeo de salud, que no revisa el cuerpo de la respuesta y, en la rama principal, lee una ruta que no existe en el controlador de calle y podría revertir cada activación; y atender del diagnóstico lo que toca monitoreo (aviso de datos viejos en el panel y límite a la reconexión). *Resultado esperado:* un diagnóstico que no dé "sano" cuando el cuerpo dice lo contrario y un panel que avise si lo que muestra ya no es actual.

**Actividad 11.** Cumplida en laboratorio, calle en simulación. *Plan de acción:* revisar al inicio de la próxima semana la instalación en calle y avisar al responsable antes de cada cambio de fase (simulación, archivo, borrado real). *Resultado esperado:* la retención archivando y retirando en calle sin afectar la operación.

**Pendientes de la semana anterior.**

- **Decisión de firmware sobre el rescate y la falla tardía.** El responsable indicó que la regla nueva no es exclusiva de los dos cruceros para los que se pensó y que la regla base es que el ciclo empiece y termine a su hora. Sigue sin decisión si, con "extender el anterior", el rescate debe quedar diferido, y ampliar la regla a otros repartos requiere una tabla de escenarios y desfases que él debe definir.
- **Respaldo del almacén.** Sigue abierto: los respaldos viven en una sola máquina virtual (riesgo aceptado) y el almacenamiento segmentado quedó documentado como pendiente.
- **Condiciones previas** (certificado del sistema central, identidad, rojos de 1 segundo, parche del compilador, chequeo de salud). Sin avance de mi parte, salvo el certificado dentro del canal de la retención.

### 3.2 Pendientes generados en la semana

1. **Diagnóstico de visión.** 23 hallazgos sin atender y una decisión del responsable pendiente, el significado de los códigos de dirección está escrito de dos formas opuestas en el código.

### 3.3 Actividades cercanas

La Semana 13 inicia la **Actividad 12** (seguridad y modos de fallo controlado), que se mantiene hasta la semana 14:

- **Actividad 12.** Asegurar mecanismos de seguridad y modos de fallo controlado, permitiendo que el sistema responda adecuadamente ante errores o condiciones anómalas.

El trabajo de la próxima semana cae ahí, atender las tareas definidas en el diagnóstico, empezando por los tres críticos (proteger las credenciales de las cámaras, que el panel avise cuando la presencia ya no es actual y revisar los falsos "libre"), que además sostienen mi entregable principal, la ventana de detección y su puente hacia el control de luces. En paralelo se cierran los pendientes de las actividades que terminan, la revisión en calle, la gracia por release y el chequeo de salud.

**Trabajo previsto fuera del cronograma: sistema operativo propio para los controladores con Yocto.** No corresponde a una actividad del cronograma, aunque se acerca a la mejora de arquitectura y a la seguridad (un sistema con solo lo indispensable reduce lo que hay que mantener y proteger). La idea es reemplazar el sistema operativo actual por una imagen mínima y reproducible, probada primero en un controlador que arranca desde tarjeta SD, donde no hay riesgo de dejar la tarjeta inservible, y después adaptada a los equipos con memoria interna. Al cierre de la semana inició el estudio del arranque y se preparó en el servidor un volumen de tamaño fijo (150 GB) para compilar, de modo que no pueda llenar el disco de producción. El objetivo de la próxima semana es una primera imagen mínima que arranque en la tarjeta de prueba, sin quitar prioridad a los hallazgos críticos.

---

## 4. Autorreflexión personal: problemas enfrentados, aprendizajes y desempeño

### 4.1 El patrón que más se repitió esta semana

Por segunda semana, lo que parecía decir que todo estaba bien no lo decía. El dictamen de RET-01 tenía tres afirmaciones que el código corrigió al cerrar RET-02, y el diagnóstico mostró suites en verde con tres fallas críticas. En ambos casos lo que resolvió fue medir lo real, el código que corre y no el documento que lo describe, y corregir mi propio texto cuando no coincidía.

### 4.2 Errores del procedimiento y lo que aprendí de cada uno

La línea de configuración del servidor web se agregó antes de crear su archivo y la verificación falló, el orden correcto es primero el archivo, luego la verificación y al final la línea. Y una prueba que pasaba como administrador falló sin privilegios, y de ahí salió el hueco de permisos, así que lo que corre con usuarios normales se prueba también como usuario normal.

También aprendí algo de alcance, la retención evita que se acumulen releases, pero no resuelve el espacio del disco, y conviene que el responsable no espere de ella más de lo que da.

### 4.3 Aprendizajes profesionales y consideraciones éticas

- Cuando el controlador de laboratorio se reinició solo antes de instalar, no atribuí el reinicio a la retención ni lo ignoré, lo anoté como causa desconocida y tomé una base nueva.
- Los límites de las validaciones se informaron como límites, y la regla nueva del firmware se dejó como "probada solo en simulación", no como validada.
- La retención no tiene una opción de forzar el borrado, y en calle se instaló en simulación, con autorización y sin pasar de fase sin aviso ni revisión. El anexo del diagnóstico omite cómo aprovechar los hallazgos de seguridad.

### 4.4 Sobre el cronograma

El bloque de las Actividades 9, 10 y 11 cierra con la 11 completa en laboratorio, la 9 sólida sin campo y la 10 como la más baja, como se advirtió en el reporte anterior. La propuesta de darle tiempo explícito a la Actividad 10 no se tradujo en una herramienta nueva, y conviene decirlo con claridad. La Actividad 12 inicia en la semana 13 con su trabajo ya definido por el diagnóstico. El sistema operativo propio queda fuera del cronograma y se vigilará que no la desplace.

### 4.5 Desempeño, bienestar y acciones que voy a mantener

La semana tuvo mucha carga técnica, y lo que más ayudó fue dejar cada documento escrito el mismo día del trabajo y apoyarme en criterios de aprobación fijados de antemano. Por cuidado propio, dejé la revisión de la instalación en calle para el inicio de la próxima semana y no para el fin de semana.

Acciones que se van a sostener:

- Correr como usuario normal las pruebas de lo que corre con usuarios normales.
- Validar siempre desde el repositorio y verificar archivo por archivo lo instalado.
- No pasar de fase en calle sin aviso y sin revisión, aunque la prueba en laboratorio haya salido bien.
