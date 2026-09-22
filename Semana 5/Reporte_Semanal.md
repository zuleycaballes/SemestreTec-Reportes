# Reporte Semanal

| Campo | Dato |
|---|---|
| **Nombre** | Zuleyca Guadalupe Balles Soto |
| **Matrícula** | A01741687 |
| **Programa Académico** | ITC |
| **Empresa** | Dirección de Vialidad y Semaforización del Ayuntamiento de Hermosillo |
| **Tutor Académico** | Mario Durán Vega |
| **Fecha** | 20 de septiembre de 2026 |
| **Periodo reportado** | Semana 5 - 14 al 18 de septiembre de 2026 |

> **Nota sobre confidencialidad.** El proyecto está sujeto a un acuerdo de confidencialidad. Este reporte describe metodología, decisiones técnicas y aprendizajes obtenidos, omitiendo identificadores de infraestructura, rutas de despliegue, credenciales, hallazgos de configuración escalados por canal interno y cualquier dato que permita ubicar equipos o instalaciones específicas.

---

## 1. Descripción de la fase u objetivos desarrollados en la semana declarados en el cronograma

### 1.1 Contexto del proyecto

Los términos **nodo**, **ventana de detección** y **contrato de comunicación** ya quedaron definidos en reportes anteriores. Se agrega uno nuevo para esta semana, la **identidad de controlador**, la forma en que un equipo de campo reconoce que una configuración nueva le pertenece. El sistema central identifica internamente a cada controlador con una clave de registro propia; el equipo físico, en cambio, exige su propia identidad nativa para aceptar cualquier configuración. Cuando ambas no coinciden, el equipo rechaza el envío aunque el resto del contenido sea válido.

### 1.2 Actividades declaradas en el cronograma para la Semana 5

Según el cronograma, la Semana 5 corresponde a las mismas Actividades 3, 4 y 7 de las dos semanas anteriores:

- **Actividad 3.** Coordinar el desarrollo y la mejora continua de los módulos de software, asegurando la correcta evolución de componentes de ejecución de firmware.
- **Actividad 4.** Gestionar tareas y prioridades del equipo de desarrollo de software, organizando actividades, asignando responsabilidades y dando seguimiento al cumplimiento de objetivos dentro de los tiempos establecidos.
- **Actividad 7.** Dar seguimiento a incidencias detectadas en operación, coordinando su análisis, diagnóstico y resolución en conjunto con el equipo técnico.

### 1.3 Objetivos efectivamente perseguidos

**Actividad 3 - cubierta.** Se cerró la última fase de pruebas del contrato entre el módulo de detección de vehículos y el módulo de luces (fase 10, continuación directa de las fases 1 a 9 cerradas en semanas anteriores), auditoría completa de los criterios de aceptación acordados contra la suite de pruebas existente, y cierre de los huecos de cobertura reales que se encontraron. Al fusionar esa fase con el resto del repositorio surgió un conflicto de integración entre dos líneas de desarrollo paralelas que tocaban el mismo contrato; se resolvió verificando el resultado contra la suite de pruebas completa. Se cerró además una fase adicional del rediseño de la ventana de detección, el despliegue del nuevo panel en producción detrás de un interruptor de seguridad apagado por defecto. Detalle completo en **Anexo A**.

**Actividad 7 - cubierta.** Se validó el contrato de configuración contra un controlador físico real de campo, como parte de las pruebas previas a habilitar despliegues hacia hardware real. Se identificó y documentó un desfase entre la identidad con la que el sistema central registra internamente a ese controlador y la identidad que el equipo exige para aceptar una configuración nueva, hallazgo que bloqueaba cualquier despliegue hacia él. Se dio seguimiento además a un incidente operativo, un bloqueo del sistema de migraciones de base de datos que impidió completar un despliegue en curso y que se resolvió, en coordinación con el equipo, reiniciando de fábrica el servidor afectado. Detalle del primer punto en **Anexo B**.

**Actividad 4 - cubierta parcialmente.** Se escaló formalmente al responsable del proyecto el hallazgo de identidad de controlador, con dos opciones de solución documentadas; el responsable del proyecto decidió y preparó, la misma semana, una corrección de raíz, cuya validación en laboratorio y aplicación quedan como coordinación pendiente. Quedan también pendientes de confirmación formal tres decisiones puntuales tomadas al resolver el conflicto de integración del contrato entre detección y luces.

### 1.4 Relación entre el cronograma y el trabajo de la semana

Esta semana el cronograma y el trabajo realizado volvieron a coincidir en la Actividad 7: el seguimiento a incidencias estaba planeado para esta semana, y tanto el hallazgo de identidad de controlador como el incidente de migraciones cayeron exactamente en esa categoría. Cerrar la última fase de pruebas del contrato entre detección y luces tampoco estaba explícitamente en el cronograma de esta semana, pero es continuación directa de la Actividad 3 ya en curso desde semanas anteriores.

**Actividad anticipada respecto al cronograma.** La fase de despliegue de la ventana de detección corresponde a la Actividad 6 (semanas 6 a 9), que se sigue trabajando por delante de lo programado, continuando el patrón ya señalado en reportes anteriores.

---

## 2. Descripción del desarrollo de las actividades semanales (metodología, recursos, procedimientos)

### 2.1 Metodología

Cada tarea de la semana cerró con un criterio definido antes de empezar y con un reporte escrito al terminar. Antes de escribir pruebas nuevas se auditó cada criterio de la tarea contra el código y las pruebas ya existentes, para no duplicar trabajo. Al resolver el conflicto de integración no se dio por bueno solo lo que la herramienta de control de versiones marcó como conflicto, se revisaron también los archivos tocados sin marcador, porque un merge automático puede introducir corrupción de contenido sin dejar ningún rastro visible. Los hallazgos de comportamiento (la ventana de detección, la identidad del controlador) se verificaron siempre contra la petición de red y la respuesta del equipo físico, y no contra lo que el código parecía indicar.

### 2.2 Recursos y herramientas

- **Lenguajes y entornos:** Python para el backend, JavaScript con una biblioteca de interfaz basada en componentes para la parte web.
- **Infraestructura:** base de datos relacional, servicios en ejecución continua para los módulos de detección y luces, un controlador de campo real para la validación de contrato, comunicación entre módulos por peticiones HTTP.
- **Proceso:** control de versiones con revisión manual más allá de lo marcado como conflicto, suite de pruebas automatizadas corrida completa antes de dar por cerrado cualquier cambio, escalación formal al responsable del proyecto antes de aplicar decisiones que afectan un sistema en operación real.

### 2.3 Procedimientos seguidos y actividades realizadas

**Cierre de la fase final de pruebas del contrato entre detección y luces, y resolución del conflicto de integración.** Se auditaron los criterios de aceptación pendientes contra la suite existente antes de escribir algo nuevo, y solamente se cerraron los huecos. El merge posterior con el resto del repositorio chocó con una integración paralela del mismo contrato hecha por otra línea de trabajo; la revisión encontró y corrigió corrupción de contenido que el merge había introducido sin marcador de conflicto. Detalle completo en **Anexo A**.

**Rediseño de la ventana de detección, fase de despliegue.** Se montó el nuevo panel en producción, detrás de una flag apagada por defecto, de modo que una falla del rediseño se pueda revertir sin redesplegar. En el camino se encontraron y corrigieron seis defectos, todos con la misma forma, una operación que aparentaba haber funcionado sin haberlo hecho, y en todos los casos lo que lo destapó fue revisar el dato real, no leer el código.

**Diagnóstico de identidad de controlador.** Se validó el contrato de configuración contra un controlador físico, reproduciendo el rechazo en los casos válidos del conjunto de pruebas, con y sin una bandera de reasignación, para aislar la causa exacta antes de proponer una solución. Detalle completo en **Anexo B**.

**Incidente de migraciones de base de datos.** Al intentar completar un despliegue, el sistema de migraciones bloqueó por números de migración reutilizados que ya estaban aplicados en producción, sin comando de reconciliación disponible. Se coordinó con el equipo el reinicio de fábrica del servidor afectado, lo que limpia la base de datos y deja como pendiente reaplicar el esquema completo.

### 2.4 Evidencia

El respaldo de lo descrito son los reportes de cierre y diagnóstico redactados durante la semana, uno por el cierre de la fase final de pruebas del contrato entre detección y luces, uno por la resolución del conflicto de integración, uno por el cierre de la fase de despliegue de la ventana de detección, y uno por el diagnóstico de identidad de controlador. Son documentos internos del proyecto, así que no se anexan completos por el acuerdo de confidencialidad.

Se anexan dos de ellos en versión con la información sensible omitida:

- **Anexo A.** Cierre de la fase final de pruebas del contrato entre detección y luces, y resolución del conflicto de integración.
- **Anexo B.** Diagnóstico de un desfase de identidad entre el registro central y un controlador de campo.

Los demás pueden entregarse en el mismo formato si el asesor los requiere.

---

## 3. Listado de las actividades siguientes o pendientes no resueltas, con justificación

### 3.1 Estado de las actividades declaradas para la semana

**Actividad 4, gestión de tareas y prioridades.** Cubierta parcialmente. *Justificación:* la validación en laboratorio de la corrección de identidad de controlador depende de una ventana autorizada y no de trabajo pendiente propio; la confirmación de los tres puntos del conflicto de integración depende de respuesta del responsable del proyecto. *Plan de acción:* dar seguimiento la próxima semana. *Resultado esperado:* corrección validada y aplicada, y los tres puntos confirmados formalmente.

### 3.2 Pendientes generados en la semana

**1. Validar en laboratorio y aplicar la corrección de identidad de controlador.** *Plan de acción:* respaldar la base de datos, aplicar la migración correspondiente con el ejecutor del proyecto, y confirmar en laboratorio que el controlador acepta la configuración antes de cualquier envío real. *Resultado esperado:* el controlador de campo diagnosticado acepta configuración nueva sin rechazo.

**2. Confirmar con el responsable del proyecto los tres puntos abiertos de la resolución del conflicto de integración.** *Plan de acción:* dar seguimiento la próxima semana. *Resultado esperado:* cierre formal de los tres puntos.

### 3.3 Actividades cercanas

Según el cronograma, la Semana 6 incorpora las Actividades 5 y 6, además de continuar con las Actividades 3, 4 y 7 ya en curso.

Sobre esa base, lo concreto para la semana es reanudar el despliegue en cuanto el esquema de base de datos esté completo, dar seguimiento a la validación de la corrección de identidad de controlador, y cerrar los tres puntos pendientes de confirmación del conflicto de integración.

---

## 4. Autorreflexión personal: problemas enfrentados, aprendizajes y desempeño

### 4.1 El patrón que más se repitió esta semana

Se repitió, con una variante nueva, el mismo cuidado de semanas anteriores de verificar el dato real antes de confiar en la apariencia de éxito. Esta semana ese cuidado se extendió a un caso distinto, dos identificadores que deberían significar lo mismo pero no lo hacen. Ocurrió con la fusión de código, donde la ausencia de marcador de conflicto no significó ausencia de corrupción, y con el controlador de campo, donde una clave de registro interna válida no significó una identidad que el equipo físico reconociera. En ambos casos, dar por bueno el identificador equivocado habría producido una falla silenciosa en vez de un rechazo temprano.

### 4.2 Errores propios y lo que aprendí de cada uno

Al escribir las pruebas nuevas de la fase final del contrato se cometieron dos errores. El primero, construir el primer intento de una prueba con coordenadas normalizadas cuando la función bajo prueba en realidad trabaja con coordenadas reales de imagen; se corrigió antes de formalizar la prueba. El segundo, dejar fijo el reloj interno en cero en el primer intento de una prueba de integración, lo que hizo que un paso de la cadena tomara por accidente una rama distinta a la que se quería probar; se corrigió fijando el tiempo de forma explícita en cada paso. El aprendizaje es el mismo en los dos casos, una prueba nueva no está exenta de los descuidos que busca atrapar en el código que prueba.

### 4.3 Aprendizajes profesionales y consideraciones éticas

- Los tres puntos de la resolución del conflicto de integración que cambiaban comportamiento ya decidido (quitar una validación que hacía imposible una fase ya aprobada, renombrar un archivo que se describe a sí mismo como contrato compartido con otro sistema, eliminar pruebas que afirmaban lo contrario de una decisión ya confirmada) se dejaron marcados como pendientes de confirmación explícita, en vez de darlos por resueltos porque la lógica pareciera clara.
- Al encontrar que un archivo se describe a sí mismo como compartido con otro sistema o equipo, se dejó anotado que no había forma de confirmar eso desde el propio repositorio, en vez de asumir que no importaba.
- El hallazgo de identidad de controlador se escaló completo, con las dos opciones de solución documentadas, en vez de tomar la decisión por cuenta propia, tratándose de un cambio que afecta equipos de campo ya desplegados.

### 4.4 Sobre el cronograma

El trabajo de mayor volumen esta semana volvió a correr por delante de lo formalmente programado; la fase de despliegue de la ventana de detección corresponde a una actividad programada para semanas más adelante en el cronograma, continuando el patrón de semanas anteriores.

### 4.5 Desempeño, bienestar y acciones que voy a mantener

La semana combinó cierres de fase con dos incidentes no planeados el mismo día, uno técnico (identidad de controlador) y uno operativo (bloqueo de migraciones), ninguno con una respuesta obvia inmediata. En ambos se prefirió escalar y coordinar con el equipo antes que resolver por cuenta propia bajo presión; el reinicio de fábrica del servidor deja un pendiente concreto para la semana siguiente, pero evita dejar el sistema en un estado a medio corregir.

Acciones que se van a sostener:

- Verificar cualquier identificador que cruce entre dos sistemas distintos, en vez de asumir que significa lo mismo de ambos lados.
- Revisar los archivos que una fusión de código no marcó como conflicto, no solo los que sí lo marcó.
- Escalar hallazgos que afectan equipos ya desplegados, con opciones documentadas, en vez de decidir por cuenta propia.