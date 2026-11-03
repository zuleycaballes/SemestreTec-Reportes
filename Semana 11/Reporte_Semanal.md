# Reporte Semanal

| Campo | Dato |
| --- | --- |
| **Nombre** | Zuleyca Guadalupe Balles Soto |
| **Matrícula** | A01741687 |
| **Programa Académico** | ITC |
| **Empresa** | Dirección de Vialidad y Semaforización del Ayuntamiento de Hermosillo |
| **Tutor Académico** | Mario Durán Vega |
| **Fecha** | 1 de noviembre de 2026 |
| **Periodo reportado** | Semana 11 - 26 al 30 de octubre de 2026 |

> **Nota sobre confidencialidad.** El proyecto está sujeto a un acuerdo de confidencialidad. Este reporte describe metodología, decisiones técnicas y aprendizajes obtenidos, omitiendo identificadores de infraestructura, rutas de despliegue, credenciales y cualquier dato que permita ubicar equipos o instalaciones específicas.

---

## 1. Descripción de la fase u objetivos desarrollados en la semana declarados en el cronograma

### 1.1 Contexto del proyecto

Los términos **nodo**, **ventana de detección**, **contrato de comunicación**, **sincronía**, **identidad de controlador**, **corte controlado (PC-11)**, **escenario**, **castigo por ausencia**, **latch** y **Auditoría** ya quedaron definidos en reportes anteriores. Se agregan cinco para esta semana. La **ventana de cierre** es el tramo final de un escenario en el que los grupos de luces que terminan deben despejar el cruce (parpadeo, amarillo y rojo de despeje). La **decisión congelada** consiste en si el siguiente escenario que depende de un detector se atiende o se castiga, que el controlador fija al abrirse esa ventana y no vuelve a evaluar al entrar. Un **cambio tardío** es una detección que aparece o desaparece cuando esa decisión ya está tomada. Si aparece con tiempo suficiente para completar el despeje, el controlador puede **rescatar** el escenario; si ya no alcanza, lo **difiere**, es decir, lo castiga en ese ciclo y conserva el latch para atenderlo en el siguiente.

La semana tuvo dos frentes. El primero fue validar, en el controlador de laboratorio, qué hace el semáforo cuando la detección cambia o falla justo antes del escenario que depende del detector. El segundo fue revisar y confirmar el diseño de la retención automática de versiones del firmware (RET-01), contrastándolo con el código y con lo que realmente está instalado en los equipos. Además, durante unos tres días una falla de internet limitó el trabajo con el repositorio y controladores.

### 1.2 Actividades declaradas en el cronograma para la Semana 11

Según el cronograma, la Semana 11 continúa las Actividades 9, 10 y 11, que iniciaron en la semana 10 y se mantienen hasta la semana 12:

- **Actividad 9.** Validar cambios mediante pruebas controladas y en campo, asegurando que las modificaciones implementadas no afecten la operación del sistema antes de su despliegue.
- **Actividad 10.** Implementar herramientas de monitoreo y diagnóstico, como registros de eventos (logs), trazas del sistema y métricas de desempeño que faciliten el análisis operativo.
- **Actividad 11.** Coordinar el despliegue de nuevas versiones del sistema, asegurando una implementación controlada en los nodos activos y minimizando riesgos durante la actualización.

### 1.3 Objetivos efectivamente perseguidos

**Actividad 9 - en curso.** Se validó, en pruebas controladas en el controlador de laboratorio, cómo decide el semáforo cuando la detección cambia o falla cerca de la transición hacia el escenario con detector. El resultado fue de 7 pruebas aprobadas, cada una con su ciclo de seguimiento también aprobado:

- Una detección tardía que llega dentro de la ventana de cierre, el escenario se rescató, con su propio motivo registrado, y el latch se consumió una sola vez.
- Una detección que desaparece después de haber sido aceptada, ya sea dentro de la ventana de cierre o después de entrar al escenario (2 pruebas), el escenario se mantuvo atendido en ambos casos.
- La cámara caída o el detector desconocido desde antes de la ventana (2 pruebas), se aplicó la política configurada, sin inventar una detección.
- La misma falla pero llegando con la decisión ya congelada (2 pruebas), el escenario se castigó, es decir, una falla de la cámara no se toma como si fuera una detección real.

*Justificación de cobertura parcial:* la actividad pide pruebas controladas y en campo. Todo se hizo en el controlador de laboratorio y no se probó nada en un equipo de calle. Además, se obtuvo un ciclo válido por prueba, con una sola configuración (castigo "extender el siguiente" y política de atender), un solo caso de rescate, y no se cubrieron las políticas de omitir, respaldo y llamada periódica, ni el castigo "extender el anterior" en el equipo real. Detalle completo en **Anexo A**.

**Actividad 10 - sin avance directo.** Esta semana no se trabajó en herramientas de monitoreo ni de diagnóstico. *Justificación:* el tiempo se dedicó a la validación (Actividad 9) y al diseño de la retención (Actividad 11). Lo único cercano es de diseño y no se implementó, la retención contempla un estado de solo lectura y registrar cada intento de subida con su causa, para poder diagnosticar fallos.

**Actividad 11 - en curso.** Se cerró RET-01, la fase que investiga y confirma el diseño de la retención automática de versiones antiguas del firmware con respaldo en el sistema central, que es parte del mecanismo con el que se actualizan los controladores. El diseño se revisó contra el código y contra lo instalado en dos controladores, se contrastó con tres herramientas establecidas (OSTree, git-annex y dnf) y con el propio script de despliegue del sistema central, y de ahí salieron ocho correcciones. El responsable del proyecto aprobó el diseño. También se identificaron doce condiciones en las que se podría retirar una versión que todavía hace falta, cada una con su mitigación. *Justificación de avance parcial:* RET-01 es solo diseño, no se implementó, reinició ni desplegó nada. La implementación corresponde a RET-02, que arrancó esta semana y sigue en curso, y la instalación en los controladores corresponde a RET-03. Detalle completo en **Anexo B**.

### 1.4 Relación entre el cronograma y el trabajo de la semana

Esta semana el cronograma y el trabajo realizado coincidieron en las Actividades 9 y 11. La 9 avanzó en su parte de laboratorio, y la 10 no tuvo avance. Las tres siguen programadas hasta la semana 12.

**Actividades no programadas para esta semana.**

- **Mejora de arquitectura (Actividad 6, programada para las semanas 6 a 9) y seguridad (Actividad 12, programada para las semanas 13 y 14).** El diseño de la retención busca estabilidad y mantenimiento del actualizador, y la revisión de los equipos dejó hallazgos de seguridad fuera del alcance de RET. Se documentaron y no se atendieron.

**Falla de internet.** Durante unos tres días la conexión falló y limitó el trabajo. No se podían subir ni bajar cambios a GitHub (la consola de Git marcaba un error de tiempo de espera, HTTP 408), las comunicaciones fallaban y apenas se podía entrar al controlador, que sacaba la sesión a los pocos minutos, así que no se podían hacer tareas pesadas. No es una actividad del cronograma, pero sí frenó el avance de la semana, y cuando la conexión se normalizó el trabajo siguió sin mayores problemas.

---

## 2. Descripción del desarrollo de las actividades semanales (metodología, recursos, procedimientos)

### 2.1 Metodología

El criterio de la semana fue contrastar lo que dicen el repositorio y el diseño contra lo que realmente corre en los equipos, y no dar por buena la evidencia obtenida en condiciones poco confiables. La validación se diseñó para poder fallar, con un ciclo de seguimiento después de cada prueba, ciclos inválidos que no cuentan ni a favor ni en contra, y siete fallas inyectadas a copias del motor para comprobar que el script realmente las detecta. Los tiempos se midieron con el reloj del propio controlador y no con el de la computadora. Un primer intento hecho en condiciones de red malas se descartó como evidencia. En RET-01 toda la interacción con los equipos fue de lectura, y cada corrección al diseño se ligó a un hallazgo concreto, a una herramienta comparable o a un riesgo identificado. Las aprobaciones se registraron por escrito. Lo que no se probó se dejó marcado como pendiente, con su razón.

### 2.2 Recursos y herramientas

- **Infraestructura:** controlador de laboratorio con firmware propio, su servicio de control de luces y el puente de visión, servidor del sistema central.
- **Aplicación:** firmware del CSI y su actualizador en Python, backend del sistema central, repositorio de código en Git.
- **Verificación:** script de validación que consume el flujo de estado del controlador por WebSocket, pruebas automáticas del firmware, copias del motor con fallas inyectadas.

### 2.3 Procedimientos seguidos y actividades realizadas

**Validación de cambios tardíos y pérdida de visión.** Al controlador de laboratorio, que traía cuatro escenarios sin detectores, se le agregó el detector en el segundo escenario (el consumidor) y quedó con el castigo "extender el siguiente" y la política de atender ante detector desconocido (lo que hace el controlador cuando la cámara se cae o un detector no entrega una lectura confiable). El primer intento se hizo desde mi computadora por Wi-Fi y terminó como evidencia insuficiente, sin evaluar ninguna prueba (35 % de pérdida de paquetes, 891 ms de mediana por envío y el flujo de estado caído), así que no se usó. La corrida válida se ejecutó dentro del propio controlador con el puente de visión pausado, duró unos 21 minutos y midió 12,994 cuadros con 2,483 envíos de estado, ninguno rechazado. Primero hace una calibración sin inyección y después cada prueba va seguida de un ciclo de seguimiento; el envío del estado se planea para que caiga justo al abrirse la ventana de cierre, y cada ciclo se clasifica como rescatado o diferido. Un ciclo se descartó por fase (hueco de 23 ms) y no contó.

**Sensibilidad y hallazgos.** Sobre copias del motor se inyectaron siete fallas, una por una (por ejemplo, reevaluar la decisión congelada, revocar una admisión, ignorar la política o rescatar sin revisar el despeje), y el script detectó las siete. Además se hicieron dos pasadas en simulación con el servicio real. Salieron tres hallazgos. (1) Con "extender el anterior" el rescate en vivo casi no ocurre, porque el envío del estado fuerza su propio tick con menos tiempo que el despeje completo y el escenario queda diferido; con "extender el siguiente" sí ocurre. (2) Una falla de la cámara que llega con la decisión congelada castiga el escenario aunque la política sea atender, lo que conviene confirmar con el responsable. (3) Un documento de diseño anterior ya no describe el comportamiento actual y no se usó como expectativa.

**Falla de internet y subida al repositorio.** Durante unos tres días el push y pull de la rama de trabajo terminaba con HTTP 408 y el acceso al controlador se cortaba a los pocos minutos. Al medir el envío, el servidor recibía unos 315 KB en 52 segundos (alrededor de 6 KB/s), muy por debajo de lo necesario para un archivo de captura comprimido de unos 2.4 MB, y cambiar el buffer de envío o el backend TLS no ayudó, así que la causa fue la conexión y no la configuración. Las pruebas pesadas con el controlador se corrieron dentro del propio equipo, como la corrida válida de la validación.

**RET-01: verificaciones y comparación.** Antes de trabajar se confirmó de qué ramas salía el diseño, que el código instalado en los dos equipos no coincide con la rama principal ni entre sí, que no hay trabajo paralelo sobre el actualizador y cómo protegía versiones la limpieza manual. Con código fuente y manuales oficiales se compararon cuatro ejes (protección de versiones, exclusión mutua, confirmación del respaldo y recuperación) en OSTree, git-annex, dnf y el propio script de despliegue del sistema central, que se usó como contraejemplo. Mender y SWUpdate se descartaron porque trabajan con particiones A/B.

**RET-01: evidencia de los equipos, correcciones y aprobación.** En los dos controladores solo se leyeron servicios, versiones, procesos y espacio. El hallazgo principal fue que, en uno de ellos, un servicio auxiliar llevaba 12 días ejecutando código de una versión antigua y no está en la lista de servicios que reinicia el actualizador; una regla de "conservar solo la actual y la anterior" la habría borrado en uso. Además, de nueve archivos del actualizador comparados por hash, cinco difieren en al menos un equipo, y los dos equipos no tienen el mismo actualizador. Con esto se propusieron ocho correcciones, detectar el uso por el directorio de trabajo real de cada proceso, definir "pendiente" por la bitácora, contar la espera desde el último cambio observado, no reutilizar versiones retiradas, esperar el bloqueo con tope (el mecanismo que impide que dos operaciones del actualizador corran a la vez), subir el respaldo por el canal seguro existente con credencial por equipo, verificar de nuevo la copia justo antes de borrar y distinguir por hash una subida fallida de una versión dañada y de una con fallas. Se aprobó el diseño ajustado con una condición (distinguir esos tres casos), se aprobó que el CSI suba su propio respaldo, se aprobó la espera y reintento de la activación programada siempre que no se atore ningún proceso (incluido el visto bueno vial) y se aceptó por ahora la máquina virtual del sistema central como única copia. La RET-02 (implementar y probar sin desplegar) arrancó esta semana.

### 2.4 Evidencia

El respaldo de lo descrito son cuatro documentos internos redactados durante la semana, el reporte de validación de cambios tardíos y pérdida de visión, el reporte de la falla al subir al repositorio, el dictamen de diseño de RET-01 y su reporte de cierre. Son documentos internos del proyecto, así que no se anexan completos por el acuerdo de confidencialidad.

Se anexan dos en versión con la información sensible omitida:

- **Anexo A.** Validación de cambios tardíos y pérdida de visión en el controlador de laboratorio: cómo decide el firmware, criterios, resultados y límites.
- **Anexo B.** RET-01, diseño de la retención automática de versiones: investigación, correcciones, evidencia de los equipos, riesgos y aprobación.

---

## 3. Listado de las actividades siguientes o pendientes no resueltas, con justificación

### 3.1 Estado de las actividades declaradas para la semana

**Actividad 9, validar cambios con pruebas controladas y en campo.** Cubierta en pruebas controladas, pendiente en campo. *Justificación:* todo se validó en el controlador de laboratorio y ninguna prueba se corrió en calle; además, cada prueba tiene un solo ciclo válido y una sola configuración. *Plan de acción:* repetir con más ciclos, con el castigo "extender el anterior" en el laboratorio y con las políticas de omitir, respaldo y llamada periódica; contrastar los motivos de decisión con la Auditoría; y repetir en calle, en una ventana autorizada y ahora que el controlador de calle ya está actualizado, la validación física. *Resultado esperado:* un comportamiento ante detección tardía y pérdida de la cámara validado en laboratorio con varias configuraciones y en campo antes de activarse en un nodo en operación.

**Actividad 10, monitoreo y diagnóstico.** Sin avance directo. *Justificación:* la semana se dedicó a las Actividades 9 y 11, y lo relacionado con monitoreo quedó solo en el diseño de la retención. *Plan de acción:* corregir el chequeo de salud para que revise el cuerpo de la respuesta. *Resultado esperado:* un diagnóstico que no dé "sano" cuando el cuerpo de la respuesta dice lo contrario.

**Actividad 11, coordinar el despliegue de nuevas versiones.** Diseño cubierto, implementación en curso. *Justificación:* RET-01 era solo investigación y diseño, por indicación de la tarea, y RET-02 sigue en desarrollo. *Plan de acción:* terminar RET-02 (implementar y probar sin desplegar) sobre el diseño ajustado y sus criterios de aceptación; RET-03 (instalar) solo con autorización. *Resultado esperado:* la retención probada con los casos de riesgo del dictamen y lista para pedir autorización de instalación.

**Pendientes de la semana anterior.** El controlador de calle ya fue actualizado por el responsable del proyecto, así que el corte controlado (PC-11) dejó de ser un pendiente mío. No participé en esa actualización, por lo que su evidencia no forma parte de este reporte. Las condiciones previas que había listado (certificado del sistema central, sincronización de la identidad, ajuste de los rojos de 1 segundo, revisión del parche del compilador por su dueño y chequeo de salud) no tuvieron avance de mi parte esta semana. Del alcance de detección que no se había cubierto avanzó lo siguiente, el castigo "extender el siguiente", que solo se había probado en simulador, se probó ahora en el controlador real, y el flujo de estado por WebSocket se usó en el controlador real como fuente de evidencia. Siguen sin cubrirse otros detectores, el castigo "escenario específico", los castigos por reparto, los pulsos menores a 1 segundo, el modo estricto de contratos en el equipo real, y las dos observaciones de diseño (una detección que empieza dentro de la propia entrada del escenario no sirve a ese escenario ni se guarda, y el latch no caduca por tiempo).

### 3.2 Pendientes generados en la semana

**1. Decisión de firmware sobre el rescate y la falla tardía.** *Justificación:* con "extender el anterior" el escenario queda diferido y no rescatado, y una falla de la cámara con la decisión congelada castiga el escenario aunque la política sea atender. *Plan de acción:* plantear ambos casos al responsable y al equipo de firmware y actualizar el documento de diseño anterior. *Resultado esperado:* una decisión documentada y, si cambia el comportamiento, la prueba actualizada.

**2. Continuación de RET-02.** *Justificación:* RET-02 sigue en curso y parte de que el actualizador instalado no coincide con el repositorio y los dos equipos difieren entre sí. Además, uno de los equipos puede no tener la dirección, la autoridad certificadora ni la credencial del sistema central, y faltan la ruta de subida con tope de 64 MB y un directorio durable para el almacén. *Plan de acción:* trabajar sobre una base común (comparar el actualizador instalado en cada equipo contra el repositorio, elegir una versión base, regenerar las sumas de verificación y dejar el mismo actualizador en los dos equipos) y resolver esas dependencias antes de probar la subida del respaldo. *Resultado esperado:* una base común y la subida del respaldo probada de extremo a extremo.

**3. Respaldo del almacén.** *Justificación:* los respaldos quedarán por ahora en una sola máquina virtual (riesgo aceptado). *Plan de acción:* mantener documentado el almacenamiento segmentado como pendiente. *Resultado esperado:* el riesgo del respaldo único visible hasta que haya disco.

### 3.3 Actividades cercanas

Según el cronograma, la Semana 12 es la última de las Actividades 9, 10 y 11, y a partir de la semana 13 inicia la Actividad 12 (seguridad y modos de fallo controlado):

- **Actividad 9.** Validar cambios mediante pruebas controladas y en campo, asegurando que las modificaciones implementadas no afecten la operación del sistema antes de su despliegue.
- **Actividad 10.** Implementar herramientas de monitoreo y diagnóstico, como registros de eventos (logs), trazas del sistema y métricas de desempeño que faciliten el análisis operativo.
- **Actividad 11.** Coordinar el despliegue de nuevas versiones del sistema, asegurando una implementación controlada en los nodos activos y minimizando riesgos durante la actualización.

Los pendientes de esta semana caen directamente en ellas. Las repeticiones de la validación de detección y la validación en calle corresponden a la Actividad 9; el chequeo de salud, a la Actividad 10; RET-02 y el seguimiento de la actualización del controlador de calle, a la Actividad 11. La falla de la cámara con la decisión congelada y el servicio de actualización sin autenticación se relacionan con la Actividad 12, que inicia después. El resultado esperado es llegar a la semana 13 con la validación de detección ampliada, la retención implementada y probada sin desplegar, y la validación en calle planeada sobre el controlador ya actualizado.

---

## 4. Autorreflexión personal: problemas enfrentados, aprendizajes y desempeño

### 4.1 El patrón que más se repitió esta semana

Lo que parece decir que todo está bien no siempre lo dice. La rama principal del repositorio no era lo que corría en los equipos. Un documento de diseño ya no describía el comportamiento actual, y una regla simple de conservar la versión actual y la anterior habría borrado una versión en uso. En todos los casos lo que resolvió fue medir lo real, qué proceso usa cada versión o qué hash tiene cada archivo, en vez de confiar en el indicador.

### 4.2 Errores propios y lo que aprendí de cada uno

El principal fue hacer el primer intento de la validación desde mi computadora por Wi-Fi, con una red que perdía paquetes. El script lo marcó como evidencia insuficiente en vez de dar un veredicto, y la corrida válida se hizo dentro del controlador. El aprendizaje es medir la calidad de la red antes de una prueba sensible a tiempos, y correr la prueba cerca del sistema cuando se pueda.

El segundo tiene que ver con el diseño de RET-01, el diseño de partida se había escrito sobre el repositorio y daba por hecho que los equipos coincidían con él. Al contrastarlo con lo instalado, cinco de nueve archivos del actualizador diferían y un servicio seguía en una versión antigua. Desde ahora, la fuente de verdad para diseñar una actualización es lo que está instalado, no la rama.

### 4.3 Aprendizajes profesionales y consideraciones éticas

- El primer intento fallido de la validación quedó registrado con su causa y no se usó como evidencia, en lugar de ocultarlo o interpretarlo.
- Los límites de la validación (un ciclo por prueba, una configuración, nada en calle, sin contraste con la Auditoría) se informaron como límites, y el comportamiento de la falla tardía se dejó como pendiente de confirmación, no como correcto.
- En los equipos solo se hicieron lecturas, no se tocó ninguna configuración ni servicio, y el diseño no incluye una opción de forzar el borrado automático. Si no se puede verificar el respaldo, se conserva la versión.

### 4.4 Sobre el cronograma

Esta semana el cronograma coincidió con el trabajo real en las Actividades 9 y 11, y no en la 10, que no tuvo avance y que quedó con una sola semana más. La falla de internet de unos tres días limitó parte de la semana, aunque no corresponde a ninguna actividad del cronograma. El diseño de la retención se acerca a la mejora de arquitectura (Actividad 6) y sus hallazgos a la seguridad (Actividad 12), ambas fuera de su ventana. Conviene que en la semana 12 la Actividad 10 reciba tiempo explícito para no cerrar el bloque con el avance más bajo de las tres.

### 4.5 Desempeño, bienestar y acciones que voy a mantener

En lo personal, la semana estuvo bien en general. Lo frustrante fue que durante unos tres días el internet no funcionó bien, y eso dificultó también el trabajo con los controladores (la sesión se cortaba a los pocos minutos y no podía hacer tareas pesadas), pero después se normalizó y todo salió bien. Lo que ayudó fue dejar cada documento (la validación, el dictamen y el cierre) escrito el mismo día en que se hizo el trabajo, y apoyarme en criterios de aprobación definidos de antemano y no en juicio del momento.

Acciones que se van a sostener:

- Medir la calidad de la red antes de pruebas sensibles a tiempos y correrlas cerca del sistema.
- Diseñar a partir de lo instalado en los equipos y no solo de lo que dice el repositorio.
- Mantener el registro de lo no verificado como pendiente, con su razón, hasta comprobarlo.
