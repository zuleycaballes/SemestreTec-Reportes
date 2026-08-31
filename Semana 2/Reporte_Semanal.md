# Reporte Semanal

| Campo | Dato |
|---|---|
| **Nombre** | Zuleyca Guadalupe Balles Soto |
| **Matrícula** | A01741687 |
| **Programa Académico** | ITC |
| **Empresa** | Dirección de Vialidad y Semaforización del Ayuntamiento de Hermosillo |
| **Tutor Académico** | Mario Durán Vega |
| **Fecha** | 30 de agosto de 2026 |
| **Periodo reportado** | Semana 2 - 24 al 28 de agosto de 2026 |

> **Nota sobre confidencialidad.** El proyecto está sujeto a un acuerdo de confidencialidad. Este reporte describe metodología, decisiones técnicas y aprendizajes obtenidos, omitiendo identificadores de infraestructura, rutas de despliegue, credenciales, hallazgos de configuración escalados por canal interno y cualquier dato que permita ubicar equipos o instalaciones específicas.

---

## 1. Descripción de la fase u objetivos desarrollados en la semana declarados en el cronograma

### 1.1 Contexto del proyecto

Los términos que se usan en este reporte, **nodo** para una intersección con su controlador instalado, y **ventana de detección** para la pantalla donde se configura qué cámara corresponde a cada semáforo y sobre qué parte de la imagen se detectan los vehículos, ya quedaron definidos en el reporte de la Semana 1.

### 1.2 Actividades declaradas en el cronograma para la Semana 2

Según el cronograma, la Semana 2 corresponde a las Actividades 1 y 2, las mismas que la Semana 1:

- **Actividad 1.** Analizar el estado actual del sistema en campo, mediante la revisión de la infraestructura instalada, componentes de hardware y software activos, así como la identificación de puntos críticos que afecten su funcionamiento.
- **Actividad 2.** Evaluar el desempeño del sistema en operación real, considerando sincronización entre intersecciones, tiempos de ciclo, desfases y posibles fallos detectados durante su ejecución.

Aparte de eso se tenía planeado, por acuerdo con el equipo y no por cronograma, retomar la depuración de un componente de aforo que sigue en desarrollo por otra parte del equipo.

### 1.3 Objetivos efectivamente perseguidos

**Actividad 1 - cubierta.** Al realizar un diagnóstico de CPU en un controlador de campo, se encontraron puntos críticos. El ciclo principal del componente de visión no tiene un límite de velocidad activo por defecto así que corre a la máxima capacidad del hardware, y una de las dos formas de leer video no tiene ningún control de tasa.

**Actividad 2 - cubierta parcialmente.** El mismo diagnóstico evaluó desempeño en operación real y descartó que el consumo alto fuera un problema de temperatura o de hardware limitado. También se supo, al preguntar con el equipo, que la diferencia de consumo contra el equipo de laboratorio se debía a una optimización que alguien había aplicado ahí sin avisar, no a una falla del equipo de campo.

**Actividades anticipadas respecto al cronograma.** Buena parte del trabajo de la semana correspondió a actividades programadas para semanas posteriores:

- Mejora continua de módulos de firmware (Actividad 3, semanas 3 a 5): los cuatro cierres del módulo de actualización de periféricos.
- Mecanismos de seguridad y modos de fallo controlado (Actividad 12, semanas 13 y 14): inmutabilidad de versiones instaladas, diagnóstico de salud del componente en tiempo real, y verificación de que el equipo se recupera solo después de un reinicio físico.
- Mejoras a la arquitectura del sistema, en la ventana de configuración de detección (Actividad 6, semanas 6 a 9): cierre de la fase que guarda la asignación cámara-dirección, y resolución del conflicto de definición que tenía detenida la siguiente fase.
- Documentación de cambios y arquitectura (Actividad 13, semana 15): cada cierre de esta semana quedó por escrito.

**Aforo, sin avance.** No se pudo debuggear porque la parte que faltaba programar de ese componente sigue sin terminarse, no hay nada que depurar todavía sobre algo que no existe.

### 1.4 Relación entre el cronograma y el trabajo de la semana

El cronograma sigue marcando diagnóstico para la Semana 2, y ese diagnóstico se cubrió con el CPU. El resto de la semana quedó ocupado por actividades que en el cronograma corresponden a semanas posteriores, cerrar el módulo de periféricos y avanzar la ventana de detección, porque es lo que el proyecto necesitaba en ese momento.

El aforo, que sí estaba planeado retomar, se quedó frenado por una dependencia externa al alcance del trabajo. En vez de forzarlo, el tiempo se usó en trabajo que sí estaba disponible, cerrar pendientes atrasados y avanzar la ventana de detección.

El punto donde más se nota la diferencia con lo planeado es la ventana de detección. Ahí se encontró una contradicción en cómo se interpreta la dirección de una región de detección, y se decidió detener esa parte del trabajo en vez de asumir cuál lectura era la correcta y seguir avanzando. Esa pausa no estaba en el plan, pero evitó construir sobre una decisión que no correspondía tomar de forma unilateral.

---

## 2. Descripción del desarrollo de las actividades semanales (metodología, recursos, procedimientos)

### 2.1 Metodología

Cada tarea de la semana cerró con un criterio definido antes de empezar y con un reporte escrito al terminar. Los cambios de firmware se prueban primero en el equipo de laboratorio, el controlador de campo no se toca sin que el equipo lo decida en conjunto, porque ya está operando una intersección real. Cuando se encontró un punto que no era técnico sino de definición, en la ventana de detección, se documentó con evidencia de código y se escaló en vez de decidirlo unilateralmente.

### 2.2 Recursos y herramientas

- **Lenguajes y entornos:** Python para el backend y las herramientas de firmware, JavaScript con una biblioteca de interfaz basada en componentes para la parte web.
- **Infraestructura:** gestor de servicios del sistema operativo, base de datos relacional, servidor web.
- **Hardware:** los controladores del sistema, uno en laboratorio para pruebas y otros instalados en operación.
- **Proceso:** control de versiones con convención de mensajes de commit verificada automáticamente, y validación de cada archivo entregado antes de integrarlo.

### 2.3 Procedimientos seguidos y actividades realizadas

**Cierre de periféricos.** Cuatro puntos quedaron cerrados esta semana. El primero evita que una versión ya instalada pueda quedar sobrescrita con otro contenido sin que quede registro del cambio, ahora se compara el contenido real contra el nuevo antes de reemplazar nada. El segundo es un diagnóstico que junta configuración, estado del driver, la unidad del sistema y los procesos activos para saber si el componente está funcionando bien, en el camino se corrigió que la primera versión de este diagnóstico comparaba mal dos relojes distintos y reportaba fallas que no eran reales. El tercero corrige que la unidad del sistema tenía fijo a mano el usuario bajo el que corre el proceso, ahora se resuelve solo al arrancar. El cuarto confirma, con un reinicio físico real del equipo de laboratorio, que el componente vuelve a levantarse solo sin que nadie tenga que intervenir.

**Diagnóstico de CPU en un controlador de campo.** Fue solo lectura, sin tocar configuración ni reiniciar nada. Se encontró que el ciclo principal del componente de visión no tiene un límite de velocidad activo por defecto, corre a la máxima capacidad que el hardware permite, y que una de las dos formas de leer video no tiene ningún control de tasa. Se descartó que fuera un problema de temperatura o de hardware limitado. Quedó propuesta una mitigación, pendiente de autorización porque implica reiniciar un servicio en un equipo de campo. Al preguntar con el resto del equipo, salió que alguien había optimizado el equipo de laboratorio sin avisar, lo que explicaba por qué ese equipo y el de campo se veían tan distintos en consumo.

**Fase de guardar la asignación cámara-dirección.** Se verificó que reasignar una cámara libere efectivamente a la dirección anterior en base de datos, y que el caso de dos cámaras terminando en la misma dirección se reporte en pantalla en vez de que una desaparezca sin explicación.

**Hallazgo y escalamiento del conflicto de dirección.** Se encontraron dos formas contradictorias de interpretar qué significa la dirección de una región de detección, si describe de dónde llegan los vehículos o hacia dónde se dirigen. El código en producción sostenía una lectura y el diseño nuevo sostenía la otra, y si se elegía mal, las regiones hubieran quedado etiquetadas al revés sin ningún error visible, hasta que alguien construyera lógica de semáforos encima de esa lectura equivocada. Se documentó con evidencia directa de código y se escaló al responsable del proyecto, porque afectaba una convención que después usaría el módulo de semáforos. La respuesta confirmó que la lectura correcta es de dónde llegan los vehículos, y adelantó además hacia dónde va el modelo a futuro, identificar cada calle de llegada por su nombre más el sentido, porque la misma calle se llama igual en ambos sentidos. Eso desbloquea la siguiente fase, pero deja una tarea nueva, revisar si el diseño visual que ya se había validado antes se hizo bajo la lectura contraria.

### 2.4 Evidencia

El respaldo de lo descrito son los reportes de cierre redactados durante la semana, uno por cada punto de periféricos, el diagnóstico de CPU, el cierre de la fase de asignación cámara-dirección, y los documentos del conflicto de dirección con la respuesta recibida. Son documentos internos del proyecto, así que no se anexan completos por el acuerdo de confidencialidad.

Se anexan dos de ellos en versión con la información sensible omitida:

- **Anexo A.** Diagnóstico de consumo de CPU en un controlador de campo.
- **Anexo B.** Cierre de la verificación de arranque automático del módulo de periféricos.

Los demás pueden entregarse en el mismo formato si el asesor los requiere.

---

## 3. Listado de las actividades siguientes o pendientes no resueltas, con justificación

### 3.1 Estado de las actividades declaradas para la semana

**Aforo.** Sin avance. *Justificación:* la parte que faltaba programar de ese componente, a cargo de otra parte del equipo, sigue sin terminarse. *Plan de acción:* retomarlo en cuanto esa parte esté lista. *Resultado esperado:* poder empezar la depuración real.

**Ventana de detección, fase de guardar la región.** Detenida durante casi toda la semana por un punto de definición, resuelto al cierre de la semana. *Justificación:* no era una decisión técnica sino de dominio, había que confirmar con quien tiene la autoridad de decidirlo. *Plan de acción:* revisar el diseño visual bajo la convención ya confirmada antes de retomar la escritura hacia el controlador. *Resultado esperado:* región de detección guardándose de forma correcta y verificable.

### 3.2 Pendientes generados en la semana

**1. Revisar el diseño visual de la ventana de detección.** La convención de dirección que se confirmó esta semana es la contraria a la que se había usado para validar el diseño en una fase anterior. *Plan de acción:* revisar la disposición del croquis antes de retomar la escritura de regiones. *Resultado esperado:* diseño consistente con la convención confirmada, sin necesidad de tocar datos ya guardados porque todavía no existe ninguno con la convención equivocada.

### 3.3 Actividades cercanas

Según el cronograma, la Semana 3 corresponde a las Actividades 3, 4 y 7:

- **Actividad 3.** Coordinar el desarrollo y la mejora continua de los módulos de software, asegurando la correcta evolución de componentes de ejecución de firmware.
- **Actividad 4.** Gestionar tareas y prioridades del equipo de desarrollo de software, organizando actividades, asignando responsabilidades y dando seguimiento al cumplimiento de objetivos dentro de los tiempos establecidos.
- **Actividad 7.** Dar seguimiento a incidencias detectadas en operación, coordinando su análisis, diagnóstico y resolución en conjunto con el equipo técnico.

Sobre esa base, lo concreto para la semana es retomar la fase de guardar la región de detección, empezando por revisar el diseño visual bajo la convención ya confirmada.

---

## 4. Autorreflexión personal: problemas enfrentados, aprendizajes y desempeño

### 4.1 El patrón que más se repitió esta semana

Varias veces esta semana el trabajo no salió como estaba planeado, y en todos los casos la respuesta que funcionó fue la misma, no forzar el plan original y usar el tiempo en algo real que sí estaba disponible. Con el aforo bloqueado, se cerraron pendientes atrasados. Con conexión caída el lunes, se hizo un diagnóstico que no la necesitaba. Con la ventana de detección, cuando el bloqueo era de definición y no técnico, se optó por detenerse en vez de decidir unilateralmente.

### 4.2 Errores propios y lo que aprendí de cada uno

No hubo un error técnico que reportar esta semana, pero sí un error de cálculo al planear, se había comprometido avanzar en el aforo sin confirmar antes que la parte de la que depende ya estuviera lista. El aprendizaje es que antes de comprometerse con una tarea específica para la semana conviene confirmar que no dependa de algo que otra persona todavía no entrega, en vez de asumir que ya va a estar listo.

### 4.3 Aprendizajes profesionales y consideraciones éticas

- Cuando se encontró la contradicción en la definición de dirección, se escaló en vez de decidirla unilateralmente. Era una decisión que afectaba a otro módulo del sistema y no correspondía resolverla sin la autorización de quien lleva esa parte, aunque hubiera una opinión formada sobre cuál lectura parecía más razonable.
- El hallazgo de que alguien había optimizado el equipo de laboratorio sin avisar deja claro el costo de un cambio que no se comunica, otra persona termina gastando tiempo tratando de explicar una diferencia que ya tenía explicación.
- Se reportó que el aforo no avanzó, en vez de suavizarlo o de rellenar el reporte con detalle de otras tareas para que no se note. Un reporte que no dice claramente qué no se hizo deja de ser útil para quien lo lee.
- Este reporte se redactó cuidando el acuerdo de confidencialidad, describiendo qué se hizo y qué se decidió sin exponer infraestructura ni datos sensibles del cliente.

### 4.4 Sobre el cronograma

La diferencia entre lo planeado y lo hecho esta semana no salió de preferencia personal, salió de una dependencia bloqueada y de una interrupción externa. Lo que sí se puede controlar es reportar con precisión qué se trabajó y por qué, en vez de forzar que el reporte coincida con lo que se había dicho que se iba a hacer.

### 4.5 Desempeño, bienestar y acciones que voy a mantener

La semana tuvo un cambio de plan externo y una interrupción no planeada, y lo que evitó que eso se sintiera como una semana perdida fue tener siempre algo útil a la mano para redirigir el tiempo en vez de quedar a la espera de que se resolviera el bloqueo.

Acciones que se van a sostener:

- Confirmar que una tarea no dependa de algo externo antes de comprometerse con ella para la semana.
- Detenerse y escalar cuando el bloqueo es de definición y no técnico, en vez de decidir unilateralmente para no perder tiempo.
- Tener siempre pendientes reales a la mano para no dejar un imprevisto como tiempo perdido.
