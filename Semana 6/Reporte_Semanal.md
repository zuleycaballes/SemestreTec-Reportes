# Reporte Semanal

| Campo | Dato |
|---|---|
| **Nombre** | Zuleyca Guadalupe Balles Soto |
| **Matrícula** | A01741687 |
| **Programa Académico** | ITC |
| **Empresa** | Dirección de Vialidad y Semaforización del Ayuntamiento de Hermosillo |
| **Tutor Académico** | Mario Durán Vega |
| **Fecha** | 27 de septiembre de 2026 |
| **Periodo reportado** | Semana 6 - 21 al 25 de septiembre de 2026 |

> **Nota sobre confidencialidad.** El proyecto está sujeto a un acuerdo de confidencialidad. Este reporte describe metodología, decisiones técnicas y aprendizajes obtenidos, omitiendo identificadores de infraestructura, rutas de despliegue, credenciales y cualquier dato que permita ubicar equipos o instalaciones específicas.

---

## 1. Descripción de la fase u objetivos desarrollados en la semana declarados en el cronograma

### 1.1 Contexto del proyecto

Esta semana giró en torno a tres actividades diferentes, una reconstrucción completa del ambiente de producción tras un reinicio de fábrica del servidor, la verificación del mecanismo de sincronización de hora entre el server y los CSI, y una sesión práctica de modelado de coordinación semafórica sobre el corredor donde se instalarán los próximos controladores.

### 1.2 Actividades declaradas en el cronograma para la Semana 6

Según el cronograma, la Semana 6 corresponde a las Actividades 5, 6 y 8:

- **Actividad 5.** Supervisar la integración entre hardware y software, verificando la correcta comunicación con controladores, dispositivos en campo y sistemas embebidos.
- **Actividad 6.** Proponer e implementar mejoras en la arquitectura del sistema, orientadas a incrementar la estabilidad, escalabilidad y facilidad de mantenimiento.
- **Actividad 8.** Optimizar la lógica de control semafórico, ajustando ciclos, transiciones y sincronización entre intersecciones para mejorar el flujo vehicular.

### 1.3 Objetivos efectivamente perseguidos

**Actividad 6 - cubierta.** El servidor de producción se reinició de fábrica, perdiendo todo lo que no estaba versionado en el repositorio: unidades de systemd, certificados, la llave de despliegue, el runner de integración continua y la base de datos completa. Se reconstruyó el stack completo en una sola sesión, usuario de servicio dedicado, llave de despliegue nueva, el runtime necesario, la base de datos aplicando las treinta migraciones con el runner versionado en vez de cargar un respaldo suelto, servidor web con HTTPS, el backend bajo systemd, y el pipeline de despliegue automático por integración a la rama principal, con reversión verificada. En el proceso se encontró un bug en la rama principal, el endpoint de salud "listo" regresaba error de forma permanente porque la lista de migraciones requeridas, mantenida a mano en el código, hacía referencia a una migración con numeración vieja que ya se había renumerado, sin que nadie lo notara por no correrse pruebas antes de integrar cambios. Se corrigió reemplazando la lista fija por una validación contra los archivos reales en disco, más una verificación automática antes de cada consolidación de cambios. Se documentó el procedimiento completo como referencia reutilizable.

**Actividad 5 - cubierta.** Se verificó el mecanismo de sincronización de hora entre el backend central y los controladores de campo. No usa el protocolo estándar de sincronización de red; es un mecanismo propio sobre HTTP, con un servicio de referencia de tiempo en el backend y un cliente en cada controlador que consulta periódicamente y ajusta el reloj del equipo según el tiempo de ida y vuelta de la consulta. Se validó en los dos controladores, ambos sincronizando correctamente contra la referencia del backend, con respaldo por reloj de tiempo real en caso de perder la conexión.

**Actividad 8 - cubierta parcialmente (fase de modelado y capacitación).** Se participó en una sesión práctica con una herramienta de ingeniería de tránsito para modelar sincronización semafórica, simulando la red completa de la ciudad con volúmenes de tránsito reales, enfocada en el corredor donde se instalarán los próximos controladores. Se aprendió a interpretar los diagramas tiempo-espacio y las bandas de sincronía entre intersecciones consecutivas y a lo largo de todo el corredor, con notas y capturas de referencia directa para la futura instalación. *Justificación de cobertura parcial:* fue una sesión de modelado y capacitación, no una implementación real de ajustes de ciclo, porque los controladores de ese corredor aún no están instalados en campo.

**Incidente no planeado.** Un corte de energía en la oficina a media semana interrumpió el resto de esa jornada de trabajo.

### 1.4 Relación entre el cronograma y el trabajo de la semana

Esta semana el cronograma y el trabajo realizado coincidieron, las tres actividades declaradas (5, 6 y 8) tuvieron avance, aunque la Actividad 8 quedó en fase de modelado y no de implementación, condicionada a que el hardware del corredor todavía no está instalado.

---

## 2. Descripción del desarrollo de las actividades semanales (metodología, recursos, procedimientos)

### 2.1 Metodología

Antes de asumir la causa de un problema se verificó el estado real del sistema en vez de confiar en el síntoma superficial, lo que parecía un reinicio ordinario del servidor resultó ser una máquina distinta, y solo se confirmó comparando tres señales concretas contra lo documentado (nombres de unidades de servicio, versión de sistema operativo y tipo de equipo). El mismo criterio se aplicó al bug de migraciones, se corrigió el mecanismo que producía el error, no solo el síntoma puntual.

### 2.2 Recursos y herramientas

- **Infraestructura:** sistema operativo Linux, base de datos relacional con su propio runner de migraciones versionado, servidor web con HTTPS, gestor de procesos del sistema operativo, integración continua con runner propio.
- **Comunicación de tiempo:** protocolo propio sobre HTTP entre el backend y los controladores de campo, con respaldo por reloj de tiempo real en cada controlador.
- **Modelado de tránsito:** Syncro8, herramienta de ingeniería de tránsito para simulación y coordinación semafórica (diagramas tiempo-espacio, bandas de sincronía).

### 2.3 Procedimientos seguidos y actividades realizadas

**Reconstrucción del ambiente de producción.** Ante la pérdida total del estado no versionado, se reconstruyó el stack completo siguiendo un orden verificado paso a paso (servicios de sistema, base de datos, servidor web, backend, pipeline de despliegue), confirmando cada pieza contra su comportamiento real antes de avanzar a la siguiente. El bug del endpoint de salud se corrigió alineando el código contra los archivos reales en disco y agregando una prueba y una verificación automática que impiden que se repita. Detalle completo en **Anexo A**.

**Verificación de sincronización de hora.** Se descartó primero un enfoque con el protocolo estándar de sincronización de red, por conflicto directo con el mecanismo propio ya implementado, y se validó en su lugar el mecanismo real contra los dos controladores, confirmando el camino normal y el de respaldo. Detalle completo en **Anexo B**.

**Sesión de modelado de coordinación semafórica.** Se asistió a una sesión práctica de simulación de la red semafórica completa, con foco en el corredor de instalación próxima, y se documentó con notas y capturas de pantalla el comportamiento de las bandas de sincronía por intersección, como referencia para la fase de instalación.

### 2.4 Evidencia

El respaldo de lo descrito son los reportes técnicos redactados durante la semana. Son documentos internos del proyecto, así que no se anexan completos por el acuerdo de confidencialidad.

Se anexan en versión con la información sensible omitida:

- **Anexo A.** Reconstrucción del ambiente de producción y corrección del bug de migraciones en la rama principal.
- **Anexo B.** Verificación de la sincronización de hora entre el backend y los controladores de campo.

La sesión de modelado de coordinación semafórica se respalda con notas y capturas propias, sin reporte formal por tratarse de una sesión de capacitación.

---

## 3. Listado de las actividades siguientes o pendientes no resueltas, con justificación

### 3.1 Estado de las actividades declaradas para la semana

**Actividad 8, optimizar la lógica de control semafórico.** Cubierta parcialmente. *Justificación:* el corredor trabajado esta semana en el modelado todavía no tiene controladores instalados en campo, así que no hay ajustes reales de ciclo que aplicar todavía. *Plan de acción:* retomar en cuanto el hardware del corredor esté instalado. *Resultado esperado:* usar el modelado de esta semana como referencia directa para el primer ajuste de ciclos real.

### 3.2 Pendientes generados en la semana

**1. Rotar las credenciales que quedaron expuestas durante la reconstrucción.** *Justificación:* varias contraseñas quedaron en el historial de la sesión de trabajo. *Plan de acción:* rotarlas y retirar cualquier credencial temporal usada para destrabar la reconstrucción.

### 3.3 Actividades cercanas

Según el cronograma, la Semana 7 repite las mismas Actividades 5, 6 y 8. Lo concreto para la semana es dar seguimiento a los pendientes de esta semana y continuar el modelado del corredor conforme avance la instalación de sus controladores.

---

## 4. Autorreflexión personal: problemas enfrentados, aprendizajes y desempeño

### 4.1 El patrón que más se repitió esta semana

El síntoma superficial no siempre indica la causa real, lo que parecía un reinicio ordinario del servidor resultó ser una máquina distinta por completo, y solo tres verificaciones concretas contra lo documentado lo confirmaron. El mismo cuidado, verificar el estado real antes de asumir, aplicó también al corregir el bug de migraciones.

### 4.2 Errores propios y lo que aprendí de cada uno

Una operación destructiva sobre la base de datos, hecha sin avisar al resto del equipo, generó medio día de trabajo en paralelo bajo supuestos distintos entre dos personas que asumían cosas distintas sobre el mismo sistema. El aprendizaje es avisar antes de recrear cualquier recurso compartido, aunque parezca una operación de rutina.

### 4.3 Aprendizajes profesionales y consideraciones éticas

- Ante un bug encontrado en la rama principal, se corrigió el mecanismo que lo produjo (la validación contra archivos reales) y no solo el síntoma puntual, para que la misma clase de error no se repita con la siguiente migración.
- Se documentó el procedimiento completo de reconstrucción como referencia reutilizable, en vez de dejar el conocimiento solo en quien lo ejecutó esta vez.

### 4.4 Sobre el cronograma

Esta semana el cronograma coincidió bien con el trabajo real en las tres actividades declaradas, con la salvedad de que la Actividad 8 avanzó en su fase de modelado y no todavía en implementación, por depender de una instalación de hardware que aún no ocurre.

### 4.5 Desempeño, bienestar y acciones que voy a mantener

La semana combinó una reconstrucción de infraestructura bajo presión con una sesión de aprendizaje directamente aplicable al siguiente corredor de instalación. Un corte de energía a media semana interrumpió una jornada completa, sin que eso afectara el resto de los entregables.

Acciones que se van a sostener:

- Verificar el estado real del sistema antes de asumir la causa de un problema.
- Avisar al equipo antes de cualquier operación destructiva sobre un recurso compartido.
- Documentar procedimientos como referencia reutilizable, no solo como bitácora de lo ya hecho.
