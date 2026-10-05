Anexo B - Reporte de resultado: prueba PC-10, flujo completo de Sincronía desplegada

Fecha del cierre: 2 de octubre de 2026
Origen: documento interno de resultado de prueba, entregado al responsable del proyecto.

> Versión con datos omitidos. Este anexo reproduce el contenido técnico del reporte interno omitiendo direcciones de red, identificadores de nodo y de controlador, versiones de despliegue, huellas (hash), nombres de base de datos y rutas de configuración, conforme al acuerdo de confidencialidad. Lo que se conserva es el problema, el razonamiento, los hallazgos y las verificaciones hechas.

---

## Decisión

**Aprobado con parcial.** El paso 14 y el caso "identificador de controlador omitido en un nodo con dos controladores" (paso 8e) quedan pendientes de hardware multicontrolador, con la ejecución autorizada por jefatura.

## Contexto

| Campo | Dato |
|---|---|
| Fechas de ejecución | 29 de septiembre y 2 de octubre de 2026 |
| Ambiente | Servidor de producción, con el despliegue vigente en cada fecha (los pasos 8d, 10 con envío sin destello, 12 y 13 se ejecutaron sobre el despliegue del 2 de octubre) |
| Migraciones requeridas | Aplicadas sin error |
| Nodo | Nodo de laboratorio |
| Controlador canario | Controlador de laboratorio, con un solo controlador en el nodo |
| Controlador no tocado | Controlador de calle: se usó solo como identidad de texto y como identidad de prueba negativa, sin lectura fresca del equipo |

## Preparación y hallazgos previos

1. **Bloqueante 1 (corregido).** Al abrir Sincronía, la app leía el equipo con lectura fresca y esa lectura sobrescribía en la base de datos el borrador guardado sin enviar, por lo que el usuario perdía su trabajo. La decisión fue no leer automáticamente si hay un borrador pendiente y preguntar si se envía para reescribir el equipo. Se corrigió en el frontend (lógica de lectura, pantalla y estilos, más pruebas del estado del borrador) y se integró a la rama principal; el archivo nuevo del estado del borrador estaba presente en el despliegue verificado.
2. **Identidad entre el sistema central y el equipo (resuelta).** El controlador de laboratorio reportaba la identidad del ambiente de desarrollo. Con las migraciones aplicadas, el sistema central separa tres identidades, el identificador interno del registro, el puesto lógico del controlador y la identidad aplicada en el equipo. Los controladores se dieron de alta de nuevo, y los registros anteriores quedaron inactivos. La identidad aplicada del controlador de laboratorio se fijó automáticamente en la primera lectura y coincide con lo que el equipo reporta en su estado y su configuración.

## Resultados por paso

| # | Paso | Esperado | Observado | Resultado |
|---|---|---|---|---|
| 1 | Identificador del controlador en la URL y en las respuestas | El identificador viaja en la URL y en las respuestas HTTP | La URL lleva el controlador y la consulta de configuración responde 200 con el identificador | Cumple (ver O4) |
| 2 | Cargar desde el equipo con lectura fresca | Programa, formato v1 y horarios del equipo | Ciclo 2 con 40 s en una vía y 20 s en la otra, modo fijo, sin horarios configurados | Cumple |
| 3 | Editar y guardar sin enviar | Solo cambia la fila de ese controlador | La fila pasa a origen local y sin huella de envío; la fila del controlador de calle no cambia; la fecha de actualización del nodo no cambia al guardar | Cumple |
| 4 | Recargar y conservar el borrador | El borrador es igual tras recargar | Aparece el aviso de borrador guardado sin enviar; el origen sigue local y la fecha de última lectura no cambia al recargar | Cumple (requirió el fix) |
| 5 | Exportar a JSON y a PDF | JSON v1; el PDF no suma las transiciones | JSON con versión 1, identidad del equipo y llaves canónicas; el PDF del ciclo 2 suma 60 s (40 + 20) sin sumar los 3 s de amarillo | Cumple |
| 6 | Importar y reconstruir | Sin pérdida | "JSON v1 aplicado, codec del backend"; de constructor a JSON, idéntico (64 de 64 líneas, 0 diferencias); tras guardar sigue "EN SYNC CON CSI" | Cumple (ver O2) |
| 7 | Cambios pendientes | Confirmación antes de perder cambios | "Volver" pide confirmación; recargar dispara la advertencia del navegador; al revertir desaparece "Sin guardar" | Cumple (el selector de controlador no aplica: hay uno por nodo) |
| 8a | Controlador inactivo | 409 | 409 de controlador inactivo | Cumple |
| 8b | Controlador no asociado al nodo | 409 | 409 de controlador no asociado (el de calle, en el nodo de laboratorio) | Cumple |
| 8c | Controlador inexistente | 404 | 404 de controlador no encontrado | Cumple |
| 8d | Controlador sin conexión | 502 y sin escritura | Se simuló con una regla temporal de firewall con reversión automática verificada. La interfaz muestra "CSI Offline" y oculta las acciones contra el equipo; la API responde 502 por conexión rechazada | Cumple (ver O5 y O6) |
| 8e | Identificador omitido en un nodo con dos controladores | 409 por ambigüedad | No ejecutable: no existe un nodo multicontrolador | Parcial |
| 8f | JSON de otro controlador | Confirmación explícita | Con la identidad del controlador de calle pide confirmar que "pertenece a otro controlador"; al cancelar no se aplica | Cumple (ver O3) |
| 9 | Registro antes del envío | Fecha de envío, huella enviada y último registro | Se registran antes de cada envío | Cumple |
| 10 | Envío al canario | Envío aceptado | Los envíos del 29 de septiembre fueron con destello de entrada; el del 2 de octubre fue sin destello, con éxito y etapa final | Cumple (ver O8) |
| 11 | Releer y comparar la huella | Converge | Tras el envío, la relectura devuelve una huella igual a la enviada y la comparación converge | Cumple |
| 12 | Se actualiza solo el controlador enviado | Solo cambia el de laboratorio | Comparación de la foto final contra la inicial: el controlador de calle queda idéntico, sin envíos; las filas inactivas no cambian; solo cambió la del controlador de laboratorio | Cumple |
| 13 | Un rechazo no registra envío | Fecha y huella sin cambio | Un envío con lista de ciclos vacía responde 400 de programa inválido sin contactar al equipo; la fecha y la huella de envío no cambian; el registro de envíos guarda el intento fallido en la etapa de validación, con mensaje resumido y sin huella | Cumple (ver O7) |
| 14 | Aislamiento entre dos controladores del mismo nodo | El segundo no cambia | No ejecutable: no existe un nodo multicontrolador | Parcial (pendiente del hardware) |

## Criterios de terminado

| Criterio | Estado |
|---|---|
| Todo el flujo atraviesa el codec del backend | Cumple |
| Una configuración guardada vuelve igual | Cumple (con el fix del borrador) |
| El envío correcto converge entre el sistema central y el equipo | Cumple |
| Los rechazos no se registran como envíos exitosos | Cumple |
| Dos controladores distintos permanecen aislados | Parcial: el aislamiento entre controladores de nodos distintos está verificado (paso 12); el del mismo nodo queda pendiente del hardware multicontrolador |

## Observaciones (no bloquean)

- **01.** El inspector de la pestaña JSON mostró en una ocasión 68 s para un ciclo de 60 s. El PDF y el equipo coinciden en 60 s. Falta revisar el cálculo del inspector.
- **02.** Tras aplicar un JSON idéntico al guardado aparece "Sin guardar". La configuración no cambia (diferencia 0, misma huella): es un falso positivo de la comparación por texto en la interfaz.
- **03.** Al cancelar la copia de un JSON de otro controlador, el mensaje dice "JSON inválido", aunque el JSON es válido.
- **04.** Con un solo controlador en el nodo, la URL no incluye el controlador hasta que se usa el selector; con el parámetro explícito funciona.
- **05.** La lectura sin conexión responde 502 sin un código estructurado, a diferencia de los otros rechazos.
- **06.** Con el equipo sin conexión, el banner sigue mostrando "SEMÁFORO EN OPERACIÓN NORMAL" (último estado conocido).
- **07.** Los rechazos tempranos quedan en el registro de envíos con el modo de aplicación vacío, porque los ciclos se validan antes de leer el modo.
- **08.** Los primeros envíos al canario (29 de septiembre) fueron con destello de entrada; se repitió sin destello para cubrir el paso 10.

## Continuidad

- Ejecutar los pasos 8e y 14 cuando llegue el hardware multicontrolador.
- Blindar el backend: la protección del borrador está en la interfaz, y la lectura fresca del backend sigue reemplazando el borrador si otro cliente la llama directamente. Hoy solo Sincronía la usa, pero conviene blindarla antes de que otra pantalla lo haga.
- Corregir las observaciones 01 a 03 y 05 a 07 en orden de impacto.

## Evidencia

Las salidas de consola, consultas y respuestas HTTP están transcritas en el reporte interno, con hora, identificador y valores observados. Las capturas de la ejecución las conserva la autora y no se anexan.
