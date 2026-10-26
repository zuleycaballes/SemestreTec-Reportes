Anexo B - Reporte de validación: la detección cambia lo que ejecuta el semáforo (recepción de estados, castigo por ausencia y latch)

Fechas de las corridas: 20, 21 y 23 de octubre de 2026
Origen: documentos internos de validación, entregados al responsable del proyecto.

> Versión con datos omitidos. Este anexo reproduce el contenido técnico de los reportes internos omitiendo direcciones de red, identificadores de nodo y de controlador, nombres de archivos, de scripts, rutas y comandos internos, versiones y huellas de código, y los nombres internos de cada escenario, conforme al acuerdo de confidencialidad. Lo que se conserva es el problema, el razonamiento, los hallazgos y las verificaciones hechas.

---

## Resumen y veredicto

**Cumplido para el detector y el escenario configurados en laboratorio.** Se comprobó, en simulador y en el controlador real de laboratorio, que el firmware recibe y conserva los estados del detector, que el detector cambia la ejecución del escenario (castigo por ausencia) y que una detección breve de 1 segundo no se pierde antes de que llegue el escenario que la consume (latch). Todo se ejecutó en laboratorio y no hubo ningún cruce activo.

| Validación | Fecha | Donde | Resultado |
| --- | --- | --- | --- |
| Recepción de estados del detector | 20 y 21 de octubre | Simulador y controlador real | 39 de 39 en simulador; 26 de 26 en el controlador real |
| Servicio normal y castigo por ausencia | 21 de octubre | Controlador real y Auditoría | 3 de 3 con demanda y 3 de 3 sin demanda, contrastadas con la Auditoría |
| Latch de detección | 23 de octubre | Controlador real | 3 inyecciones y 3 controles, todos aprobados |

## Cómo decide el motor

1. La demanda de un escenario con detector sale **solo de un latch pendiente**. Un detector visto encendido en vivo no basta por sí mismo.
2. El latch se arma cuando el detector está activo mientras corre **otro** escenario del ciclo, o por el flanco de subida de un envío de estado. Dentro de su propio escenario no se arma.
3. Al entrar el escenario con demanda se consumen sus latches, una vez por visita. El latch no caduca por tiempo.
4. La decisión de entrada se **congela en la ventana de cierre del escenario anterior**. Sin demanda, el castigo configurado decide qué escenario se ejecuta de forma efectiva.
5. El estado publicado separa el escenario programado del efectivo. El castigo que se muestra como configurado no prueba que se haya aplicado; la prueba es el escenario efectivo más la bandera de castigo aplicado.

## 1. Recepción de estados del detector

El envío tiene el formato: fuente, versión del esquema, secuencia, cámara en línea, mapa de detectores completo con valores booleanos estrictos y un mapa opcional de detectores desconocidos. La secuencia debe crecer.

| Caso | Respuesta | Estado tras el envío |
| --- | --- | --- |
| Detector encendido | 200, secuencia aceptada 1 | El estado publicado muestra el detector encendido |
| Detector apagado | 200, secuencia aceptada 2 | El estado publicado lo muestra apagado |
| Envío incompleto | 422 | Sin cambios |
| Detector desconocido | 422 | Sin cambios |
| Secuencia repetida | 409 | Sin cambios |
| Valor no booleano (1, 0, "true", nulo) | 422 | Sin cambios |

Tras cada rechazo, el mapa de detectores y la secuencia son idénticos al último estado válido, el modo se mantiene en normal, el contador de estado sigue avanzando y un envío válido posterior se acepta.

- **Simulador:** 39 de 39 verificaciones con dos configuraciones de detectores. El flujo de estado por WebSocket (11 de 11) y el modo estricto de contratos también pasaron. El conjunto de pruebas del contrato quedó en 27 pruebas, con 2 nuevas.
- **Controlador real:** 26 de 26 verificaciones. El controlador de laboratorio declara un solo detector, por lo que la prueba usó ese. Al terminar, todos los detectores quedaron en apagado y la cámara se reactivó.
- **No probado en el controlador real:** el flujo de estado por WebSocket y el modo estricto, porque implicaban reiniciar el servicio con otra variable de entorno.

## 2. Servicio normal y castigo por ausencia

Configuración de referencia: un ciclo de 60 segundos con tres escenarios de 20, un detector en el segundo escenario y castigo "extender el anterior". Se ejecutan dos casos y se mide sobre **todos** los cuadros de cada ejecución.

| Criterio | Caso A, con demanda | Caso B, sin demanda |
| --- | --- | --- |
| Escenario programado | 2 | 2 |
| Escenario efectivo | 2 | 1 |
| Demanda | Verdadera | Falsa |
| Castigo aplicado | No | Sí |
| Decisión congelada | Sí | Sí |
| Motivo | Contiene "latch" | Contiene "sin demanda" |
| Fuente de la demanda | El detector | Ninguna |
| Salida física de luces | Verde del escenario 2 | Verde del escenario 1 |

**Resultado en el controlador real (21 de octubre, unos 6.5 minutos):** 3 ejecuciones válidas por caso, todas aprobadas, más una ejecución inválida de arranque en el caso A, que es esperada. En el caso B cada ejecución del escenario 2 duró 20.0 segundos con el escenario 1 efectivo, así que el ciclo conserva sus 60 segundos. No hubo huecos en el flujo (intervalo máximo entre cuadros de 0.12 s), 4,035 cuadros y 736 envíos de estado sin ninguno rechazado, con un único emisor de detectores.

**Contraste con la Auditoría del sistema central.** Se revisaron 10 eventos de decisión de castigo de la serie y se abrieron dos con el detalle. Coinciden el escenario programado y el efectivo, la demanda, el castigo, las fuentes, el motivo y el modo de castigo. La demanda del caso A vino de la prueba y no de la cámara. El reloj de la computadora parece haber ido unos 27 segundos desfasado respecto al del controlador (no se midió), por lo que el cruce se hizo por identificador de ejecución y no por hora.

**Simulación.** El castigo "extender el siguiente" pasó con 3 y 3. Para comprobar que el script falla cuando el detector no hace nada, se inyectaron fallas sobre copias del motor: si el detector siempre cuenta como demanda, falla el caso sin demanda; si nunca cuenta, falla el caso con demanda y el otro queda inválido.

## 3. Latch de detección

Se envía una detección de 1 segundo durante el escenario anterior al que la consume, se apaga, y se espera a que entre el escenario consumidor.

| Paso | Criterio | Resultado en las 3 inyecciones |
| --- | --- | --- |
| 1 | El detector se ve encendido | A los 15, 16 y 30 ms |
| 2 | El latch aparece con sus 5 campos y sin haber sido atendido | En el mismo cuadro |
| 3 | El detector se ve apagado | A los 48, 32 y 25 ms |
| 4 | El latch permanece, con el detector apagado | En el 100 % de los cuadros (191, 190 y 194; 180, 178 y 182 ya con el detector apagado) |
| 5 | El detector llevaba al menos 1 s apagado al entrar | Margen de 17.3, 17.3 y 17.4 s |
| 6 | El escenario consumidor entra sin castigo | Programado igual a efectivo, demanda verdadera, motivo de latch pendiente, detector apagado en todos los cuadros |
| 7 | El latch desaparece una sola vez | No aparece desde el primer cuadro del escenario (0 de 209, 207 y 205) y no reaparece |

**Ciclo de control** (sin detección): el escenario consumidor se castiga, el efectivo es el anterior, la demanda es falsa, no hay fuentes y no queda ningún latch, en los 3 ciclos. Esto demuestra que el consumo es único.

**Resultado en el controlador real (23 de octubre, unos 7 minutos):** 3 inyecciones y 3 controles, todos aprobados, con 3,990 cuadros y 737 envíos de estado sin ninguno rechazado. Antes de las pruebas del controlador real, las pruebas del firmware ya pasaban (57 pruebas, 105 subcasos) sin modificar ningún archivo del firmware.

**Simulación.** Se repitió en el servicio real en modo virtual con 3 y 3 aprobados. Con fallas inyectadas sobre copias del motor, el script falla en los pasos esperados: si el latch se borra al apagar el detector, las 3 inyecciones fallan en los pasos 4 y 6; si el latch nunca se consume, la inyección falla en el paso 7 y los controles quedan inválidos.

## Alcance y límites

- Se demostró que una detección de 1 segundo del detector configurado, durante el escenario anterior, queda en el latch, sobrevive a que el detector se apague, hace que el escenario consumidor entre sin castigo y se consume una sola vez.
- **No cubre:** otros detectores ni otros escenarios, pulsos más cortos que 1 segundo, los castigos distintos de "extender el anterior" (el siguiente solo en simulador, "escenario específico" sin correr y los castigos por reparto no soportados por el script), ni cámara real en el caso sin demanda.
- **Observaciones de diseño abiertas que afectan estas pruebas:** una detección que empieza dentro de la propia entrada del escenario no sirve a ese escenario ni se guarda, y el latch no caduca por tiempo.

## Evidencia y cómo repetir

Cada corrida deja tres archivos de captura: el registro por cuadro, la captura cruda comprimida y el resumen con el veredicto. El código de salida es 0 si todo es correcto, 1 si algo falla y 2 si la evidencia es insuficiente. Para repetir, se pausa el puente de la cámara, se confirma que el controlador es el de laboratorio y se ejecuta el script con el número de ciclos deseado y un prefijo distinto de salida, porque con el mismo prefijo se sobrescriben los archivos. Las direcciones, el controlador y los nombres de los scripts se omiten en esta versión.
