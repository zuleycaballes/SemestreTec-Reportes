# Reporte Semanal

| Campo | Dato |
| --- | --- |
| **Nombre** | Zuleyca Guadalupe Balles Soto |
| **Matrícula** | A01741687 |
| **Programa Académico** | ITC |
| **Empresa** | Dirección de Vialidad y Semaforización del Ayuntamiento de Hermosillo |
| **Tutor Académico** | Mario Durán Vega |
| **Fecha** | 12 de octubre de 2026 |
| **Periodo reportado** | Semana 8 - 5 al 9 de octubre de 2026 |

> **Nota sobre confidencialidad.** El proyecto está sujeto a un acuerdo de confidencialidad. Este reporte describe metodología, decisiones técnicas y aprendizajes obtenidos, omitiendo identificadores de infraestructura, rutas de despliegue, credenciales y cualquier dato que permita ubicar equipos o instalaciones específicas.

---

## 1. Descripción de la fase u objetivos desarrollados en la semana declarados en el cronograma

### 1.1 Contexto del proyecto

Los términos **nodo**, **ventana de detección**, **contrato de comunicación**, **Sincronía** e **identidad de controlador** ya quedaron definidos en reportes anteriores. Se agregan cuatro para esta semana. La semana giró en torno a la preparación del corte controlado del parche a producción (**PC-11**), incluida la forma de regresar el sistema si algo falla, y el reemplazo definitivo de la ventana de detección por la versión rediseñada.

### 1.2 Actividades declaradas en el cronograma para la Semana 8

Según el cronograma, la Semana 8 corresponde a las mismas Actividades 5, 6 y 8 de las dos semanas anteriores:

- **Actividad 5.** Supervisar la integración entre hardware y software, verificando la correcta comunicación con controladores, dispositivos en campo y sistemas embebidos.
- **Actividad 6.** Proponer e implementar mejoras en la arquitectura del sistema, orientadas a incrementar la estabilidad, escalabilidad y facilidad de mantenimiento.
- **Actividad 8.** Optimizar la lógica de control semafórico, ajustando ciclos, transiciones y sincronización entre intersecciones para mejorar el flujo vehicular.

### 1.3 Objetivos efectivamente perseguidos

**Actividad 5 - cubierta parcialmente.** El trabajo de integración fue de dos tipos. Primero, se preparó el plan de ejecución del corte controlado (PC-11), que define cómo se verifica la comunicación entre el sistema central y el controlador después del despliegue, estado de salud de los servicios, coincidencia de la identidad del controlador en cuatro fuentes distintas, validez del formato de configuración y coincidencia de huellas (hash) entre lo enviado y lo leído. Segundo, se probó en el controlador de laboratorio la nueva función de la ventana de detección que lee desde el equipo la configuración que está corriendo, sin escribirle nada. Esa prueba fue la que expuso que el equipo del laboratorio se había apagado por su fuente de poder. *Justificación de cobertura parcial:* el corte no se ejecutó y no se probó nada en el controlador de calle.

**Actividad 6 - cubierta.** La ventana de detección rediseñada ya estaba integrada en las ramas principales, pero convivía con el panel anterior y con una bandera para alternar entre ambos. Se retiró el panel anterior del flujo, de modo que la ventana nueva quedó como la única. Para poder hacerlo sin que el equipo perdiera nada de lo que usa en operación, se portaron a la ventana nueva tres funciones que solo existían en el panel anterior: ajuste de sensibilidad de detección, lectura de la configuración del controlador y descarte o restauración del borrador. Con esto dejó de existir la bandera.

**Actividad 8 - no cubierta.** Esta semana no se trabajó la lógica de control semafórico ni se ajustó ningún ciclo. *Justificación:* la semana tuvo dos días de trabajo en el proyecto, que se dedicaron a las actividades anteriormente mencionadas, y el ajuste real de ciclos y de sincronía entre intersecciones sigue condicionado a la instalación de los controladores del corredor.

### 1.4 Relación entre el cronograma y el trabajo de la semana

Esta semana el cronograma y el trabajo realizado coincidieron en las Actividades 5 y 6. La Actividad 5 quedó parcial por depender de autorizaciones, y la Actividad 8 no tuvo avance.

**Actividad no programada para esta semana.** El diagnóstico del tiempo de espera al leer la configuración del controlador de laboratorio, que terminó siendo una falla de energía del equipo, es seguimiento a incidencias (Actividad 7). Se atendió porque impedía terminar la prueba de la función nueva.

---

## 2. Descripción del desarrollo de las actividades semanales (metodología, recursos, procedimientos)

### 2.1 Metodología

El criterio de la semana fue no confiar en un plan por el solo hecho de estar escrito, y no tocar nada de calle. El checklist del corte se contrastó contra el estado real de los repositorios y de los scripts de despliegue antes de convertirlo en un procedimiento, y todo lo que no se pudo comprobar quedó marcado como pendiente de verificación en lugar de darse por bueno. Para la ventana de detección se portó solo lo que el equipo indicó que se necesita, y se probó en el controlador de laboratorio, dejando el de calle intacto. El corte se documentó con el estado **no ejecutada**, para que el plan no pueda confundirse con evidencia de una corrida real.

### 2.2 Recursos y herramientas

- **Infraestructura:** nodo y controlador de laboratorio con firmware propio, servidor de producción Linux y base de datos relacional con su runner de migraciones versionado.
- **Aplicación:** frontend en JavaScript con biblioteca de interfaz basada en componentes, backend en Python, WebSocket que comunica el sistema central con el controlador.
- **Verificación:** compilación del frontend, pruebas unitarias de validación, simulación de la pantalla con jsdom y pruebas manuales en el nodo de laboratorio con capturas.

### 2.3 Procedimientos seguidos y actividades realizadas

**Plan de ejecución del corte controlado (PC-11).** El punto de partida fue un checklist de 18 puntos, 7 condiciones de reversa y 4 criterios de terminado. Se contrastó contra los repositorios del sistema central y del firmware, y contra los scripts de despliegue y de actualización del controlador. Con eso se armó un procedimiento en cuatro fases: preparación previa a la ventana (pruebas sobre los mismos artefactos que se van a desplegar, sumas de verificación, respaldo completo con restauración probada en una base temporal, migraciones ensayadas en esa copia y arranque de la versión anterior contra el esquema nuevo); corte del sistema central con las escrituras de Sincronía detenidas; despliegue en laboratorio con verificación de identidad, formato de configuración y huellas; y observación con prueba de punta a punta. Incluye la reversa para el controlador, para la aplicación y para los datos, y un ensayo obligatorio en laboratorio antes de promover. En el contraste aparecieron cinco discrepancias entre el checklist y la realidad, que quedaron como requisito antes de abrir la ventana. Detalle completo en **Anexo A**.

**Reemplazo de la ventana de detección.** El primer paso fue identificar qué tenía el panel anterior que la ventana nueva no tenía. Eran tres funciones que hacen falta en operación (sensibilidad de detección, lectura de la configuración del controlador y descarte o restauración del borrador), y se decidió no portar dos más (activar o desactivar detectores y cámaras, y el editor de zonas de detección en formato JSON) porque no se necesitan. Se modificaron seis archivos del frontend, sin tocar el backend. La sensibilidad se valida antes de guardar (rangos de confianza y de ubicación, tiempos no negativos). La lectura del controlador no escribe nada en el equipo y, si hay un borrador pendiente, pide confirmación antes y solo lo descarta si la lectura salió bien. Además, el envío de la configuración al controlador queda deshabilitado mientras haya ajustes sin guardar, para no enviar algo distinto de lo que se ve en pantalla. Antes de entregar los archivos se verificó el comportamiento con jsdom; después se compiló el frontend completo y se corrieron 11 pruebas nuevas. Detalle completo en **Anexo B**.

**Pruebas en el controlador de laboratorio.** Se comprobó que revertir los ajustes y las perillas de sensibilidad funcionan, que la lectura de la configuración del controlador funciona, y se completaron las pruebas de envío y restauración de la configuración con un valor distinto del original. No se probó nada en el controlador de calle.

**Tiempo de espera al leer la configuración.** En la primera prueba, la lectura devolvió que el sistema de visión no respondió a tiempo. La interfaz mostró el error como debía. El registro del backend mostraba la conexión con el equipo vencida cada pocos segundos, lo que apuntaba a que el equipo no estaba respondiendo y no a un defecto de la función nueva. La causa fue que el equipo del laboratorio se había apagado por su fuente de poder. Al recuperar la energía, la lectura funcionó sin cambiar código.

### 2.4 Evidencia

El respaldo de lo descrito son los reportes técnicos redactados durante la semana, el plan de ejecución del corte controlado y el reporte del reemplazo de la ventana de detección. Son documentos internos del proyecto, así que no se anexan completos por el acuerdo de confidencialidad.

Se anexan ambos en versión con la información sensible omitida:

- **Anexo A.** Plan de ejecución del corte controlado y rollback (PC-11): fases, discrepancias encontradas y criterios de terminado.
- **Anexo B.** Reemplazo de la ventana de detección: cambios, comportamiento nuevo, verificación y hallazgo de la fuente de poder.

---

## 3. Listado de las actividades siguientes o pendientes no resueltas, con justificación

### 3.1 Estado de las actividades declaradas para la semana

**Actividad 5, supervisar la integración hardware y software.** Cubierta parcialmente. *Justificación:* el corte (PC-11) no se ejecutó y el flujo de la ventana no se probó en el controlador de calle. *Plan de acción:* cerrar las discrepancias del plan, correr la preparación previa en una copia restaurada y solicitar la ventana con los responsables y la autorización vial. *Resultado esperado:* PC-11 aprobada con evidencia, o rechazada con la causa documentada.

**Actividad 8, optimizar la lógica de control semafórico.** No cubierta. *Justificación:* la semana tuvo dos días de trabajo en el proyecto y se priorizó lo demás; igualmente, no hay ajustes reales de ciclo que aplicar mientras los controladores del corredor no estén instalados. *Plan de acción:* retomar el ajuste de ciclos y de sincronía, con el modelado de la semana 6 como referencia, en cuanto el hardware del corredor esté instalado. *Resultado esperado:* primer ajuste real de ciclos sobre un corredor coordinado.

La Actividad 6 se cubrió.

### 3.2 Pendientes generados en la semana

**1. Ejecución del corte controlado (PC-11).** *Justificación:* El rollback debe ensayarse en laboratorio antes de hacer el cambio, y cualquier envío que afecte luces requiere la autorización del responsable vial. *Plan de acción:* correr la preparación previa en copia restaurada, ajustar el plan y llenar la hoja de ejecución con SHA, responsables y ventana. *Resultado esperado:* ventana autorizada y corte ejecutado con la plantilla de cierre llena.

**Pendientes de la semana 7 sin avance.** Los pasos 8e y 14 de la PC-10 (aislamiento entre controladores del mismo nodo), que requieren hardware multicontrolador que aún no llega; la protección contra cuadros viejos en el croquis de detección; el blindaje del backend para que el borrador de Sincronía esté protegido también del lado del servidor; y el cierre del diagnóstico de cámaras, que incluye confirmar el origen de las tres fuentes huérfanas y decidir sobre tres mejoras propuestas, a criterio del responsable del proyecto. *Justificación:* no se trabajaron porque no ha sido prioridad. *Plan de acción:* se mantienen en la lista y se retoman en orden de impacto.

### 3.3 Actividades cercanas

Según el cronograma, la Semana 9 repite las mismas Actividades 5, 6 y 8 y cierra ese bloque; a partir de la Semana 10 el cronograma pasa a las Actividades 9, 10 y 11. Lo concreto para la Semana 9 es avanzar la preparación previa del corte, continuar la validación del contrato de comunicación entre el sistema central y el controlador contra el despliegue real (PC-09), y dar seguimiento a los pendientes de arriba.

---

## 4. Autorreflexión personal: problemas enfrentados, aprendizajes y desempeño

### 4.1 El patrón que más se repitió esta semana

El checklist del corte nombraba una migración que ya estaba aplicada, daba por aditivas migraciones que ya no lo eran, atribuía al chequeo de salud del controlador verificaciones que no hace y se apoyaba en un flujo de identidad que ya cambió. Con el tiempo de espera de la lectura pasó algo parecido, el primer síntoma parecía una falla de la función recién escrita, y la causa estaba en el equipo. En ambos casos lo mejor fue ir a la fuente (el repositorio, el registro del backend) antes de actuar sobre la primera explicación.

### 4.2 Errores propios y lo que aprendí de cada uno

Esta semana no hubo un error propio que afectara el trabajo, pero refuerzó la idea de dejar evidencia por prueba (resultado esperado contra observado) en el momento, y verificar quién usa un componente antes de retirarlo, no después.

### 4.3 Aprendizajes profesionales y consideraciones éticas

- No se probó ni se tocó el controlador de calle. Todo se hizo en el controlador de laboratorio, y el flujo de calle quedó supeditado a la aprobación del responsable del backend o de la jefatura.
- El plan de corte deja explícito que ningún envío que afecte luces se hace por el solo hecho de tener acceso técnico, requiere la autorización del responsable vial.
- El corte quedó documentado como no ejecutada. Un plan bien escrito no es evidencia de una corrida, y no debe presentarse como tal.
- Se portó solo lo que el equipo dijo necesitar, en lugar de copiar todo el panel anterior por completitud.

### 4.4 Sobre el cronograma

Esta semana el cronograma coincidió con el trabajo real en las Actividades 5 y 6; la 5 quedó parcial por depender de autorizaciones y la 8 no tuvo avance. El seguimiento a incidencias (Actividad 7) volvió a ocupar parte de la semana aunque el cronograma lo programó hasta la semana 5.

La semana tuvo dos días de trabajo en el proyecto, el lunes 5 y el martes 6. El miércoles 7 viajé a la conferencia Out for Undergrad Business, en Nueva York, y regresé hasta el domingo 11. Lo que se hizo en esos dos días quedó documentado, con su estado y sus pendientes escritos.

### 4.5 Desempeño, bienestar y acciones que voy a mantener

Con dos días disponibles lo que ayudó fue decidir pronto qué cabía y qué no, el plan de corte y la ventana de detección sí, y los pendientes de la semana anterior no. La conferencia fue una experiencia de desarrollo profesional útil, con una preparación previa de las industrias y de las preguntas que quería hacer, un espacio con la comunidad latina e hispana del evento y una plática sobre trayectoria profesional y ambientes de trabajo respetuosos.

Acciones que se van a sostener:

- Contrastar cualquier checklist o plan contra el estado real del repositorio y del entorno antes de ejecutarlo.
- Ante un síntoma, revisar primero la capa de infraestructura (energía, conectividad, registros) antes de modificar código.
- Dejar por escrito lo pendiente y de quién depende antes de ausentarme.
