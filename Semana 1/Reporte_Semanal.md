# Reporte Semanal

| Campo | Dato |
|---|---|
| **Nombre** | Zuleyca Guadalupe Balles Soto |
| **Matrícula** | A01741687 |
| **Programa Académico** | ITC |
| **Empresa** | Dirección de Vialidad y Semaforización del Ayuntamiento de Hermosillo |
| **Tutor Académico** | Mario Durán Vega |
| **Fecha** | 23 de agosto de 2026 |
| **Periodo reportado** | Semana 1 - 17 al 21 de agosto de 2026 |

> **Nota sobre confidencialidad.** El proyecto está sujeto a un acuerdo de confidencialidad. Este reporte describe metodología, decisiones técnicas y aprendizajes obtenidos, omitiendo identificadores de infraestructura, rutas de despliegue, credenciales, hallazgos de configuración escalados por canal interno y cualquier dato que permita ubicar equipos o instalaciones específicas.

---

## 1. Descripción de la fase u objetivos desarrollados en la semana declarados en el cronograma

### 1.1 Contexto del proyecto

El proyecto es un sistema para gestionar y monitorear semáforos inteligentes. Tiene dos partes, una plataforma web desde donde se administra y se monitorea el sistema, y el firmware que corre en los controladores instalados en la intersección. Yo trabajo en las dos, y también en el despliegue de versiones nuevas a los controladores.

Dos términos que uso a lo largo del reporte: llamo **nodo** a cada intersección con su controlador instalado, y **ventana de detección** a la pantalla de la plataforma web donde se configura qué cámara va relacionada con cada semáforo y sobre qué parte de la imagen se detectan los vehículos.

### 1.2 Actividades declaradas en el cronograma para la Semana 1

Según el cronograma, la Semana 1 corresponde a las Actividades 1 y 2:

- **Actividad 1.** Analizar el estado actual del sistema en campo, mediante la revisión de la infraestructura instalada, componentes de hardware y software activos, así como la identificación de puntos críticos que afecten su funcionamiento.
- **Actividad 2.** Evaluar el desempeño del sistema en operación real, considerando sincronización entre intersecciones, tiempos de ciclo, desfases y posibles fallos detectados durante su ejecución.

### 1.3 Objetivos efectivamente perseguidos

**Actividad 1 — en curso.** El diagnóstico del estado actual se realizó y produjo una lista de puntos críticos concretos, entre ellos, ausencia de un mecanismo verificable para actualizar o revertir componentes del firmware; un defecto en el script de pruebas de cierre del sistema que restauraba la referencia a la versión activa pero no la de respaldo, dejando a los equipos sin capacidad de reversión; versiones huérfanas de pruebas anteriores presentes en ambos equipos; archivos de configuración de operación que no estaban bajo control de versiones; y unidades de servicio en el repositorio apuntando a rutas que ningún equipo utiliza.

**Actividad 2 — cubierta parcialmente.** Se revisó el desempeño del controlador en operación. Uno de los equipos corría con una zona horaria distinta a la que declara el código y ninguno tenía sincronización de reloj activa. Se midió la deriva del equipo de campo antes de tocarlo y los dos quedaron sincronizados.

La parte de la actividad que compara intersecciones entre sí no se puede hacer todavía. Hoy hay un solo nodo funcional instalado, y el controlador de laboratorio es un espejo de ese mismo nodo. Sin una segunda intersección no hay contra qué comparar tiempos de ciclo ni desfases. Queda declarado como limitación de lo que hay instalado hoy, no como pendiente de esta semana.

**Actividades anticipadas respecto al cronograma.** Parte del trabajo de la semana correspondió a las Actividades 3, 11, 12 y 13, programadas originalmente para semanas posteriores:

- Evolución de un componente de ejecución del firmware (Actividad 3, semanas 3–5).
- Mecanismo de despliegue controlado de nuevas versiones en los nodos, minimizando riesgo durante la actualización (Actividad 11, semanas 10–12).
- Modos de fallo controlado: verificación posterior al despliegue y reversión automática cuando la verificación no se cumple (Actividad 12, semanas 13–14).
- Mejoras a la arquitectura del sistema, en la parte del rediseño de la ventana de configuración de detección (Actividad 6, semanas 6–9).
- Documentación de cambios y arquitectura (Actividad 13, semana 15).

Se incorporaron además, de forma parcial, la Actividad 10 en su parte de registros de eventos y trazas, y la Actividad 9 como práctica de validación mediante pruebas controladas antes de cualquier despliegue.

### 1.4 Relación entre el cronograma y el trabajo de la semana

El cronograma reparte quince actividades a lo largo de dieciséis semanas suponiendo una secuencia lineal, primero diagnosticar, luego mejorar, luego desplegar, al final documentar. En la práctica, sobre un sistema que ya está operando, varias de esas actividades corren al mismo tiempo todas las semanas, y cuál domina depende de lo que el sistema pida esa semana.

Lo que ordenó el trabajo de estos días salió del entregable de la Actividad 1. El diagnóstico encontró que el sistema en operación no contaba con forma verificable de actualizar ni de revertir un componente. Concretamente, el gestor de servicios reporta éxito en cuanto el comando de reinicio se ejecuta, no cuando el proceso queda funcionando, de modo que una actualización que dejaba el servicio muerto se registraba como exitosa y nada lo advertía. A eso se sumó el defecto que dejaba a los equipos sin capacidad de reversión.

Ese hallazgo invierte la prioridad. Con un sistema instalado en vía pública, seguir acumulando diagnóstico mientras no existe manera segura de aplicar ni de deshacer una corrección significa producir hallazgos que no se pueden accionar, cualquier intervención derivada del diagnóstico habría sido más riesgosa que el problema que pretendía resolver.

Se decidió adelantar la construcción del mecanismo de despliegue controlado y de fallo controlado, aplicándolo primero a un componente no crítico, donde una falla no afecta ningún semáforo, para después extenderlo a los módulos que manejan el ciclo semafórico con el mecanismo ya probado. En paralelo avanzó el rediseño de la ventana de configuración de detección (Actividad 6).

La documentación (Actividad 13) se adelantó de manera deliberada y no como sustituto de trabajo técnico, cada subtarea cierra con un reporte escrito. En un sistema en operación, un cambio sin registro de por qué se hizo es un cambio que nadie puede revertir con criterio.

El calendario del proyecto refuerza lo anterior. Con la puesta en operación de un corredor completo en cuestión de semanas, las próximas van a estar ordenadas por lo que ese despliegue exija, no por el reparto original de dieciséis semanas.

Cabe aclarar el alcance de todo esto, nada de lo que hice esta semana cae fuera de las quince actividades declaradas. Lo que cambia no es el alcance del proyecto, es el orden y la concurrencia. Por eso cada reporte va a declarar sobre qué actividades se trabajó y por qué, en lugar de afirmar un alineamiento semana por semana que no correspondería a lo trabajado.

---

## 2. Descripción del desarrollo de las actividades semanales (metodología, recursos, procedimientos)

### 2.1 Metodología

El trabajo se organiza en subtareas con criterio de cierre explícito, definido por escrito antes de comenzar. Al terminar cada una se redacta un reporte de cierre que documenta el objetivo, lo entregado, el cumplimiento punto por punto de los criterios, los defectos encontrados durante la ejecución y los pendientes que quedan abiertos. Esta semana se produjeron cinco de estos reportes, que constituyen la evidencia documental de las actividades.

La práctica de verificación más determinante es ejecutar en tres entornos antes de cerrar cualquier subtarea: el contenedor de desarrollo sobre Linux, la máquina de trabajo sobre Windows y el hardware real en el equipo de laboratorio. La mayoría de los defectos encontrados no eran detectables en el entorno donde se escribió el código, porque ese entorno corre con privilegios elevados, sobre un solo sistema operativo y con las rutas siempre en el mismo lugar.

Toda corrección se acompaña de pruebas automatizadas, incluyendo pruebas negativas, por cada defecto corregido se escribe una prueba que vuelve a fallar si alguien revierte la corrección. El propósito es que un arreglo no pueda deshacerse por accidente sin que la suite lo señale.

Los cambios de riesgo se prueban primero en el equipo de laboratorio. El equipo instalado en campo no se interviene sin decisión explícita del equipo de trabajo, por tratarse de una instalación en operación.

### 2.2 Recursos y herramientas

- **Lenguajes y entornos:** Python y Bash para las herramientas de despliegue y el firmware del componente; JavaScript con una biblioteca de interfaz basada en componentes para la plataforma web.
- **Infraestructura:** gestor de servicios del sistema operativo, base de datos relacional para el backend y una base embebida local para la bitácora de actualizaciones, servidor web.
- **Hardware:** los controladores del sistema; uno en laboratorio para pruebas y otro instalado en operación.
- **Proceso:** control de versiones con convención de mensajes de commit verificada automáticamente, y validación de compilación de cada archivo entregado antes de integrarlo.

### 2.3 Procedimientos seguidos y actividades realizadas

**Activación transaccional con verificación posterior.** El problema principal es que el gestor de servicios reporta éxito en cuanto el comando de reinicio se ejecuta, no cuando el proceso queda funcionando, un proceso que muere medio segundo después produce un reinicio aparentemente exitoso. La solución fue que el propio proceso publique su identidad (versión cargada, identificador de proceso y una marca de tiempo que se refresca periódicamente) y que la herramienta de activación sondee después del reinicio hasta comprobar cuatro condiciones: que el servicio siga activo, el identificador de proceso haya cambiado, la versión publicada sea la que se acaba de activar, y la marca de tiempo esté vigente. Si alguna falla, la herramienta restaura el estado anterior por sí sola y lo anota en la bitácora.

**Corrección de un defecto detectado al validar en hardware.** La reversión automática restauraba la versión anterior sin comprobar que esa versión sí pudiera verificarse. Al probarlo en laboratorio, revertir hacia una versión previa al mecanismo dejó el servicio muerto, el sistema quedaba peor que antes de intentar. Se corrigió agregando una guarda que rechaza la operación antes de mover algo y explica el motivo, en lugar de intentarla y fallar a la mitad.

**Verificación del arranque tras reinicio físico del equipo.** Era el último criterio de aceptación pendiente. Se reinició el equipo de laboratorio y el servicio quedó operativo sin intervención y sin reintentos. La bitácora del arranque mostró además que el componente se reconectó por su cuenta cuando el módulo del que depende todavía no estaba listo, que es el comportamiento buscado y que no se había podido observar en un arranque real.

**Publicación del estado de ejecución hacia la plataforma web.** El requisito era explícito en no crear una segunda interpretación del estado. En lugar de consultar el gestor de servicios desde el servicio web, se agregó un subcomando a la herramienta que ya toma esa decisión, de modo que la interfaz y el mecanismo de reversión leen exactamente la misma fuente.

**Cierre del bloque de diseño de la ventana de detección.** Se entregó un prototipo navegable con cinco escenarios simulados que reproducen los casos difíciles: nodo sin configurar, configuración parcial, configuración completa, conflicto heredado y cámara sin señal. La simulación quedó separada visualmente del producto, para que quien revise la pantalla no confunda un control de prueba con una función real. El prototipo se revisó con el responsable del proyecto y el rediseño quedó validado ahí mismo.

**Primera fase del reemplazo de la ventana: lectura real sin escritura.** La ventana consume la configuración real del equipo y no escribe nada; los controles de guardado quedan deshabilitados mostrando la razón. Se verificó punto por punto contra los ocho criterios de la fase.

**Plan de reemplazo por fases.** Se documentaron cinco fases, cada una con entregable utilizable, criterio de verificación y consecuencia si falla. El orden se fijó para que la escritura de configuración de detección sea lo último, de modo que un fallo temprano no pueda dejar una intersección mal configurada.

### 2.4 Evidencia

El respaldo de lo descrito son los cinco reportes de cierre que redacté durante la semana, con el resultado de las suites de pruebas y de las verificaciones hechas sobre el equipo de laboratorio. Son documentos internos del proyecto, así que no se anexan completos por el acuerdo de confidencialidad.

Se anexan dos de ellos en versión con información omitida:

- **Anexo A.** Activación y reversión verificada, por parte de firmware.
- **Anexo B.** Ventana de detección, fase 1 de lectura real, por parte de plataforma web.

Los tres restantes pueden entregarse en el mismo formato si el asesor los requiere.

---

## 3. Listado de las actividades siguientes o pendientes no resueltas, con justificación

> Los pendientes que se listan son los generados dentro de la propia semana, más las dos actividades del cronograma que quedan parcialmente cubiertas.

### 3.1 Estado de las actividades declaradas para la semana

**Actividad 1 - Diagnóstico del estado actual.** Cubierta en su parte de software y de configuración de los equipos.

**Actividad 2 - Evaluación del desempeño en operación real.** Cubierta en lo que hoy se puede medir, zona horaria, sincronización de reloj y deriva del equipo de campo. La parte comparativa entre intersecciones no tiene objeto todavía, porque hay un solo nodo funcional instalado y el equipo de laboratorio lo espeja. *Justificación:* no es incumplimiento, es que hoy no existe una segunda intersección con la cual comparar. *Plan de acción:* retomarla cuando exista una segunda intersección en operación. *Resultado esperado:* mediciones comparables entre nodos reales.

### 3.2 Pendientes generados en la semana

**1. Despliegue del mecanismo de actualización en el equipo instalado en campo.** No se realizó por decisión deliberada del equipo, no por falta de avance. El equipo de campo opera una intersección real y el criterio acordado es agotar los casos en laboratorio antes de intervenirlo. *Plan de acción:* presentar la solicitud formal de autorización ahora que el ciclo completo (instalar, activar, revertir y sobrevivir a un reinicio físico) quedó verificado; el procedimiento está escrito y es una repetición de la secuencia probada. *Resultado esperado:* ambos equipos con el mismo mecanismo y con bitácora, de modo que sea posible saber qué versión corre en cada uno sin conectarse a ellos.

**2. Dos avisos de la interfaz sin verificar en pantalla.** Los avisos de versión desalineada y de procesos duplicados están cubiertos por pruebas, pero no se pudieron observar en el navegador porque en el momento de probar no existía ninguno de los dos escenarios. *Plan de acción:* provocar ambas condiciones deliberadamente en laboratorio y verificar el renderizado.

**3. Representación de nodos con configuraciones no cubiertas.** El croquis de la ventana es fijo a cuatro direcciones y el catálogo del sistema admite ocho, incluyendo diagonales. No se abordó porque hoy ningún nodo activo se encuentra en esa condición, lo cual se verificó por consulta a la base. La limitación quedó documentada junto con la medida de contención adoptada, de modo que incorporarlas más adelante sea un cambio de posiciones y no de lógica.

### 3.3 Actividades cercanas

**Lo inmediato, para la Semana 2.**

- **Terminar la implementación de la ventana de detección.** Faltan tres fases del plan. La segunda escribe la asignación de cámara a cada dirección semafórica, y hay que verificar que reasignar despoje efectivamente a la dirección anterior en base de datos y que una falta de permiso se muestre como mensaje y no como pantalla rota. La tercera escribe la región de interés, requiere confirmar antes con el equipo de firmware la semántica de un identificador hoy ambiguo en el código heredado, y se prueba primero en laboratorio porque es la primera en la que un error podría dejar mal configurada una detección real. La cuarta monta la ventana nueva en lugar de la actual, sin borrar archivos, de modo que revertir sea deshacer un commit.
- Cerrar lo que queda de periféricos, incluido el despliegue del mecanismo de actualización en el controlador instalado en campo, que es lo único del ciclo que sigue sin hacerse.
- Verificar los dos avisos que quedaron sin observarse, provocando ambas condiciones a propósito en laboratorio.
- **Semana 2 del cronograma.** Vuelve a declarar las Actividades 1 y 2. Con el diagnóstico cubierto y la parte comparativa sin objeto por ahora, el trabajo continúa sobre las Actividades 3 y 6, que ya se venían adelantando.

---

## 4. Autorreflexión personal: problemas enfrentados, aprendizajes y desempeño

### 4.1 El problema de fondo de esta etapa

El patrón que más ha definido mi aprendizaje es que los defectos no aparecen donde se escribe el código. El entorno de desarrollo corre con privilegios elevados, sobre un solo sistema operativo y con las rutas siempre en el mismo lugar; el hardware real y la máquina de trabajo exponen de inmediato lo que ese entorno oculta. De los defectos encontrados en esta etapa, la mayoría se manifestaron al ejecutar fuera del contenedor.

### 4.2 Errores propios y lo que aprendí de cada uno

**Cometí un error que yo mismo había documentado como riesgo.** La confusión entre dos identificadores distintos (el del inventario y el local del controlador) estaba señalada por escrito en dos documentos previos, redactados por mí, y aun así la cometí al escribir el componente: el resultado era pedir video de una cámara que el controlador no conoce. Lo que me hizo verlo no fue la revisión, sino haber preparado los datos de prueba de manera que los dos identificadores no coincidieran; la primera versión de esos datos los tenía iguales por casualidad, y con ellos el defecto era invisible. El aprendizaje es doble, documentar un riesgo no lo mitiga, y un escenario de prueba donde dos valores distintos coinciden por accidente no prueba que el código los distinga.

**Reporté como aprobadas cuatro pruebas que nunca se ejecutaron.** Habían quedado escritas después del punto de salida, de modo que la ejecución terminaba antes de alcanzarlas, y la suite reportó éxito. Aprendí que una suite que termina bien no garantiza que corrió lo que se le agregó al final, y que conviene comprobar el conteo de pruebas ejecutadas y no solo el resultado.

### 4.3 Aprendizajes profesionales y consideraciones éticas

Trabajar sobre un sistema que gobierna intersecciones reales cambia el peso de las decisiones técnicas, y varias de esta semana fueron en el fondo decisiones éticas más que de ingeniería.

- Decidí que la herramienta reporte los procesos ajenos que detecta en lugar de cerrarlos, y dejé una prueba que verifica que el código no contenga ninguna instrucción capaz de terminar un proceso. Cerrar algo que otra persona lanzó, sin saber por qué lo lanzó, es peor que avisar.
- Preferí que el mecanismo de reversión se niegue y explique el motivo antes que intentar una operación cuyo resultado no puede comprobar. Eso obliga a reportar honestamente que en cierta configuración no hay reversión disponible, que es incómodo de decir pero es la verdad; el mensaje optimista habría sido más cómodo y habría dejado a alguien creyendo que tiene una red de seguridad que no tiene.
- Sostuve la distinción entre "sin datos" y "libre" en la interfaz, incluso cuando complica el diseño. Un estado que se lee como tranquilizador cuando en realidad no hay información es una decisión con consecuencias en un sistema de control de tránsito.
- Al encontrar observaciones que no correspondían a mi tarea, las escalé por el canal interno y las documenté como asuntos separados, en lugar de resolverlas por mi cuenta sin autorización o dejarlas pasar por no ser mías. Aprendí a distinguir entre lo que me toca arreglar y lo que me toca reportar.
- Redacté este reporte cuidando el acuerdo de confidencialidad que firmé, puedo describir métodos, decisiones y aprendizajes sin exponer infraestructura, credenciales ni información sensible del cliente. Sostener eso frente a la necesidad de "demostrar" detalle es parte de la responsabilidad profesional.

### 4.4 Sobre el cronograma

La diferencia de esta semana está justificada, pues el trabajo que adelanté es el que más le interesa técnicamente al cliente, y conviene que reconozca que ese sesgo existió y que no fue la única razón.

Igualmente, las tareas salen de lo que el proyecto necesita en esa semana y eso no se sabe con meses de anticipación. Prometer en cada reporte un alineamiento que no podría cumplir sería mentira. Lo que sí puedo hacer es reportar con precisión sobre qué actividades del cronograma trabajé y por qué, aunque el reparto original diga otra cosa.

Lo que voy a cuidar es que esa diferencia se sostenga siempre en una razón del proyecto y no en lo que se me antoje trabajar. Adelantar algo porque el sistema lo exigía es una cosa, y adelantarlo porque quiero es otra.

### 4.5 Desempeño, bienestar y acciones que voy a mantener

La semana concentró cinco cierres y dos frentes de trabajo en paralelo, con verificaciones en hardware que no siempre se pueden agendar cuando conviene. Lo que evitó que eso se volviera desgaste fue definir el criterio de cierre por escrito antes de empezar cada tarea, sin ese criterio, una tarea puede reabrirse indefinidamente y siempre parece que falta algo.

Acciones concretas que quiero sostener:

- Escribir el criterio de cierre y la lista de pendientes antes de comenzar, no después.
- Reservar tiempo de verificación en hardware dentro de la tarea, en lugar de dejarlo para el final, que es donde se convierte en presión.
- Pedir decisión del equipo cuando un cambio afecta a otras personas, en lugar de asumirla. Los pendientes de esta semana que dependen de terceros no son retrasos, son decisiones que no me corresponde tomar solo.
- Separar la jornada de trabajo del tiempo de estudio y de descanso, porque la clase de error que cometí esta semana (repetir dos veces el mismo descuido) es exactamente el que aparece con fatiga acumulada.