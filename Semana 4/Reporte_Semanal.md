# Reporte Semanal

| Campo | Dato |
|---|---|
| **Nombre** | Zuleyca Guadalupe Balles Soto |
| **Matrícula** | A01741687 |
| **Programa Académico** | ITC |
| **Empresa** | Dirección de Vialidad y Semaforización del Ayuntamiento de Hermosillo |
| **Tutor Académico** | Mario Durán Vega |
| **Fecha** | 13 de septiembre de 2026 |
| **Periodo reportado** | Semana 4 - 7 al 11 de septiembre de 2026 |

> **Nota sobre confidencialidad.** El proyecto está sujeto a un acuerdo de confidencialidad. Este reporte describe metodología, decisiones técnicas y aprendizajes obtenidos, omitiendo identificadores de infraestructura, rutas de despliegue, credenciales, hallazgos de configuración escalados por canal interno y cualquier dato que permita ubicar equipos o instalaciones específicas.

---

## 1. Descripción de la fase u objetivos desarrollados en la semana declarados en el cronograma

### 1.1 Contexto del proyecto

Los términos **nodo**, **ventana de detección** y **contrato de comunicación** ya quedaron definidos en reportes anteriores. Esta semana se cierra el contrato de comunicación entre detección y luces (fases 6 a 9, continuación de las fases 1 a 5 cerradas la semana pasada) y se realiza además una sincronización de firmware en un controlador de campo real.

### 1.2 Actividades declaradas en el cronograma para la Semana 4

Según el cronograma, la Semana 4 corresponde a las mismas Actividades 3, 4 y 7 de la semana anterior:

- **Actividad 3.** Coordinar el desarrollo y la mejora continua de los módulos de software, asegurando la correcta evolución de componentes de ejecución de firmware.
- **Actividad 4.** Gestionar tareas y prioridades del equipo de desarrollo de software, organizando actividades, asignando responsabilidades y dando seguimiento al cumplimiento de objetivos dentro de los tiempos establecidos.
- **Actividad 7.** Dar seguimiento a incidencias detectadas en operación, coordinando su análisis, diagnóstico y resolución en conjunto con el equipo técnico.

### 1.3 Objetivos efectivamente perseguidos

**Actividad 3 - cubierta.** Se cerraron las cuatro fases restantes del contrato entre el módulo de detección y el módulo de luces, la regla que congela la decisión sobre el siguiente escenario en el momento correcto (con un defecto encontrado y corregido en el mecanismo de rescate), la política configurable para cuando la cámara falla o reporta un detector desconocido, la validación de que cada detector esté correctamente ligado a su escenario, y la observabilidad que expone hacia afuera el motivo real de cada decisión. Detalle completo en **Anexo A**.

**Actividad 7 - cubierta.** Se dio seguimiento a dos incidencias detectadas durante una actualización de firmware en un controlador de campo real, un fallo de arranque por un archivo faltante, y un rechazo del chequeo de salud por una discrepancia de configuración entre el paquete y el equipo de destino. Ambas se diagnosticaron y resolvieron el mismo día, sin dejar el equipo caído. Detalle completo en **Anexo B**.

**Actividad 4 - cubierta.** Se coordinaron tres puntos de decisión que afectaban directamente las prioridades y el margen de tiempo del equipo esta semana. Se confirmó con el responsable del proyecto cuántos ciclos esperar antes de reintentar cuando la cámara falla, y qué significa exactamente "usar otra fuente" en ese mismo escenario, antes de programar la fase de políticas; se confirmó también que un mismo detector usado en dos escenarios distintos entre ciclos diferentes no es ambiguo, antes de cerrar la validación de detectores; y se obtuvo autorización explícita para activar la sincronización del equipo de campo antes de la ventana de mantenimiento habitual, dado el cierre de calle programado ese mismo día.

### 1.4 Relación entre el cronograma y el trabajo de la semana

El cronograma y el trabajo realizado volvieron a coincidir en la Actividad 7, el seguimiento a incidencias estaba planeado para esta semana y la sincronización de campo produjo justo ese tipo de hallazgo. Cerrar el contrato entre detección y luces tampoco estaba explícitamente en el cronograma de esta semana puntual, pero es continuación directa de la Actividad 3 ya en curso desde la semana anterior.

**Actividad anticipada respecto al cronograma.** La sincronización de firmware en el controlador de campo corresponde a la Actividad 11 (coordinar el despliegue de nuevas versiones del sistema), programada para las semanas 10 a 13. Se adelantó porque el equipo de campo estaba diez versiones atrás del de laboratorio.

---

## 2. Descripción del desarrollo de las actividades semanales (metodología, recursos, procedimientos)

### 2.1 Metodología

Cada tarea de la semana cerró con un criterio definido antes de empezar y con un reporte escrito al terminar. Antes de programar cualquier corrección se revisó el código real en vez de asumir que el comportamiento documentado ya se cumplía, así se encontró el defecto del mecanismo de rescate (Anexo A). Los puntos que llegaron ambiguos en la tarea se aclararon antes de implementar, no después.

### 2.2 Recursos y herramientas

- **Lenguajes y entornos:** Python para el backend, JavaScript con una biblioteca de interfaz basada en componentes para la parte web.
- **Infraestructura:** servicios en ejecución continua para los módulos de detección y luces, un controlador de campo con su propio mecanismo de actualización de versiones.
- **Proceso:** suite de pruebas automatizadas corrida completa antes de dar por cerrado cualquier cambio, uso de código ya probado por el equipo en vez de parches manuales bajo presión de tiempo.

### 2.3 Procedimientos seguidos y actividades realizadas

**Cierre del contrato entre detección y luces (fases 6-9).** Cada fase cerró con su propio criterio y su propia verificación contra el código real. La más delicada fue la corrección del mecanismo de rescate en la regla de congelamiento, una caída de la cámara de apenas segundos podía otorgar paso sin evidencia real de que hubiera demanda, y se corrigió separando ese caso del mecanismo normal de falla-segura. Detalle completo en **Anexo A**.

**Sincronización de firmware en campo.** Se actualizó un controlador que llevaba diez versiones de atraso, reutilizando una corrección ya resuelta y probada por el equipo en el repositorio en vez de parchar el mecanismo de actualización a mano. Durante el despliegue surgieron dos incidencias, ambas diagnosticadas y corregidas el mismo día, dejando el equipo activo y sano antes de una ventana operativa que no se podía mover. Detalle completo en **Anexo B**.

### 2.4 Evidencia

El respaldo de lo descrito son los reportes de cierre redactados durante la semana, uno por el cierre del contrato entre detección y luces (fases 6 a 9), y uno por la sincronización de firmware en campo. Son documentos internos del proyecto, así que no se anexan completos por el acuerdo de confidencialidad.

Se anexan en versión con la información sensible omitida:

- **Anexo A.** Cierre del contrato de comunicación entre el módulo de detección de vehículos y el módulo de luces (fases 6 a 9).
- **Anexo B.** Sincronización de firmware e incidencias resueltas en un controlador de campo.

---

## 3. Listado de las actividades siguientes o pendientes no resueltas, con justificación

### 3.1 Estado de las actividades declaradas para la semana

Las tres actividades declaradas para la semana (3, 4 y 7) se cubrieron por completo; no queda ninguna abierta de las declaradas para este periodo.

### 3.2 Pendientes generados en la semana

**1. Matriz de conflictos entre grupos de semáforo, confirmada como pendiente de alta prioridad.** *Justificación:* requiere reglas reales de la geometría de cada intersección, que no corresponden a esta fase ni están definidas todavía. *Plan de acción:* coordinar con quien tenga ese conocimiento específico. *Resultado esperado:* activar el punto de verificación que ya quedó preparado en el código.

### 3.3 Actividades cercanas

Según el cronograma, la Semana 5 repite las mismas Actividades 3, 4 y 7, y a partir de la Semana 6 se incorporan las Actividades 5 y 6.

Sobre esa base, lo concreto para la semana es continuar dando seguimiento a los pendientes listados arriba y, si se confirma en el punto 4, dejar documentado si el flujo estándar de actualización cubre al equipo de campo.

---

## 4. Autorreflexión personal: problemas enfrentados, aprendizajes y desempeño

### 4.1 El patrón que más se repitió esta semana

Se repitió el mismo cuidado de semanas anteriores, verificar contra el comportamiento real antes de dar algo por corregido. En la regla de congelamiento del escenario, la suposición inicial era que ya cumplía la garantía documentada; solo se destapó el defecto al comparar el código contra la regla escrita, no al leerla por encima.

### 4.2 Errores propios y lo que aprendí de cada uno

Al escribir las pruebas nuevas de observabilidad (fase 9) se cometieron dos errores, fijar un valor artificial del reloj interno en vez de dejar que la función use el tiempo real, y un contador de repetición de ciclo incorrecto en una de las pruebas de congelamiento. Ambos se detectaron al correr la suite completa antes de cerrar la fase, y se corrigieron ahí mismo. El aprendizaje es que una prueba nueva no está exenta de los mismos descuidos que busca atrapar en el código que prueba, así que conviene correrla siempre contra la suite completa y no solo de forma aislada.

### 4.3 Aprendizajes profesionales y consideraciones éticas

- Ante una falla en producción bajo presión de tiempo, se prefirió reutilizar una corrección ya probada por el equipo en vez de improvisar un parche manual, aun cuando eso significara depender de un cambio hecho por otra persona.
- Los puntos ambiguos de la tarea (cuántos ciclos esperar antes de reintentar, qué significa exactamente "usar otra fuente" cuando la cámara falla) se consultaron con el responsable del proyecto antes de programar, en vez de resolverlos por cuenta propia.
- Se documentó explícitamente una limitación real que no se pudo cerrar del todo (la validación de que un detector exista de verdad, no solo que tenga el formato correcto), en vez de reportarla como resuelta.

### 4.4 Sobre el cronograma

El trabajo de mayor volumen esta semana correspondió a la Actividad 3, en línea con el cronograma, mientras que la sincronización de campo se adelantó respecto a lo programado formalmente, continuando el patrón de semanas anteriores de que el trabajo técnico avanza un poco por delante del cronograma declarado.

### 4.5 Desempeño, bienestar y acciones que voy a mantener

La semana combinó el cierre de cuatro fases técnicas seguidas con un despliegue en un equipo de campo real, bajo una ventana operativa que no se podía mover por el cierre de calle programado ese día. Tener código ya resuelto y probado por el equipo disponible en el repositorio fue lo que permitió cumplir esa ventana sin tener que improvisar una corrección bajo presión de tiempo.

Acciones que se van a sostener:

- Verificar el código real contra la regla documentada antes de dar por hecho que ya se cumple.
- Consultar con el responsable del proyecto los puntos ambiguos antes de implementar, aunque la solución técnica parezca clara.
- Preferir reutilizar código ya probado por el equipo en vez de parchar a mano bajo presión de tiempo.
