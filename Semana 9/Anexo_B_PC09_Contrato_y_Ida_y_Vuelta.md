Anexo B - Reporte de cierre: PC-09, contrato de configuración e ida y vuelta en el despliegue

Fecha del cierre: 15 de octubre de 2026
Origen: documento interno de cierre de prueba, entregado al responsable del proyecto.

> Versión con datos omitidos. Este anexo reproduce el contenido técnico del reporte interno omitiendo direcciones de red, identificadores de nodo y de controlador, versiones y huellas de código, huellas (hash) de configuración, nombres de archivos y rutas y comandos internos, conforme al acuerdo de confidencialidad. Lo que se conserva es el problema, el razonamiento, los hallazgos y las verificaciones hechas.

---

## Resumen y veredicto

**Aprobada** (15 de octubre de 2026). La corrida final contra el despliegue real usó la configuración de referencia oficial del CSI, sin variantes, y todos los pasos dieron correcto. La corrida anterior, con la configuración que traía otra zona horaria en el programa semanal, quedó rechazada solo por la huella del semanal; ya está corregido.

El sistema central y el CSI interpretan igual el contrato de configuración v1 en los tres casos válidos (fijo, diario y semanal). Los 19 rechazos, el 409 con identificador ajeno y la ausencia de efectos laterales pasan en la corrida final. Los archivos están aplicados, el sistema central y el CSI desplegados y la corrida final hecha. Quedan solo pasos que no son de código.

## Versiones desplegadas

La corrida final se hizo contra versiones verificadas en el despliegue de ambos lados: el sistema central actualizado y corriendo, y el CSI con su release nuevo activo. Se registraron las versiones de ambos y la huella del archivo parcheado del compilador, que coincide con la del parche una vez normalizados los saltos de línea de Windows a los de Linux. Lo desplegado es exactamente el parche, sin cambios extra. Los valores concretos se omiten en esta versión.

## Resultados

Se corrieron tres veces los mismos pasos, con el mismo nodo y el mismo controlador. La primera usó la configuración oficial anterior y quedó rechazada; la segunda cambió solo la zona del programa semanal a la del nodo y quedó aprobada; la tercera, final, usa la configuración oficial corregida, sin variantes, y quedó aprobada.

| Caso válido | Configuración oficial anterior | Con la zona del nodo (segunda y final) |
| --- | --- | --- |
| Fijo | Igual en ambos lados | Igual en ambos lados |
| Diario | Igual en ambos lados | Igual en ambos lados |
| Semanal | Distinto: cada lado calcula una huella diferente | Igual en ambos lados |

En los tres casos válidos, la validación del CSI y la inspección y construcción del sistema central responden 200. El resto de pasos pasa igual en las tres corridas:

- Consulta de identidad del CSI: contrato de configuración, versión 1, identidad habilitada e identificador de controlador detectado.
- Lectura de la configuración dos veces con la misma huella, y registro legible.
- 19 rechazos con el código y la categoría esperados (estructura, versión, grupo, duración peatonal, programa, huecos, solapes, semana incompleta, zona inválida y las reglas de penalización y escenarios de la especificación funcional), todos sin configuración pendiente ni cambio de la activa.
- Identificador de controlador ajeno: HTTP 409 por identificador no coincidente, sin efectos laterales.

## Defectos encontrados y corrección

Las corridas anteriores (contra la versión previa del CSI y la primera contra la nueva) fallaron por tres causas distintas. Las tres se corrigieron en código y quedaron verificadas en el despliegue; la tercera, además, es una decisión de criterio: el nodo manda.

| Defecto | Síntoma | Corrección | Estado |
| --- | --- | --- | --- |
| CSI: el compilador cobraba el rojo posterior a fragmentos que no cierran ningún grupo | Programas diario y semanal rechazados con 422 por presupuesto de transición insuficiente | El rojo posterior se cobra solo si un grupo realmente pasa a rojo en ese límite | Corregido y desplegado |
| Sistema central: la inspección descartaba las etiquetas de los grupos de fases | El programa fijo con huella distinta: el v1 reconstruido no tenía etiqueta | La inspección devuelve las etiquetas; el constructor, el contexto de construcción y el frontend las llevan; si un cliente no las manda, se recuperan del último v1 guardado del controlador | Corregido y desplegado, incluida la recuperación desde lo guardado |
| Zona horaria: el sistema central construye con la del nodo y la configuración oficial traía otra | Programa semanal con huella distinta: la zona del nodo contra la de la Ciudad de México | El nodo manda: la configuración oficial usa la zona del nodo en sus tres casos (CSI y sistema central, idénticos) y la inspección avisa cuando un v1 trae otra zona | Corregido y desplegado; la corrida final con la configuración oficial da aprobada |

El parche del CSI no relaja las garantías: un fragmento por debajo del verde mínimo vehicular, o que sí cierra un grupo, sigue rechazado. Lo cubren cuatro pruebas nuevas, y se quitó de dos archivos de pruebas el caso que esperaba el rechazo de los programas diario y semanal.

En el sistema central, una prueba nueva recorre la inspección y la construcción con los tres programas y compara la huella. Falla sin el cambio y pasa con él.

## Criterios de cierre

Los criterios 1, 2 y 3 se cumplen; del 4 falta solo revisar que los archivos de evidencia no tengan secretos antes del commit. El tercero cambia de redacción porque, por diseño, el identificador administrativo del controlador y el identificador de Luces no son iguales.

| Criterio | Estado | Evidencia |
| --- | --- | --- |
| Los casos válidos convergen al mismo v1 ejecutable | Se cumple con la configuración oficial corregida y sin variantes; con la anterior fallaba el semanal | Huellas iguales en fijo, diario y semanal; la validación del CSI con resultado ejecutable en los tres |
| Los inválidos fallan antes de persistirse | Se cumple | 19 rechazos 422 con el código esperado; registro idéntico antes y después |
| El identificador administrativo está ligado al identificador de Luces, y la huella canónica coincide | Se cumple | La construcción con el identificador administrativo acepta el identificador de Luces que reporta el CSI; uno ajeno da 409 por identificador no coincidente |
| Evidencia con ambas versiones y sin secretos | Se cumple en el reporte de resultado; falta revisar los archivos de evidencia antes del commit | El reporte de resultado trae las dos versiones; el token solo viaja en una variable de entorno |

La redacción original del tercer criterio decía que el sistema central y el CSI "reportan el mismo identificador". Debe decir que el identificador administrativo (propiedad del sistema central) está correctamente ligado al identificador operativo de Luces, sin que sean iguales.

**Fuera de la PC-09 (no la bloquean)**

- **Inventario de cámaras (sistema central).** Una prueba espera que una consulta devuelva también las cámaras de otro nodo del mismo controlador, y el servicio ya no lo hace. Es un tema de cámaras: hay que decidir cuál de los dos es el comportamiento correcto.
- **Estado de salud del CSI.** Se vio un fallo de escritura de la salida de luces con el controlador de salida desconectado. Es un problema aparte, no investigado, y no tiene que ver con el contrato.

## Evidencia y cómo repetir

La evidencia queda en la carpeta de evidencia del repositorio del sistema central, con un reporte de resultado por corrida:

- Corrida final con la configuración oficial corregida, sin variantes: aprobada. Reemplaza a la corrida rechazada anterior (solo por la huella del semanal).
- Variante previa con el programa semanal en la zona del nodo: aprobada; su nota explica que ya no hace falta repetirla.

Para repetirla, se ejecuta el script de validación desde la raíz del sistema central, con el token de sesión en una variable de entorno y apuntando a la carpeta de la configuración oficial del repositorio del CSI, que ya trae la zona del nodo y se usa tal cual, sin copias ni variantes. Las direcciones, el nodo, el controlador y las versiones que recibe el script se omiten en esta versión.
