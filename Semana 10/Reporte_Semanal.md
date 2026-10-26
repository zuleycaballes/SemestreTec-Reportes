# Reporte Semanal

| Campo | Dato |
| --- | --- |
| **Nombre** | Zuleyca Guadalupe Balles Soto |
| **Matrícula** | A01741687 |
| **Programa Académico** | ITC |
| **Empresa** | Dirección de Vialidad y Semaforización del Ayuntamiento de Hermosillo |
| **Tutor Académico** | Mario Durán Vega |
| **Fecha** | 25 de octubre de 2026 |
| **Periodo reportado** | Semana 10 - 19 al 23 de octubre de 2026 |

> **Nota sobre confidencialidad.** El proyecto está sujeto a un acuerdo de confidencialidad. Este reporte describe metodología, decisiones técnicas y aprendizajes obtenidos, omitiendo identificadores de infraestructura, rutas de despliegue, credenciales y cualquier dato que permita ubicar equipos o instalaciones específicas.

---

## 1. Descripción de la fase u objetivos desarrollados en la semana declarados en el cronograma

### 1.1 Contexto del proyecto

Los términos **nodo**, **ventana de detección**, **contrato de comunicación**, **sincronía**, **identidad de controlador**, **contrato de configuración (v1)** y **corte controlado (PC-11)** ya quedaron definidos en reportes anteriores. Se agregan siete para esta semana. Un **escenario** es cada tramo del ciclo del semáforo en el que ciertos grupos de luces tienen verde; en la configuración de referencia, tres escenarios de 20 segundos forman un ciclo de 60. El **castigo por ausencia** es lo que hace el controlador cuando un escenario depende de un detector y no hay demanda, en lugar de dar el verde completo, lo cede (por ejemplo, extendiendo el escenario anterior) y el ciclo conserva su duración. El **latch** es la memoria de una detección, si un vehículo se detecta mientras corre otro escenario y se va antes de que le toque su turno, el controlador recuerda que hubo demanda, atiende el escenario sin castigarlo y borra esa memoria una sola vez. La **Auditoría** es el módulo del sistema central que muestra las bitácoras y las decisiones que reportan los controladores.

La semana tuvo dos frentes. El primero fue cerrar en el controlador de laboratorio el corte controlado (PC-11) y el cierre del release (PC-12). El segundo fue validar que la detección realmente cambia lo que hace el semáforo, desde que el firmware recibe los estados del detector hasta que una detección breve se conserva hasta el turno de su escenario. Además se activó la ingesta de bitácoras de los controladores en el sistema central y hubo una junta para revisar las reglas de configuración de cada crucero de un corredor.

### 1.2 Actividades declaradas en el cronograma para la Semana 10

Según el cronograma, la Semana 10 inicia las Actividades 9, 10 y 11, que se mantienen hasta la semana 12.

- **Actividad 9.** Validar cambios mediante pruebas controladas y en campo, asegurando que las modificaciones implementadas no afecten la operación del sistema antes de su despliegue.
- **Actividad 10.** Implementar herramientas de monitoreo y diagnóstico, como registros de eventos (logs), trazas del sistema y métricas de desempeño que faciliten el análisis operativo.
- **Actividad 11.** Coordinar el despliegue de nuevas versiones del sistema, asegurando una implementación controlada en los nodos activos y minimizando riesgos durante la actualización.

### 1.3 Objetivos efectivamente perseguidos

**Actividad 9 - en curso.** Se repitió la PC-09 sobre la versión nueva del firmware y después sobre el release activo del sistema central, y quedó aprobada en 57 verificaciones. Además se validó, en pruebas controladas, la cadena completa de detección en el controlador de laboratorio:

- El firmware recibe y conserva los estados del detector y rechaza sin daño los datos inválidos, 26 verificaciones en el controlador real, después de 39 en simulador.
- El detector cambia lo que ejecuta el semáforo con castigo por ausencia, 3 ejecuciones válidas con demanda y 3 sin demanda, contrastadas con la Auditoría.
- Una detección de 1 segundo no se pierde antes de que llegue su escenario, 3 inyecciones y 3 controles, con el latch consumido una sola vez.

*Justificación de cobertura parcial:* la actividad pide pruebas controladas y en campo. Todo lo anterior se hizo en el controlador de laboratorio y no se probó nada en un equipo de calle. Tampoco se cubrieron otros detectores (el controlador de laboratorio solo tiene uno configurado), otros tipos de castigo ni pulsos menores a 1 segundo. Detalle completo en **Anexo B**.

**Actividad 10 - en curso.** Se activó la ingesta de eventos de los controladores en el sistema central. Estaba apagada a propósito, con respuesta "no configurada", hasta contar con su credencial, y ahora rechaza correctamente con 401 las peticiones sin autenticar. En la validación del castigo, la Auditoría funcionó como segunda fuente de evidencia, pues las decisiones de castigo que reporta el controlador coincidieron con lo que midió el script de prueba. *Justificación de avance inicial:* el historial local acumulado en los controladores y el agente que lo sincroniza siguen fuera de lo hecho esta semana, y el chequeo de salud del CSI todavía no revisa el contenido de su respuesta.

**Actividad 11 - en curso** El lunes se desplegó firmware en el controlador de laboratorio, se ensayó el rollback a la versión anterior y se regresó a la nueva, sin perder identidad ni configuración. Se respaldó la base de datos del sistema central con un respaldo restaurable, se desplegó el parche que enlaza al controlador activo con su instancia retirada. Con eso quedó desbloqueado el plan de configuración por controlador. *Justificación de avance parcial:* el controlador de calle sigue en la versión anterior y no se tocó. Detalle completo en **Anexo A**.

### 1.4 Relación entre el cronograma y el trabajo de la semana

Esta semana el cronograma y el trabajo realizado coincidieron en las tres actividades declaradas (9, 10 y 11). La 11 tuvo el mayor avance, la 9 avanzó en su parte de laboratorio y la 10 tuvo un avance inicial. Las tres siguen programadas hasta la semana 12.

**Actividades no programadas para esta semana.**

- **Seguimiento a incidencias (Actividad 7, programada para las semanas 3 a 5).** Durante el corte se atendieron tres incidencias de laboratorio, la línea del archivo de nombres del controlador de laboratorio que apuntaba al sistema central se había perdido, y por eso dos intentos de sincronización fallaron; el controlador de laboratorio estuvo apagado mientras se instalaba algo en él, y su lectura de estado regresó vacía; y el sistema central ignoraba la referencia a la instancia retirada del controlador, defecto que se corrigió con un parche. Se atendieron porque impedían cerrar el corte.
- **Junta de reglas de configuración por crucero.** El jueves se revisaron las reglas de configuración de 4 cruceros de un corredor, para confirmar que lo que se necesita en calle se pueda hacer en el programa y, si no, ajustar o agregar funciones. Todavía no existe un levantamiento de requerimientos formal; esta junta es la base para hacerlo. No corresponde a una actividad de esta semana, pero se relaciona con la Actividad 8 (ajuste real de ciclos, que quedó con cobertura parcial) y con la Actividad 14 (planificación de la expansión).

---

## 2. Descripción del desarrollo de las actividades semanales (metodología, recursos, procedimientos)

### 2.1 Metodología

El criterio de la semana fue que una prueba tiene que poder fallar. Cada validación de detección se diseñó con precondiciones explícitas, si una ejecución empezaba con un detector encendido, con una memoria vieja pendiente o con la cámara fuera de línea, se marcaba como **inválida** y no contaba ni a favor ni en contra. Cada prueba tuvo además un ciclo de control, sin detección, que demuestra que el resultado es consecuencia del detector. Siempre que fue posible, el resultado se contrastó con una fuente independiente del script, la Auditoría del sistema central, cruzando por identificador de ejecución y por hora. Todo se ejecutó en el controlador de laboratorio. Lo que no se comprobó se dejó marcado como pendiente o como no verificado.

### 2.2 Recursos y herramientas

- **Infraestructura:** controlador de laboratorio con firmware propio y servicio de visión, servidor de producción Linux con su base de datos relacional, y la Auditoría del sistema central.
- **Aplicación:** backend en Python, frontend en JavaScript con biblioteca de interfaz basada en componentes, firmware del CSI en Python con su servicio de control de luces, y el updater del CSI con su script de salud y de rollback.
- **Verificación:** script de validación de contrato, scripts de validación de detectores, pruebas automatizadas del firmware y del sistema central.

### 2.3 Procedimientos seguidos y actividades realizadas

**Corte controlado (PC-11) en laboratorio.** Antes de tocar nada se respaldaron los nueve archivos de identidad y configuración del controlador, con sus sumas de verificación. Se instaló el firmware desde el panel de actualización. Luego se verificó el estado de los servicios, el contrato de tiempos, la versión de la configuración y que la huella de la configuración fuera la misma en dos lecturas y igual al respaldo. Se ensayó el rollback a la versión anterior, se confirmó que el sistema quedara sano con la misma identidad, y se activó de nuevo el firmware. La observación posterior duró unos 6 minutos sin advertencias, que es corta y quedó anotada como excepción por si se pide una ventana mayor. Una limitación del ensayo es que, tras un rollback, la versión anterior queda igual a la actual, así que un segundo rollback seguido no tendría a dónde volver.

**Cierre del release (PC-12) y PC-09 sobre el release activo.** Al consolidar la evidencia se detectó que la corrida de PC-09 declaraba una versión del sistema central que no era la que realmente corría, y que la PC-10 se había ejecutado sobre una versión anterior, con cerca de 30 cambios de diferencia respecto a la de producción. Por eso se repitió la PC-09 sobre el release activo. El primer intento quedó rechazado, todo el lado del CSI pasó y fallaron las tres inspecciones del sistema central, por falta de respuesta, probablemente por el certificado interno. Se repitió con la verificación del certificado desactivada y quedó aprobado. Quedó registrado que ese resultado no valida el certificado, y que eso es una deuda de seguridad menor que resuelve el trabajo del certificado. La PC-10 no se repitió; en su lugar se hizo un envío de configuración de prueba desde Sincronía al controlador de laboratorio en modo sin destello, que pasó. El cierre quedó con **PC-09 aprobada, PC-10 aprobada con parcial** (dos pasos esperan un nodo con varios controladores) y **PC-11 aprobada en laboratorio con excepciones**.

**Ingesta de bitácoras (BIT-03).** Se generó un token aleatorio y se agregó a la configuración privada del backend, que conserva sus permisos restringidos. Se reinició solo el servicio del backend, porque el sistema solo relee esa configuración al arrancar. Se verificó que una petición sin token responde 401 y que el backend deja la entrada en su registro.

**Configuración de referencia para las pruebas de detección.** Se armó una configuración con un ciclo de tres escenarios de 20 segundos, un detector en el segundo escenario y castigo "extender el anterior". Se validó con el sistema central, el analizador del firmware y la revisión de tiempos. Se confirmó que se rechaza, como debe ser, un identificador histórico de detector y el castigo "extender el anterior" en el primer escenario. Se descubrieron dos reglas: la configuración debe declarar los mismos cuatro grupos que tiene el controlador de laboratorio, o el firmware no la aplica al instante, y las etiquetas de los grupos deben ser distintas del nombre del grupo, o el firmware las quita. En el controlador, la configuración persistió después de un reinicio y el ciclo se mantuvo en 60 segundos. Si la cámara está caída, el segundo escenario se atiende por seguridad. El caso sin demanda se probó con detector simulado y la cámara detenida, porque la cámara apunta a la calle y casi nunca hay 40 segundos sin vehículos.

**Validación de recepción de estados del detector.** Se probó primero en simulador y luego en el controlador real de laboratorio, con un cliente HTTP que envía estados del detector y los compara contra el estado publicado por el firmware. Se probaron: detector encendido y apagado, envío incompleto, detector desconocido, secuencia repetida y valores no booleanos. Los válidos se aceptan y los inválidos se rechazan con 422 o 409 sin alterar el último estado válido, el ciclo sigue y un estado válido posterior se acepta. Se añadieron dos pruebas al firmware. El flujo de estado por WebSocket y el modo estricto de contratos solo se probaron en simulador, porque en el controlador real implicaban reiniciar el servicio con otra variable de entorno.

**Validación del servicio normal y del castigo por ausencia.** Se corrió un script que ejecuta dos casos y compara lo programado contra lo efectivo y contra la salida física de las luces. En el caso con demanda, el escenario 2 se atiende completo. En el caso sin demanda, el escenario 2 se castiga y se extiende el 1, el ciclo se mantiene en 60 segundos y las luces en verde son las del escenario 1. Hicieron falta tres ejecuciones válidas por caso, y todas pasaron. El contraste con la Auditoría fue sobre 10 eventos de decisión de castigo de la serie, el escenario programado y el efectivo, la demanda, el castigo, las fuentes y el motivo coinciden con lo medido.

**Validación del latch.** El viernes se corrió el script del latch, mientras corre el escenario 1 se envía una detección de 1 segundo y se apaga, y se verifica que el latch aparezca en el mismo cuadro, que permanezca mientras el detector está apagado, que el escenario 2 entre sin castigo y que el latch desaparezca una sola vez. Cada inyección va seguida de un ciclo de control sin detección, donde el escenario sí se castiga y no queda ningún latch. En las 3 inyecciones y los 3 controles, todos pasaron. El detector estaba apagado 17.3 segundos antes de entrar al escenario, y el latch estuvo presente en el 100 % de los cuadros intermedios. Además, el script se probó en simulador con dos fallas inyectadas (que el latch se borre al apagar el detector y que nunca se consuma) y en ambos casos reportó la falla.

**Junta de reglas por crucero.** El jueves se revisaron las reglas de configuración de un corredor, crucero por crucero. El objetivo fue comprobar que cada configuración que se necesita en calle pueda hacerse en el programa. Se alcanzaron a ver unos cuatro cruceros. De lo que salió se tomará un levantamiento de requerimientos para ajustar o agregar funciones; todavía no se redacta.

### 2.4 Evidencia

El respaldo de lo descrito son siete reportes técnicos redactados durante la semana, el reporte de avance del corte controlado, el cierre del release, el reporte de la ingesta de bitácoras, el de la configuración de referencia y los tres de validación de detección (recepción de estados, castigo por ausencia y latch). Son documentos internos del proyecto, así que no se anexan completos por el acuerdo de confidencialidad.

Se anexan dos en versión con la información sensible omitida:

- **Anexo A.** Corte controlado (PC-11) y cierre del release (PC-12) en laboratorio: cronología, resultados, excepciones y decisión.
- **Anexo B.** Validación de la detección en el controlador de laboratorio: recepción de estados, castigo por ausencia y latch.

---

## 3. Listado de las actividades siguientes o pendientes no resueltas, con justificación

### 3.1 Estado de las actividades declaradas para la semana

**Actividad 9, validar cambios con pruebas controladas y en campo.** Cubierta en pruebas controladas, pendiente en campo. *Justificación:* todo se validó en el controlador de laboratorio y ninguna prueba se corrió en calle. *Plan de acción:* repetir en calle, en una ventana autorizada y con el equipo de campo presente, la validación física del corte, y ampliar las pruebas de detección a otros detectores, a otros tipos de castigo y a pulsos menores a 1 segundo. *Resultado esperado:* un cambio validado en laboratorio y en campo antes de activarse en un nodo en operación.

**Actividad 10, monitoreo y diagnóstico.** Avance inicial. *Justificación:* esta semana solo se activó la ingesta del lado del sistema central; el envío del historial local de los controladores y el agente que lo sincroniza no se trabajaron, y el chequeo de salud del CSI todavía solo revisa el código HTTP. *Plan de acción:* corregir el chequeo de salud para que revise el cuerpo de la respuesta, y retomar la sincronización del historial local. *Resultado esperado:* un diagnóstico que no dé "sano" cuando el cuerpo de la respuesta dice lo contrario, y las bitácoras de los controladores llegando al sistema central.

**Actividad 11, coordinar el despliegue de nuevas versiones.** Cubierta en laboratorio, pendiente en calle. *Justificación:* El controlador de calle depende de un certificado, de la sincronización de su identidad y de correcciones previas. *Plan de acción:* seguir el orden de la sección 3.2. *Resultado esperado:* el corte cerrado en el controlador de calle, o detenido con la causa documentada.

**Pendientes de la semana anterior.** El corte controlado en el controlador de calle sigue abierto. Repetir la PC-09 se cumplió, pero la revisión del parche del compilador por su dueño sigue pendiente. No hubo avance esta semana en que la disponibilidad del sistema revise el registro de migraciones completo, ni en la prueba de importación antes de activar un release, ni en el orden de migrar y desplegar; los tres quedan para las semanas 11 y 12.

### 3.2 Pendientes generados en la semana

**1. Corte controlado (PC-11) en el controlador de calle.** *Justificación:* depende de que antes se ejecute el runbook del certificado del sistema central, que quedó autorizado pero sin ejecutar. *Plan de acción:* ejecutar el runbook del certificado, repartir la autoridad certificadora al controlador de laboratorio y luego al de calle, quitar la línea provisional del archivo de nombres y el certificado instalado a mano, sincronizar la identidad del controlador de calle con "Sincronizar CID activo" (nunca con "Reemplazar"), con el equipo de campo presente y los respaldos ya tomados, y validarlo contra su línea base. *Resultado esperado:* PC-11 cerrada en calle, o detenida con la causa documentada.

**2. Condiciones antes de activar la versión nueva en calle.** *Justificación:* con los ajustes de rojo de 1 segundo que tienen sus dos escenarios compilaría un ciclo de 62 segundos y no de 60; el parche del compilador toca temporización de seguridad y falta que lo revise su dueño; y el chequeo de salud da "sano" revisando solo el código HTTP. *Plan de acción:* poner esos ajustes en 0, repetir la validación física, conseguir la revisión del parche y corregir el chequeo de salud. *Resultado esperado:* una activación en calle que parta de condiciones verificadas.

**3. Alcance de detección que no se cubrió.** *Justificación:* el controlador de laboratorio tiene un solo detector configurado, el castigo "extender el siguiente" solo se probó en simulador y "escenario específico" no se corrió, los castigos por reparto no los soporta el script y el flujo de estado por WebSocket y el modo estricto no se probaron en el controlador real. Siguen además abiertas dos observaciones de diseño, una detección que empieza dentro de la propia entrada del escenario no sirve a ese escenario ni se guarda, y el latch no caduca por tiempo. *Plan de acción:* ampliar la configuración de laboratorio, repetir con los demás castigos y otros detectores. *Resultado esperado:* validación de detección completa para los castigos y detectores que se usarán en calle.

### 3.3 Actividades cercanas

Según el cronograma, la Semana 11 sigue con las Actividades 9, 10 y 11, hasta la semana 12:

- **Actividad 9.** Validar cambios mediante pruebas controladas y en campo, asegurando que las modificaciones implementadas no afecten la operación del sistema antes de su despliegue.
- **Actividad 10.** Implementar herramientas de monitoreo y diagnóstico, como registros de eventos (logs), trazas del sistema y métricas de desempeño que faciliten el análisis operativo.
- **Actividad 11.** Coordinar el despliegue de nuevas versiones del sistema, asegurando una implementación controlada en los nodos activos y minimizando riesgos durante la actualización.

Los pendientes de esta semana caen directamente en ellas. La validación de otros detectores y castigos y la validación física en calle corresponden a la Actividad 9. El chequeo de salud que revisa el cuerpo de la respuesta y la sincronización del historial local corresponden a la Actividad 10. El certificado, la sincronización del controlador de calle y la continuación del corte corresponden a la Actividad 11. El resultado esperado es llegar a la semana 12 con el corte cerrado también en calle y con la detección validada para lo que se usará ahí.

---

## 4. Autorreflexión personal: problemas enfrentados, aprendizajes y desempeño

### 4.1 El patrón que más se repitió esta semana

Una prueba que no puede fallar no prueba nada. Por eso cada validación de detección llevó un ciclo de control, precondiciones que invalidan ejecuciones contaminadas, fallas inyectadas para comprobar que el script sí las detecta y un contraste con una fuente independiente. Lo mismo se vio en el cierre del release, una corrida aprobada no servía si declaraba una versión distinta a la que corría.

### 4.2 Errores propios y lo que aprendí de cada uno

El principal fue que la corrida de PC-09 del lunes quedó con una versión del sistema central declarada que no era la que realmente corría. Se detectó al consolidar el cierre, y hubo que repetir la prueba sobre el release activo. El aprendizaje es capturar la versión real junto con cada evidencia, en el momento, y no reconstruirla después.

Otro fue leer el estado de dos controladores con dos comandos seguidos sin etiquetar la salida. Uno regresó vacío porque el equipo estaba apagado mientras se instalaba algo, y sin etiqueta no era claro a cuál equipo correspondía cada resultado; se confirmó después con un comando etiquetado. Desde entonces conviene etiquetar cada comando con el equipo al que va dirigido.

### 4.3 Aprendizajes profesionales y consideraciones éticas

- Cuando la prueba contra el release activo falló por el certificado, quedó registrada como rechazada con su causa, y la aprobada posterior indica expresamente que se hizo con la verificación del certificado desactivada. No se dejó solo la que salió bien.
- El token de la ingesta se entregó por privado, no se anexa y se dejó el aviso de que no debe rotarse sin avisar, porque los dos controladores tienen que coincidir.
- Lo no comprobado se informó de esa forma, los demás castigos aparecen como pendientes, no como cumplidos.

### 4.4 Sobre el cronograma

Esta semana el cronograma coincidió con el trabajo real en las Actividades 9, 10 y 11, con distinto avance en cada una, y aún quedan las semanas 11 y 12 para completarlas. La Actividad 9 todavía no tiene validación en campo, y la 10 depende de trabajo de bitácoras que se hizo poco esta semana. El seguimiento a incidencias (Actividad 7) volvió a ocupar parte de la semana aunque el cronograma lo programó para las semanas 3 a 5. La junta de reglas por crucero apunta a las Actividades 8 y 14, ya fuera de su ventana o todavía lejos, y conviene ubicarla en el cronograma cuando se formalice el levantamiento.

### 4.5 Desempeño, bienestar y acciones que voy a mantener

Fue una semana completa de trabajo, a diferencia de la semana 8, que tuvo solo dos días, y concentró el cierre del release, tres validaciones y una junta. Lo que ayudó fue dejar el reporte de cada prueba escrito el mismo día en que se corrió, en lugar de reconstruirlo al final, y apoyarme en scripts con criterios definidos de antemano y no en juicio del momento.

Acciones que se van a sostener:

- Registrar la versión real desplegada junto con cada evidencia en el momento de la corrida.
- Diseñar cada prueba con un ciclo de control y precondiciones que invaliden las ejecuciones contaminadas.
- Etiquetar cada comando con el equipo al que va dirigido y cruzar la evidencia por identificador y no por hora.
- Mantener el registro de lo no verificado como pendiente, con su razón, hasta comprobarlo.
