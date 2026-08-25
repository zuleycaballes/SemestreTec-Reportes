# Anexo B - Reporte de cierre: ventana de detección, fase 1 (lectura real)

**Fecha del cierre:** 21 de agosto de 2026
**Origen:** documento interno de cierre de fase, entregado al responsable del proyecto.

> **Versión con datos omitidos.** Este anexo reproduce el contenido técnico del reporte interno omitiendo nombres de bases de datos, identificadores de nodos reales, rutas internas y observaciones de configuración que se escalaron por canal interno, conforme al acuerdo de confidencialidad. Lo que se conserva es el problema, el razonamiento, los defectos encontrados y las verificaciones hechas.

---

## Qué es esta fase

La ventana de detección es la pantalla desde la que se configura qué cámara va relacionada con cada semáforo de una intersección, y sobre qué parte de la imagen se buscan los vehículos. La versión anterior estaba organizada por catálogo de detectores; el rediseño la organiza por dirección semafórica, cámara y región de interés.

El reemplazo se planeó en cinco fases, ordenadas de modo que escribir sea lo último. Esta es la primera, donde la ventana lee la configuración real del nodo y no escribe absolutamente nada.

**Cero archivos de backend.** Las tres rutas necesarias ya existían desplegadas.

---

## 1. Lo que resultó no ser un problema

El identificador de dirección semafórica que el rediseño necesitaba ya existía en la base de datos desde una migración anterior, igual que la tabla de asignaciones cámara–dirección con su vigencia, y las tres rutas de servicio. Todo el rediseño se construyó sobre un campo que ya estaba; solo faltaba conectarlo.

Vale la pena anotarlo porque la suposición inicial era la contraria, y la revisión del esquema desplegado antes de escribir código ahorró una migración innecesaria.

---

## 2. Cumplimiento de la fase

| Criterio | Estado | Verificación |
|---|---|---|
| Lee direcciones reales del nodo | Cumple | Cuatro direcciones con sus nombres traídos de base de datos, no del simulado |
| Lee asignaciones vigentes | Cumple | Sin asignaciones en pruebas, el panel lo declara correctamente |
| Lee el inventario de cámaras del nodo | Cumple | Cuatro cámaras dadas de alta en pruebas, todas como disponibles |
| Resuelve el controlador por el identificador correcto | Cumple | Verificado con datos deliberadamente desfasados |
| Lee las regiones de la configuración del controlador | Cumple | Resueltas por el identificador correcto |
| No escribe nada | Cumple | Confirmar, liberar cámara y dibujar región quedan deshabilitados, con la razón a la vista |
| Cero archivos de backend | Cumple | — |

---

## 3. Hallazgo técnico: un identificador se truncaba antes de llegar al adaptador

El rediseño normaliza el identificador de dirección a cadena de texto, con pruebas que lo cubren. Esa protección llegaba tarde.

El analizador de JSON de JavaScript convierte cualquier número a punto flotante, y ese tipo pierde precisión a lo largo del tiempo. Un identificador que exceda ese límite ya viene truncado cuando el adaptador lo recibe, y en ese punto no hay forma de recuperarlo, se normaliza a cadena un valor que ya es incorrecto.

Se resolvió en la capa de lectura, la respuesta se lee como texto y los identificadores largos se entrecomillan antes de analizarse. La función vive en un módulo sin dependencias, probado aparte.

**Hoy es protección.** Los identificadores reales arrancan en 1, de modo que caben de sobra. Empieza a importar si alguna vez se siembran identificadores altos.

Lo detectó una prueba, no la revisión, la prueba construía el escenario con un número escrito directamente y el lenguaje lo truncó antes de que el código lo tocara.

---

## 4. Defectos corregidos durante la verificación

**La vista previa pedía el identificador equivocado.** El panel resolvía la imagen y el estado de transporte con el identificador del inventario, mientras que el controlador indexa por su identificador local. Pedirle una cámara por el primero es pedirle una que no conoce.

Es un riesgo que quedó señalado por escrito en dos documentos previos del mismo bloque, y aun así se coló al escribir el componente.

**Solo se pudo ver por cómo se prepararon los datos.** Las primeras cuatro cámaras de prueba quedaron con los dos identificadores idénticos, porque la tabla estaba vacía y ambas secuencias arrancaban en 1. Con esos datos el defecto es invisible. Se reinsertaron desplazando una de las secuencias, de modo que los dos identificadores dejaran de coincidir; con eso, la petición mostró el valor equivocado de inmediato.

**Un escenario de prueba donde dos identificadores distintos coinciden por casualidad no prueba que se distingan.**

**Tres importaciones incorrectas.** Se supuso la interfaz de dos módulos en vez de leerla. La herramienta de validación habitual no resuelve importaciones, por lo que no las habría detectado; se agregó una comprobación que contrasta cada importación contra lo que el archivo destino realmente exporta.

---

## 5. Estado del ambiente de pruebas

La base de datos de desarrollo tenía catorce tablas y ningún registro de migraciones aplicadas, el esquema correspondía a un punto intermedio, pero el ejecutor de migraciones no lo sabía.

Aplicar directamente habría intentado recrear tablas existentes. Se marcó una línea base, determinada comparando el esquema real contra la definición de cada migración, y después se aplicaron las quince restantes sin fallos.

Se dieron de alta cuatro cámaras de prueba con los dos identificadores deliberadamente desfasados. No corresponden a hardware real, así que la petición de imagen devuelve un error de recurso no encontrado, que es lo esperado. Deben retirarse cuando dejen de ser útiles.

---

## 6. Verificaciones

| Verificación | Resultado |
|---|---|
| Transformación del código de interfaz | Sin errores |
| Pruebas de lógica | 87 de 87, en seis suites |
| Importaciones contra lo realmente exportado | Sin faltantes |
| Verificación en pantalla contra un nodo real | Direcciones, posiciones y modo lectura correctos |

Las veintidós pruebas nuevas cubren la adaptación de la respuesta del servicio, la preservación del identificador largo, la separación entre los dos identificadores de cámara, el descarte de credenciales, y la resolución del identificador que se envía al controlador.

**Cuatro de esas pruebas se reportaron como pasando sin haberse ejecutado:** quedaron escritas después del punto donde el script terminaba, de modo que la ejecución nunca las alcanzaba. Se reubicaron y pasan. Una suite que termina en éxito no garantiza que haya corrido lo que se agregó al final.

---

## 7. Continuidad

**No verificado, sin bloquear:** el croquis con brazos sin dirección asignada. Todos los nodos activos de la base de pruebas tienen las cuatro direcciones, así que no hubo con qué probarlo. Un nodo con menos direcciones dejaría brazos vacíos y no se sabe si eso se lee como ausencia legítima o como falla.

De la misma consulta se desprende que ningún nodo activo tiene direcciones diagonales, de modo que la limitación del croquis fijo a cuatro direcciones hoy no afecta a ninguno.

**Decisión pendiente de la fase 3:** la tabla que traduce cada dirección al identificador que usa el controlador fija qué significa cada una. Mientras el dato solo indique presencia no cambia nada; importa cuando el módulo que controla las luces lo consuma para decidir qué fase del semáforo extender. Debe confirmarlo quien lleva firmware antes de implementar esa fase.

Ninguna entrega de esta fase modifica los contratos entre módulos.