# Reporte Semanal

| Campo | Dato |
| --- | --- |
| **Nombre** | Zuleyca Guadalupe Balles Soto |
| **Matrícula** | A01741687 |
| **Programa Académico** | ITC |
| **Empresa** | Dirección de Vialidad y Semaforización del Ayuntamiento de Hermosillo |
| **Tutor Académico** | Mario Durán Vega |
| **Fecha** | 19 de octubre de 2026 |
| **Periodo reportado** | Semana 9 - 12 al 16 de octubre de 2026 |

> **Nota sobre confidencialidad.** El proyecto está sujeto a un acuerdo de confidencialidad. Este reporte describe metodología, decisiones técnicas y aprendizajes obtenidos, omitiendo identificadores de infraestructura, rutas de despliegue, credenciales y cualquier dato que permita ubicar equipos o instalaciones específicas.

---

## 1. Descripción de la fase u objetivos desarrollados en la semana declarados en el cronograma

### 1.1 Contexto del proyecto

Los términos **nodo**, **ventana de detección**, **contrato de comunicación**, **sincronía**, **identidad de controlador**, y el **corte controlado (PC-11)** ya quedaron definidos en reportes anteriores. Se agregan tres para esta semana. El **contrato de configuración (v1)** es el formato único con el que el sistema central y el firmware del controlador describen un programa de luces, con sus fases, tiempos y horarios, de modo que lo que uno construye lo ejecute igual el otro; existen tres tipos de programa, **fijo**, **diario** y **semanal**. Una **migración** es un cambio versionado a la estructura de la base de datos, y el **registro de migraciones** (el ledger) es la lista de cuáles ya se aplicaron. Una **configuración de referencia oficial** es el conjunto de programas de prueba que mantiene el equipo del CSI, contra el que se compara al sistema central.

La semana giró en torno a aplicar en producción seis migraciones de la base de datos, con el despliegue fallido que siguió, y cerrar **PC-09** y PC-11, siendo la primera la prueba de que el sistema central y el CSI interpretan igual el contrato de configuración, ahora contra el despliegue real de ambos lados.

### 1.2 Actividades declaradas en el cronograma para la Semana 9

Según el cronograma, la Semana 9 corresponde a las mismas Actividades 5, 6 y 8 de las semanas anteriores, y es la última semana programada para ellas:

- **Actividad 5.** Supervisar la integración entre hardware y software, verificando la correcta comunicación con controladores, dispositivos en campo y sistemas embebidos.
- **Actividad 6.** Proponer e implementar mejoras en la arquitectura del sistema, orientadas a incrementar la estabilidad, escalabilidad y facilidad de mantenimiento.
- **Actividad 8.** Optimizar la lógica de control semafórico, ajustando ciclos, transiciones y sincronización entre intersecciones para mejorar el flujo vehicular.

### 1.3 Objetivos efectivamente perseguidos

**Actividad 5 - cubierta.** Se cerró la PC-09 contra el despliegue y con la configuración de referencia oficial del CSI. El resultado final, del 15 de octubre, fue aprobado. Los tres tipos de programa válidos (fijo, diario y semanal) se interpretan igual en ambos lados, es decir, producen la misma huella. Los 19 casos inválidos se rechazan con el código y la categoría esperados, sin dejar configuración pendiente ni modificar la activa. Como requisito previo de PC-11 se respaldaron además la identidad y la configuración de los dos controladores. Detalle completo de la PC-09 en **Anexo B**.

**Actividad 6 - cubierta.** Se aplicaron en producción seis migraciones y el registro pasó de 35 a 41 aplicadas, sin pendientes, con respaldo previo y ensayo en una copia restaurada. El primer despliegue del código nuevo falló en la verificación de salud y se revirtió solo al release anterior. La causa fue un archivo con nombre distinto al que importa el código, y no las migraciones. El despliegue posterior al fix se verificó después (salud y disponibilidad en verde, y lectura de configuración de los dos controladores sin el error). Se hicieron además mejoras de estabilidad y mantenibilidad ligadas a la PC-09: la inspección de configuraciones conserva ahora las etiquetas de fase, avisa cuando una configuración trae una zona horaria distinta a la del nodo, y quedaron pruebas automáticas nuevas, cuatro en el CSI y una de ida y vuelta en el sistema central sobre los tres programas. Detalle completo en **Anexo A**.

**Actividad 8 - cubierta parcialmente.** El defecto más relevante de la PC-09 estaba en la lógica de transiciones del CSI. El compilador cobraba el tiempo de rojo posterior a fragmentos del programa que no cierran ningún grupo, y por eso los programas diarios y semanales se rechazaban con un error 422 de presupuesto de transición insuficiente. Ahora ese rojo se cobra solo cuando un grupo realmente pasa a rojo en ese límite. El cambio no pasa por alto las garantías, porque un fragmento por debajo del verde mínimo vehicular, o que sí cierra un grupo, se sigue rechazando. *Justificación de cobertura parcial:* lo que se hizo fue corregir y validar la lógica con la que se compila y se acepta un programa, no ajustar ciclos ni sincronía entre intersecciones, y no se modificó ningún ciclo en equipos de calle. Ese ajuste real sigue condicionado a la instalación de los controladores del corredor.

### 1.4 Relación entre el cronograma y el trabajo de la semana

Esta semana el cronograma y el trabajo realizado coincidieron en las tres actividades declaradas (5, 6 y 8). Al ser la última semana programada para ellas, la 5 y la 6 se dan por cubiertas, mientras que la 8 cierra con cobertura parcial.

**Actividades no programadas para esta semana.** El diagnóstico del despliegue fallido es seguimiento a incidencias (Actividad 7), que el cronograma programó para las semanas 3 a 5; se atendió porque dejaba al sistema con el código anterior sobre el esquema nuevo. El ensayo de migraciones y el despliegue a producción corresponden a coordinar despliegues (Actividad 11), programada para las semanas 10 a 12, de modo que se trabajó por delante del cronograma.

---

## 2. Descripción del desarrollo de las actividades semanales (metodología, recursos, procedimientos)

### 2.1 Metodología

El criterio de la semana fue validar contra el sistema desplegado y no contra una copia de pruebas, y no dar por cerrado nada que no se hubiera verificado. Los cambios a la base de datos se respaldaron y se ensayaron en una copia antes de tocar producción. La validación del contrato se corrió una y otra vez con el mismo procedimiento y con la configuración oficial sin variantes, dejando evidencia por corrida, de modo que cada rechazo señalara exactamente en qué paso divergían los dos lados. Cuando una diferencia podía resolverse de dos maneras (ajustar la prueba o ajustar el sistema), se documentó la decisión y su razón. En el reporte de incidente, lo reportado pero no comprobado se marcó como "sin verificar" y no como cumplido, hasta que se comprobó.

### 2.2 Recursos y herramientas

- **Infraestructura:** servidor de producción Linux, base de datos relacional con su runner de migraciones versionado, script de despliegue con verificación de salud y reversión automática al release anterior, y controlador con firmware con su servicio de validación de configuraciones.
- **Aplicación:** backend en Python, frontend en JavaScript con biblioteca de interfaz basada en componentes, y un script de validación de contrato que ejecuta la misma secuencia contra el sistema central y el CSI.
- **Verificación:** respaldos con `pg_dump` y listado de su contenido con `pg_restore`, comparación de huellas, pruebas automáticas, y un reporte de resultado por corrida con las versiones de ambos lados.

### 2.3 Procedimientos seguidos y actividades realizadas

**Migraciones a producción.** Se hizo un respaldo de la base y se verificó su contenido. Se restauró en una base temporal para el ensayo. Se clonó la rama principal y se revisó el SQL de las seis migraciones, que no contienen `DROP TABLE` ni `DELETE` y solo eliminan una vista, usan tablas temporales y quitan el valor por defecto de una columna de identidad de controlador. En la copia se aplicaron las seis. Después se hizo un segundo respaldo justo antes de aplicarlas en producción. Detalle en **Anexo A**.

**Despliegue fallido y reversión.** El script de despliegue pasó los pasos 1 a 9 y falló en el 10, la verificación de salud, en 10 de 10 intentos, y ejecutó la reversión al release anterior. El registro del sistema mostró que los procesos del release nuevo no arrancaban por un error de importación. En la rama principal el archivo tenía un sufijo de más en el nombre, y dos módulos lo importan sin ese sufijo. El mismo nombre estaba en las demás ramas de desarrollo. Mientras tanto, el release anterior quedó sirviendo sobre el esquema ya migrado y falló al leer la configuración de dos controladores. La causa es que ese release no define un campo que sí existe en el esquema nuevo, pero no se determinó por qué el error apareció solo después de cierta hora. El fix se mergeó y el despliegue posterior se verificó después.

**Qué dejó ver el incidente.** El chequeo de disponibilidad del sistema solo exigía migraciones hasta la 35, así que siguió en verde con el esquema en 41 y con el error activo. El ensayo había cubierto únicamente el SQL, y no hubo prueba de que el código nuevo importara bien antes del despliegue. Como el script de despliegue no aplica migraciones por diseño, el flujo fue migrar y luego desplegar, y al fallar el despliegue quedó el código viejo sobre el esquema nuevo. Esas tres observaciones son la base de las mejoras propuestas.

**PC-09, validación del contrato contra el despliegue real.** El script de validación ejecuta, contra el sistema central y el CSI desplegados, esta secuencia: (1) consulta la identidad del CSI, que incluye la versión del contrato, la identidad habilitada y el identificador detectado; (2) lee la configuración dos veces y compara las huellas, y revisa que el registro sea legible; (3) para cada uno de los tres programas válidos, valida en el CSI y luego inspecciona y construye en el sistema central, y compara la huella de ambos; (4) envía 19 casos inválidos (estructura, versión, grupo, duración peatonal, programa, huecos, solapes, semana incompleta, zona inválida y reglas de penalización) y comprueba el código esperado y que no quede nada pendiente ni cambie lo activo; (5) prueba un identificador de controlador ajeno, que debe dar 409 sin efectos. Las corridas anteriores fallaron por tres causas distintas:

1. **Compilador del CSI.** Cobraba el rojo posterior a fragmentos que no cierran ningún grupo, por lo que daily y weekly se rechazaban con 422. Se corrigió para cobrarlo solo si un grupo realmente pasa a rojo en ese límite, y se agregaron cuatro pruebas. Se quitó además de dos archivos de pruebas el caso que esperaba el rechazo de daily y weekly, que dejó de ser correcto.
2. **Etiquetas de fase en el sistema central.** La inspección descartaba las etiquetas de los grupos de fases, y por eso el programa fijo reconstruido tenía una huella distinta. Ahora la inspección las devuelve, el constructor y el frontend las llevan, y si un cliente no las manda se recuperan del último programa guardado del controlador. La prueba de ida y vuelta falla sin el cambio y pasa con él.
3. **Zona horaria.** El sistema central construye con la zona horaria del nodo, y la configuración oficial traía la de la Ciudad de México en el programa semanal, por lo que su huella difería. Se decidió que **el nodo manda**, de modo que la configuración oficial usa la zona del nodo en los tres casos, igual en el CSI y en el sistema central, y la inspección avisa cuando un programa v1 trae otra zona.

Con esto se hicieron tres corridas, la primera, con la configuración oficial anterior, quedó rechazada solo por la huella del semanal. La segunda, con la zona del nodo en el semanal, quedó aprobada. La tercera, con la configuración oficial corregida y sin variantes, quedó aprobada. Las tres quedaron registradas en el reporte de cierre, incluida la rechazada.

**Corrección de un criterio de cierre.** Al verificar, el tercer criterio decía que el sistema central y el CSI "reportan el mismo identificador de controlador". Eso es incorrecto por diseño, porque el identificador administrativo (propiedad del sistema central) y el identificador operativo del CSI son distintos. El criterio quedó redactado como que el identificador administrativo está ligado al identificador operativo, y se comprobó con el 409 del identificador ajeno.

**Preparación del corte controlado (PC-11).** Dentro del periodo, la preparación se limitó a un requisito del checklist, se respaldaron la identidad, la configuración y el archivo de identidad del controlador de laboratorio y calle.

### 2.4 Evidencia

El respaldo de lo descrito son dos reportes técnicos redactados durante la semana, el reporte de incidente de las migraciones y el despliegue fallido, y el reporte de cierre de la PC-09, con la evidencia de cada corrida guardada en la carpeta de evidencia del repositorio. Son documentos internos del proyecto, así que no se anexan completos por el acuerdo de confidencialidad. 

Se anexan ambos en versión con la información sensible omitida:

- **Anexo A.** Migraciones a producción y despliegue fallido: cronología, causas, criterios verificados y hallazgos.
- **Anexo B.** PC-09, cierre del contrato de configuración y ida y vuelta en el despliegue: resultados, defectos y correcciones, criterios y pendientes.

---

## 3. Listado de las actividades siguientes o pendientes no resueltas, con justificación

### 3.1 Estado de las actividades declaradas para la semana

**Actividad 8, optimizar la lógica de control semafórico.** Cubierta parcialmente. *Justificación:* esta semana se corrigió y se validó la lógica con la que se compila y se acepta un programa de luces, pero no hay ajustes reales de ciclos ni de sincronía entre intersecciones mientras los controladores del corredor no estén instalados. *Plan de acción:* al ser esta la última semana programada para la Actividad 8, se propone retomar el ajuste real de ciclos dentro de la validación en campo (Actividad 9), usando como base los programas y el contrato ya validados, en cuanto el hardware del corredor esté instalado. *Resultado esperado:* primer ajuste real de ciclos sobre un corredor coordinado, aplicado con el contrato ya probado. Las Actividades 5 y 6 se cubrieron.

**Estado del corte controlado (PC-11)** Es posterior al periodo, pero cierra un pendiente de la semana 8 y se informa a manera de avance. Se desplegó el canario (una versión nueva del firmware del CSI en un solo controlador, el de laboratorio) y se ensayó la reversa a la versión anterior y el regreso, sin perder identidad ni configuración. La PC-09 se repitió sobre el canario con el mismo resultado (tres casos válidos con la misma huella, 19 rechazos y el 409 del identificador ajeno), y la PC-10 con parcial fue aceptada. El corte quedó cerrado en el controlador de laboratorio con la excepción de que no se detuvieron formalmente las escrituras de Sincronía durante el corte, porque el canario fue un solo controlador.

### 3.2 Pendientes generados en la semana

**1. Corte controlado (PC-11) en el controlador de calle y cierre de sus pendientes.** Pendiente desde la semana 8; el controlador de laboratorio ya quedó cerrado. *Justificación:* falta ejecutar el runbook del certificado del SGSI; después repartir la CA a los controladores, sincronizar la identidad del controlador de calle (con "Sincronizar CID activo", nunca "Reemplazar", con el equipo de campo presente y los respaldos ya tomados) y validarlo contra su línea base. Antes de activar la versión nueva en calle hay que poner en 0 los overrides de rojo de 1 s de sus dos escenarios, porque compilaría un ciclo de 62 s en lugar de 60 s, y repetir la validación física con ventana autorizada. *Plan de acción:* seguir ese orden. Además, mejorar el script de salud del CSI para que revise el cuerpo de la respuesta y no solo el código HTTP. *Resultado esperado:* PC-11 cerrada en el controlador de calle, o detenida con la causa documentada.

### 3.3 Actividades cercanas

Según el cronograma, la Semana 10 inicia las Actividades 9, 10 y 11, que se mantienen hasta la semana 12:

- **Actividad 9.** Validar cambios mediante pruebas controladas y en campo, asegurando que las modificaciones implementadas no afecten la operación del sistema antes de su despliegue.
- **Actividad 10.** Implementar herramientas de monitoreo y diagnóstico, como registros de eventos (logs), trazas del sistema y métricas de desempeño que faciliten el análisis operativo.
- **Actividad 11.** Coordinar el despliegue de nuevas versiones del sistema, asegurando una implementación controlada en los nodos activos y minimizando riesgos durante la actualización.

El corte controlado ya abre las Actividades 9 y 11 de la semana 10. Lo concreto es que los pendientes de esta semana caen directamente en ellas. Repetir la PC-09 tras la revisión del parche y la validación física del corte en el controlador de calle corresponden a la Actividad 9. Hacer que la disponibilidad revise el registro completo y dejar rastro visible de los fallos corresponde a la Actividad 10. La prueba de importación antes de activar un release, el orden de migración y despliegue y la continuación del corte controlado (PC-11) en el controlador de calle corresponden a la Actividad 11. El resultado esperado es llegar a la validación en campo con un despliegue que se verifica antes de activarse.

---

## 4. Autorreflexión personal: problemas enfrentados, aprendizajes y desempeño

### 4.1 El patrón que más se repitió esta semana

Un ensayo parcial da una seguridad parcial. El ensayo de migraciones salió limpio y daba confianza, pero solo había probado el SQL, y la falla vino de un nombre de archivo que ese ensayo no podía ver. El chequeo de disponibilidad estuvo en verde mientras el sistema fallaba, por la misma razón, revisaba una parte y no el conjunto. Repetir exactamente el mismo procedimiento contra el despliegue real, con evidencia por corrida, permitió ver en qué paso divergía cada huella, y no detenerse en el primer defecto.

### 4.2 Errores propios y lo que aprendí de cada uno

El principal fue ensayar solo el SQL de las migraciones y no comprobar que el código nuevo importara antes de migrar. El efecto fue que durante al menos seis minutos la lectura de configuración de dos controladores falló en el release anterior, con 24 apariciones del error. Se revirtió bien, pero el orden de migrar y luego desplegar dejó el esquema adelantado al código. El plan del corte controlado (PC-11) que redacté la semana anterior ya incluía probar los artefactos a desplegar y arrancar la versión anterior contra el esquema nuevo, que son justo las pruebas que habrían detectado esta falla, y esta ventana no las incluyó. El aprendizaje es que el ensayo debe incluir arrancar el código nuevo contra la copia migrada, y que lo que ya escribí en un plan debe aplicarse también a las ventanas más pequeñas y no solo al corte grande.

Por último, el tercer criterio de cierre de la PC-09 estaba mal redactado desde el diseño, porque suponía que dos identificadores debían ser iguales cuando por diseño no lo son. Lo detectó la verificación contra el sistema real, y es una razón para revisar los criterios contra el modelo antes de usarlos como prueba.

### 4.3 Aprendizajes profesionales y consideraciones éticas

- Ante la diferencia de zona horaria se documentó la decisión (el nodo manda) y su razón, y las tres corridas quedaron registradas, incluida la rechazada, en lugar de dejar solo la que salió bien.
- Lo no verificado se informó como no verificado. El merge del fix y el despliegue posterior aparecen como "reportado" y "sin verificar" en el reporte de incidente, y no como cumplidos; solo se cerraron después de comprobarlos.
- En el análisis del SQL se verificó antes de aplicar que no hubiera operaciones destructivas sobre datos, y se dejó claro que no se comparó el contenido de las tablas antes y después más allá del estado del runner.

### 4.4 Sobre el cronograma

Esta semana el cronograma coincidió con el trabajo real en las Actividades 5, 6 y 8, exceptuando que la 8 avanzó en validación de la lógica de transiciones y no en ajuste de ciclos, por depender de hardware que aún no está instalado; la Actividad 8 acumula así cobertura parcial en la semana 7, ninguna en la 8 (solo dos días de trabajo) y parcial en esta. El despliegue a producción y el seguimiento a incidencias (Actividades 11 y 7) volvieron a ocupar parte de la semana aunque el cronograma los programa en otras semanas.

### 4.5 Desempeño, bienestar y acciones que voy a mantener

Esta semana me costó el ritmo tras llegar el lunes del viaje a Nueva York, por el cambio de horario y las complicaciones del regreso, y sentí que avancé más lento de lo normal. Aun así, los dos frentes importantes se hicieron con respaldo, ensayo y evidencia, y lo que ayudó fue apoyarme en procedimientos escritos y repetibles (el ensayo de migraciones y el script de validación) en lugar de depender de lo que recordara en el momento.

Acciones que se van a sostener:

- Ensayar las migraciones en una copia que incluya arrancar el código nuevo, y no solo el SQL.
- Revisar los criterios de cierre contra el modelo real antes de darlos por buenos.
- Guardar los respaldos en dos lugares antes de dar por cerrada una ventana de migración.
