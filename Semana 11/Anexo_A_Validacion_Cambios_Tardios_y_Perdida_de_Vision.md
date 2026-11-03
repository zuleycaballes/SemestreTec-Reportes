Anexo A - Reporte de validación: cambios tardíos y pérdida de visión en la transición hacia el escenario con detector

Fecha de la corrida: 26 de octubre de 2026
Origen: documento interno de validación, entregado al responsable del proyecto.

> Versión con datos omitidos. Este anexo reproduce el contenido técnico del reporte interno omitiendo direcciones de red, identificadores de nodo, de controlador y de escenario, nombres de archivos, de scripts, de campos, de servicios, rutas y comandos internos, versiones y huellas de código, conforme al acuerdo de confidencialidad. Lo que se conserva es el problema, el razonamiento, los hallazgos y las verificaciones hechas.

---

## Resumen y veredicto

**Cumplido en el controlador de laboratorio, con alcance limitado.** Se comprobó cómo decide el controlador cuando la demanda cambia, o la visión falla, cerca de la transición hacia el escenario que depende del detector: una demanda positiva que llega tarde, una que desaparece después de aceptada la entrada, y la caída o desconocimiento de la visión con la política configurada. Todo se ejecutó en laboratorio y no hubo ningún cruce activo.

```text
Corrida en el controlador de laboratorio: OK
Pruebas: 7 de 7 aprobadas; seguimientos: 7 de 7 aprobados; ciclos inválidos: 1 (no cuenta)
Contraste con la Auditoría: pendiente (opcional)
```

| Dato de la corrida | Valor |
| --- | --- |
| Fecha y duración | 26 de octubre, de 15:09 a 15:30 (hora de Mazatlán), unos 21 minutos |
| Dónde se ejecutó | Dentro del propio controlador, con el puente de visión pausado |
| Cuadros y envíos de estado | 12,994 cuadros medidos; 2,483 envíos, ninguno rechazado (9.1 ms de mediana) |
| Configuración | Ciclo de 80 s con 4 escenarios de 20 s; castigo "extender el siguiente"; parpadeo 3 s y amarillo 3 s sin rojo de despeje (ventana de cierre de 6 s); política ante detector desconocido: atender |

**Primer intento descartado.** Se hizo desde la computadora por Wi-Fi y terminó como evidencia insuficiente sin evaluar ninguna prueba: 35 % de pérdida de paquetes, 891 ms de mediana por envío, envíos agotados y el flujo de estado caído. No se usó como evidencia sobre el firmware. La corrida válida se ejecutó dentro del controlador.

## Cómo decide el firmware cerca de la transición (leído del código)

- **Ventana de cierre y congelado.** La ventana abre cuando el tiempo restante del escenario es igual al despeje de los grupos que cierran. En el primer tick de la ventana la decisión de la próxima entrada se congela, y una admisión nunca se revoca por perder la detección después.
- **Rescate.** Una decisión congelada en "castigar" pasa a "atender" solo si hay demanda positiva verificada (visión en línea y detector activo, o un latch) **y** el despeje cabe en el tiempo restante. Con "extender el anterior" el rescate cierra grupos que el castigo dejaba abiertos y exige el despeje completo; con "extender el siguiente" esos grupos cierran igual y el rescate cabe siempre.
- **Diferido.** Si la demanda llega pero el despeje ya no cabe, el escenario se castiga en ese ciclo y el latch se conserva para atenderlo en el siguiente.
- **El envío fuerza un tick.** Enviar un estado del detector provoca un tick inmediato con menos tiempo que el despeje completo, por lo que con "extender el anterior" el rescate en vivo casi no cabe.
- **Política ante detector desconocido.** Vale la del escenario o la global: atender y respaldo atienden, omitir no atiende y la llamada periódica alterna. Un latch pendiente le gana a cualquier política.
- **Caída de visión.** Cuenta como caída si pasan más de 2.0 s sin datos. Una caída o un detector desconocido **nunca rescatan**, porque no son detección verificada.

Un documento de diseño anterior describe el comportamiento previo a la verificación del despeje y ya no es fiel, así que no se usó como expectativa.

## Criterios de aceptación

El escenario consumidor es T y el anterior I. Una prueba cuenta cuando tiene los ciclos pedidos aprobados; un ciclo inválido no cuenta ni a favor ni en contra.

| Caso | Qué se inyecta | Criterio principal |
| --- | --- | --- |
| Calibración | Nada | Ventana en el tiempo esperado, cuadros sin huecos, T castigada por falta de demanda, sin latch ni detector |
| 1. Rescate | Falso a verdadero dentro de la ventana; vuelve a falso antes de T | El valor tardío se ve dentro de la ventana. T rescatada (si se abrió con cupo) o diferida; si es diferida, latch conservado y servido el ciclo siguiente. Un rescate nunca trunca el despeje |
| 2a. Desaparece en la ventana | Verdadero antes de la ventana, falso dentro de ella | T admitida aunque ya no hay detección; latch consumido una sola vez |
| 2b. Desaparece en la entrada | Verdadero hasta entrar a T, falso dentro de T | T admitida de principio a fin: soltar el detector no revoca |
| 3a / 3b. Cámara caída / detector desconocido | La falla desde antes de la ventana hasta entrar a T | T según la política configurada, con motivo distinto de "sin demanda"; sin detección inventada |
| 3c / 3d. La falla llega con la decisión congelada | La falla dentro de la ventana | T castigada por falta de demanda: la caída no rescata como si fuera una detección real |

## Resultados en el controlador de laboratorio

| Prueba | Veredicto | Medido en el flujo de estado |
| --- | --- | --- |
| Calibración | Aprobada | Ventana de 6.0 s; T castigada por falta de demanda, sin latch ni detector |
| 1. Rescate | Aprobada, **rescatada** con cupo | El valor tardío se vio en el primer cuadro con la ventana abierta, con el latch armado en el mismo cuadro. T atendida, sin castigo, 207 cuadros estables, latch consumido desde el primer cuadro |
| 2a. Desaparece en la ventana | Aprobada | El falso se ve dentro de la ventana y se sostiene 5.4 s hasta T. T atendida con el motivo del latch pendiente, 206 cuadros; latch consumido una vez |
| 2b. Desaparece en la entrada | Aprobada | Falso visible a 1.1 s dentro de T (194 de 205 cuadros). T atendida de principio a fin |
| 3a. Cámara caída | Aprobada | Caída antes de abrir la ventana. T atendida por la política de atender, sin fuentes, latch ni detector |
| 3b. Detector desconocido | Aprobada | Igual que 3a, con la visión en línea |
| 3c / 3d. Falla tardía | Aprobadas, **castigadas** | La falla se ve en el cuadro que abre la ventana, unos 190 ms después del disparo. T castigada por falta de demanda, sin fuentes ni latch |
| Seguimiento | 7 de 7 aprobados | Cada ciclo posterior vuelve a "sin demanda confirmada", sin latch ni detector |

Un solo ciclo se marcó inválido por fase (hueco de 23 ms entre la frontera y el siguiente tick), en la prueba 3d, y se repitió.

**Lectura.** Caso 1: el escenario fue rescatado y el latch se consumió una vez; con "extender el siguiente" esta corrida no ejercita el chequeo de despeje completo. Caso 2: el escenario se mantuvo atendido sin importar cuándo desapareció la detección. Caso 3: con la falla desde antes de la ventana, T aplicó la política configurada sin inventar una detección. Falla dentro de la ventana (3c y 3d): T queda castigada y la política no se aplica ese ciclo porque la decisión ya estaba congelada; con la política de atender, habría sido servida si la falla hubiera llegado antes. **Conviene confirmarlo con el responsable.**

**Alcance.** Un ciclo válido por prueba, una sola configuración y un solo caso de rescate. No cubre las otras políticas, otros escenarios ni "extender el anterior" en el laboratorio. La corrida fue local al controlador, por lo que los 9 ms de latencia no representan la red. Sin contraste con la Auditoría.

## Resultados en simulación y sensibilidad de la prueba

- **Simulación con el servicio real.** Con "extender el anterior" todas las pruebas se aprobaron y el rescate quedó **diferido** en los 2 ciclos (0 rescates reales). Con una réplica de la configuración de laboratorio y "extender el siguiente", el rescate fue **rescatado** en 6 de 6 ciclos. Las dos ramas están cubiertas.
- **Fallas inyectadas.** Sobre copias del motor, no sobre el repositorio, se inyectaron siete fallas, una por una: reevaluar la decisión congelada, revocar una admisión congelada, ignorar la política, tratar lo no confiable como confiable, rescatar con demanda de seguridad, rescatar sin comprobar el despeje y descartar el latch diferido. El script detectó las siete.

## Continuidad

- Contraste opcional con la Auditoría, sin hacer: las decisiones de castigo de cada ciclo deben coincidir con los motivos medidos, cruzando por identificador de ejecución y no por hora.
- Confirmar con el responsable el comportamiento de la falla que llega con la decisión congelada.
- Repetir con más ciclos, con "extender el anterior" en el laboratorio y con las políticas de omitir, respaldo y llamada periódica.
- Decisión para el equipo de firmware: con "extender el anterior" el caso 1 en vivo termina diferido y no rescatado. Si debe rescatarse también ahí, el envío no debe consumir el cupo. Hoy el contrato efectivo es "diferida, con latch conservado" y así lo valida el script. Hay que actualizar el documento de diseño anterior.
- Operación: la prueba solo se corre en el controlador de laboratorio y nunca en el de calle; se pausa el puente de visión durante la prueba y debe reactivarse al terminar. La evidencia (registro por cuadro, captura cruda y resumen) queda fuera del repositorio.
