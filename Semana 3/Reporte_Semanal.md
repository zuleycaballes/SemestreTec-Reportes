# Reporte Semanal

| Campo | Dato |
|---|---|
| **Nombre** | Zuleyca Guadalupe Balles Soto |
| **Matrícula** | A01741687 |
| **Programa Académico** | ITC |
| **Empresa** | Dirección de Vialidad y Semaforización del Ayuntamiento de Hermosillo |
| **Tutor Académico** | Mario Durán Vega |
| **Fecha** | 8 de septiembre de 2026 |
| **Periodo reportado** | Semana 3 - 31 de agosto al 4 de septiembre de 2026 |

> **Nota sobre confidencialidad.** El proyecto está sujeto a un acuerdo de confidencialidad. Este reporte describe metodología, decisiones técnicas y aprendizajes obtenidos, omitiendo identificadores de infraestructura, rutas de despliegue, credenciales, hallazgos de configuración escalados por canal interno y cualquier dato que permita ubicar equipos o instalaciones específicas.

---

## 1. Descripción de la fase u objetivos desarrollados en la semana declarados en el cronograma

### 1.1 Contexto del proyecto

Los términos **nodo** y **ventana de detección** ya quedaron definidos en reportes anteriores. Se agrega uno nuevo para esta semana, el **contrato de comunicación** entre el módulo que detecta vehículos por cámara y el módulo que decide las fases del semáforo, es decir, las reglas que fijan qué datos viajan de uno a otro, con qué frecuencia y bajo qué garantías, para que una decisión de semáforo nunca se tome sobre información corrupta o incompleta.

### 1.2 Actividades declaradas en el cronograma para la Semana 3

Según el cronograma, la Semana 3 corresponde a las Actividades 3, 4 y 7:

- **Actividad 3.** Coordinar el desarrollo y la mejora continua de los módulos de software, asegurando la correcta evolución de componentes de ejecución de firmware.
- **Actividad 4.** Gestionar tareas y prioridades del equipo de desarrollo de software, organizando actividades, asignando responsabilidades y dando seguimiento al cumplimiento de objetivos dentro de los tiempos establecidos.
- **Actividad 7.** Dar seguimiento a incidencias detectadas en operación, coordinando su análisis, diagnóstico y resolución en conjunto con el equipo técnico.

### 1.3 Objetivos efectivamente perseguidos

**Actividad 3 - cubierta.** Se cerraron las cinco fases planeadas del contrato entre el módulo de detección y el módulo de luces, definición formal de sus reglas, filtro de tiempo mínimo sostenido antes de reaccionar a una detección, límite estricto de frecuencia de publicación con señal de "sigo vivo", validación completa del lado receptor antes de aplicar cualquier cambio, y formalización de la memoria de corto plazo que recuerda una demanda pendiente. Detalle completo en **Anexo A**.

**Actividad 7 - cubierta.** Se diagnosticó una incidencia reportada en un nodo real, la ventana rediseñada de detección mostraba "sin cámaras" donde el panel actual sí ve video en vivo. Se encontró la causa raíz, se confirmó que no crece hacia adelante, y se documentaron tres opciones de resolución con una recomendación para que el responsable del proyecto decida. Detalle completo en **Anexo B**.

**Actividad 4 - cubierta parcialmente.** Dos puntos de coordinación con el resto del equipo, se planteó a quien lleva esa parte del backend si una función que se iba a construir para reportar regiones descartadas seguía siendo necesaria, dado que la validación ya existente parece cubrir el mismo caso; y se llevó al responsable del proyecto la decisión de cuándo autorizar la reconciliación de inventario histórico antes de habilitar la ventana rediseñada en producción.

**Actividades anticipadas respecto al cronograma.**

- Ventana de detección (Actividad 6, semanas 6 a 9): se retomó, con la convención de dirección ya confirmada por el responsable del proyecto al cierre de la Semana 2, la fase que guarda la región de detección en el borrador de configuración del controlador. Se encontraron y corrigieron cinco defectos reales en el camino.
- Documentación de cambios y arquitectura (Actividad 13, semana 15): cada cierre de esta semana quedó por escrito, igual que en semanas anteriores.

### 1.4 Relación entre el cronograma y el trabajo de la semana

Esta semana el cronograma y el trabajo realizado coincidieron más que en semanas anteriores, la Actividad 7 (seguimiento a incidencias) estaba planeada para esta semana y el hallazgo del inventario histórico cayó exactamente en esa categoría. Por otro lado, cerrar el contrato entre detección y luces no estaba explícitamente en el cronograma de esta semana, pero es exactamente el tipo de mejora continua de módulos de software que describe la Actividad 3.

La decisión que el responsable del proyecto confirmó al cierre de la Semana 2 sobre la convención de dirección resultó correcta en la práctica, no hubo que corregir ningún dato ya guardado, porque en ese momento todavía no existía ninguno con la convención equivocada, tal como se había anticipado.

---

## 2. Descripción del desarrollo de las actividades semanales (metodología, recursos, procedimientos)

### 2.1 Metodología

Cada tarea de la semana cerró con un criterio definido antes de empezar y con un reporte escrito al terminar. Los cambios que afectan comportamiento de un sistema en operación real se confirman con el responsable del proyecto antes de implementarse, no después. Cuando surgió la duda de si una función planeada seguía siendo necesaria, se preguntó a quien lleva esa parte del backend antes de construirla, en vez de asumir.

### 2.2 Recursos y herramientas

- **Lenguajes y entornos:** Python para el backend, JavaScript con una biblioteca de interfaz basada en componentes para la parte web.
- **Infraestructura:** base de datos relacional, servicios en ejecución continua para los módulos de detección y luces, comunicación entre módulos por peticiones HTTP.
- **Proceso:** control de versiones con convención de mensajes de commit verificada automáticamente, validación de cada archivo entregado antes de integrarlo, suite de pruebas corrida completa antes de dar por cerrado cualquier cambio.

### 2.3 Procedimientos seguidos y actividades realizadas

**Contrato entre el módulo de detección y el módulo de luces.** Se cerraron cinco fases, cada una con su propio criterio de cierre y su propia verificación contra el código real. La más delicada fue la validación del lado receptor, antes cualquier tipo de dato pasaba directo a mutar el estado real, ahora un paquete que no cumple se rechaza completo, sin aplicar nada a medias. Detalle completo en **Anexo A**.

**Persistencia de la región de detección.** Con la convención de dirección ya confirmada, se implementó que la ventana rediseñada guarde la región de detección en el borrador de configuración del controlador, dejando el envío al hardware como una acción separada y explícita. Se encontraron y corrigieron cinco defectos, entre ellos: una ruta de lectura que nunca existió y cuyo error quedaba absorbido en silencio; un dato que se guardaba envuelto en un paquete completo en vez de guardarse directo; una ventana de confirmación que se cerraba antes de que el guardado real terminara; un componente que pedía el identificador equivocado para pedir una imagen de vista previa; y un guardado que, tras reconstruir el ambiente de pruebas, quedó apuntando a la función de modo simulado en vez de la función real, de modo que la pantalla mostraba éxito sin que el dato llegara a la base de datos. Los cinco comparten la misma forma, una operación que aparentaba haber funcionado sin haberlo hecho, y en los cinco casos lo que lo destapó fue revisar el dato, no leer el código.

**Diagnóstico del inventario histórico de cámaras.** Se investigó por qué un nodo real mostraba cámaras en el panel actual pero no en la ventana rediseñada. La causa fue que esas cámaras se registraron antes de que existiera la proyección automática hacia la tabla que la ventana nueva lee, y nunca se proyectaron manualmente. Se confirmó que el hueco no crece hacia adelante y se documentaron tres opciones para el responsable del proyecto. Detalle completo en **Anexo B**.

### 2.4 Evidencia

El respaldo de lo descrito son los reportes de cierre redactados durante la semana, uno por cada fase del contrato entre detección y luces, uno por el cierre de la persistencia de la región de detección, y uno por el diagnóstico de inventario. Son documentos internos del proyecto, así que no se anexan completos por el acuerdo de confidencialidad.

Se anexan dos de ellos en versión con la información sensible omitida:

- **Anexo A.** Cierre del contrato de comunicación entre el módulo de detección de vehículos y el módulo de luces.
- **Anexo B.** Diagnóstico de inventario histórico de cámaras no proyectado.

Los demás pueden entregarse en el mismo formato si el asesor los requiere.

---

## 3. Listado de las actividades siguientes o pendientes no resueltas, con justificación

### 3.1 Estado de las actividades declaradas para la semana

**Actividad 4, gestión de tareas y prioridades.** Cubierta parcialmente. *Justificación:* los dos puntos de coordinación planteados esta semana (necesidad real de una función ya cubierta por otra parte del sistema, y autorización para reconciliar inventario histórico) dependen de respuesta de otras personas del equipo y del responsable del proyecto, no de trabajo pendiente propio. *Plan de acción:* dar seguimiento la próxima semana. *Resultado esperado:* confirmación en ambos puntos y, si aplica, ejecutar la reconciliación en el ambiente de pruebas.

### 3.2 Pendientes generados en la semana

**1. Confirmar si la función de reporte de regiones descartadas sigue siendo necesaria.** *Plan de acción:* esperar respuesta de quien lleva esa parte del backend. *Resultado esperado:* cerrar el punto sin construir algo redundante, o construirlo si de verdad falta.

**2. Autorización para reconciliar el inventario histórico de cámaras.** *Plan de acción:* verificar la reconciliación en el ambiente de pruebas en cuanto se autorice, antes de correrla en producción. *Resultado esperado:* la ventana rediseñada mostrando cámaras de forma consistente en todos los nodos.

**3. Verificación de campo pendiente, heredada de la Semana 2.** Confirmar en un nodo real a qué brazo de la intersección da paso una dirección de detección específica, para terminar de validar la convención confirmada por el responsable del proyecto. *Plan de acción:* programarla con el equipo de campo. *Resultado esperado:* confirmación documental de que la convención implementada corresponde a la realidad física. No bloquea nada mientras tanto.

### 3.3 Actividades cercanas

Según el cronograma, la Semana 4 repite las mismas Actividades 3, 4 y 7 de esta semana, sin cambio hasta la Semana 6:

- **Actividad 3.** Coordinar el desarrollo y la mejora continua de los módulos de software, asegurando la correcta evolución de componentes de ejecución de firmware.
- **Actividad 4.** Gestionar tareas y prioridades del equipo de desarrollo de software, organizando actividades, asignando responsabilidades y dando seguimiento al cumplimiento de objetivos dentro de los tiempos establecidos.
- **Actividad 7.** Dar seguimiento a incidencias detectadas en operación, coordinando su análisis, diagnóstico y resolución en conjunto con el equipo técnico.

Sobre esa base, lo concreto para la semana es dar seguimiento a los dos pendientes de coordinación abiertos esta semana y, si se autoriza, ejecutar la reconciliación de inventario en el ambiente de pruebas.

---

## 4. Autorreflexión personal: problemas enfrentados, aprendizajes y desempeño

### 4.1 El patrón que más se repitió esta semana

Se repitió un patrón de semanas anteriores, cuando algo parece estar roto, revisar primero de dónde lee cada parte del sistema antes de asumir que el código nuevo tiene la falla. Con el inventario de cámaras, la reacción inicial fue pensar que el rediseño estaba fallando; resultó ser un problema de datos históricos sin migrar, no de código. Ese mismo cuidado de revisar el dato real en vez de confiar en la apariencia fue lo que destapó los cinco defectos de la persistencia de la región de detección.

### 4.2 Errores propios y lo que aprendí de cada uno

Se había planeado construir una función para que el sistema reporte qué regiones de detección se descartan por no tener detector declarado, sin antes confirmar si algo similar ya existía del lado del backend. Al revisar, resultó que la validación ya existente reporta prácticamente lo mismo. El aprendizaje es confirmar con quien lleva esa parte del sistema antes de empezar a construir algo, no después de tenerlo medio hecho.

### 4.3 Aprendizajes profesionales y consideraciones éticas

- Dos cambios de comportamiento del contrato entre detección y luces (la corrección de cómo se combinan varias cámaras de un mismo detector, y el rechazo de paquetes incompletos) se confirmaron con el responsable del proyecto antes de implementarse, aunque técnicamente la solución pareciera clara.
- Al encontrar que un archivo del sistema guarda credenciales sin cifrar, se dejó anotado como nota de seguridad aparte, aunque no era parte de la tarea asignada.
- Se reportaron los dos pendientes de coordinación como abiertos y sin resolver, en vez de darlos por hechos o minimizarlos en el reporte.

### 4.4 Sobre el cronograma

Esta semana el cronograma coincidió mejor que otras con el trabajo real, en parte porque el seguimiento a incidencias operativas ya estaba planeado y el hallazgo del inventario cayó exactamente ahí. Aun así, el trabajo de mayor volumen, siguió corriendo por delante de lo formalmente programado, continuando el patrón de semanas anteriores.

### 4.5 Desempeño, bienestar y acciones que voy a mantener

La semana avanzó con más continuidad que la anterior, no hubo un bloqueo externo que detuviera trabajo ya en curso, y la decisión pendiente de la Semana 2 llegó a tiempo para retomar sin fricción. Lo que sí quedó pendiente fue de coordinación, no técnico, lo cual se siente distinto y conviene no confundir con atraso real.

Acciones que se van a sostener:

- Confirmar con quien lleva una parte del sistema si algo ya existe antes de empezar a construirlo.
- Escalar cambios de comportamiento antes de implementarlos, aunque la solución técnica parezca obvia.
- Revisar de dónde lee cada parte del sistema antes de asumir cuál está fallando cuando dos pantallas no coinciden.
