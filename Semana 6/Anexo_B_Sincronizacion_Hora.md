Anexo B - Reporte de verificación: sincronización de hora entre el backend y los controladores de campo

Fecha del cierre: 23 de septiembre de 2026
Origen: documento interno de verificación, entregado al responsable del proyecto.

> Versión con datos omitidos. Este anexo reproduce el contenido técnico del reporte interno omitiendo direcciones de red, identificadores de equipo y rutas de configuración, conforme al acuerdo de confidencialidad. Lo que se conserva es el problema, el razonamiento, los hallazgos y las verificaciones hechas.

---

## Objetivo

Confirmar cómo se mantienen sincronizados los relojes entre el backend central y los controladores de campo, y validar que el mecanismo funciona correctamente antes de depender de él en operación real.

## Corrección del planteamiento inicial

El primer planteamiento fue usar el protocolo estándar de sincronización de red (NTP) en ambos lados. Revisando el código se descartó, el controlador de campo desactiva explícitamente ese protocolo al arrancar, y su propia documentación de instalación exige deshabilitarlo. La hora se obtiene por un mecanismo propio sobre HTTP, no por NTP. Activar el protocolo estándar en el controlador habría hecho competir dos mecanismos por ajustar el mismo reloj.

## Cómo funciona el mecanismo

El backend expone un servicio de referencia de tiempo, que arranca junto con el resto del backend y no bloquea su arranque si el puerto correspondiente ya está en uso (solo registra una advertencia). Cada controlador de campo consulta esa referencia cada 30 segundos, con varios intentos cortos por consulta, calcula el desfase usando el tiempo de ida y vuelta más bajo de esos intentos, y ajusta el reloj del sistema. La hora ajustada se espeja además a un reloj de respaldo con batería propia, que sirve como fuente al arrancar si la referencia del backend no responde todavía.

La exactitud de todo el mecanismo depende de que el reloj del propio backend esté bien sincronizado hacia una fuente externa confiable; el mecanismo hacia los controladores solo propaga esa hora, no la corrige de forma independiente.

## Evidencia recolectada

- El servicio de referencia de tiempo del backend respondió correctamente en su verificación de salud.
- El reloj del backend estaba sincronizado hacia una fuente externa confiable, con una diferencia de menos de un milisegundo.
- Se consultó el estado de dos controladores de laboratorio, ambos reportaron estar sincronizados contra la referencia del backend, con la fuente correcta y sin errores registrados. Uno arrancó tomando la hora de la referencia directamente; el otro arrancó desde su respaldo de batería porque la referencia del backend todavía no estaba accesible en ese momento, y se enganchó correctamente en el siguiente ciclo, es el comportamiento de respaldo esperado, no una falla.
- El puerto usado por el servicio de referencia de tiempo está cubierto por las reglas de firewall ya vigentes en el backend.

## Observaciones y hallazgos secundarios

- **Desfase residual sin corregir.** El reloj del sistema de cada controlador solo se ajusta al arrancar o cuando el desfase supera un umbral configurado; por debajo de ese umbral, el desfase se registra pero no se corrige de inmediato. Los controladores de laboratorio quedaron con un desfase de decenas de milisegundos respecto al backend, dentro de lo esperado por diseño, hasta que se acumule lo suficiente para que se aplique un ajuste.
- **Inconsistencia entre el reloj de respaldo y el del sistema.** En cada ciclo de sincronización se actualiza el reloj de respaldo con la hora ya corregida, aunque el reloj del sistema no se haya movido por estar debajo del umbral; el respaldo puede quedar momentáneamente adelantado respecto al reloj del sistema del mismo equipo.
- **Firewall más abierto de lo necesario.** Las reglas actuales del backend abren un rango amplio de puertos en vez de solo el que usa este mecanismo; cubre lo necesario, pero también expone cualquier otro servicio que llegara a escuchar en ese rango. Es un hallazgo de seguridad, no afecta la sincronización.
- **Margen de espera holgado para la red local.** El tiempo de espera configurado para cada consulta es generoso comparado con el tiempo de ida y vuelta real observado en la red local; conviene medir el tiempo real en enlaces remotos antes de desplegar ahí el mismo mecanismo, porque puede ser mayor.

## Criterios verificados

| Criterio | Estado |
|---|---|
| Servicio de referencia de tiempo activo en el backend | Cumple |
| Referencia alcanzable desde los controladores | Cumple |
| Controladores sincronizando contra el backend | Cumple |
| Respaldo por reloj de batería operativo | Cumple |
| Sin conflicto con el protocolo estándar de red | Cumple, por diseño |
| Reloj del backend sincronizado hacia una fuente externa | Cumple |
| Puerto del mecanismo permitido en el firewall | Cumple |
| Arranque en frío desde la referencia | Validado en un equipo; el otro es de campo y no se reinició por restricción operativa, pero corre el mismo mecanismo y su arranque desde el respaldo validó ese camino |

## Continuidad

- No quedan verificaciones abiertas en esta configuración de laboratorio.
- Medir el tiempo de ida y vuelta real antes de desplegar el mecanismo en enlaces remotos.
- Revisar si conviene reducir el rango de puertos abiertos en el firewall a solo lo necesario.
- Mejora de código identificada: escribir al reloj de respaldo solo cuando se aplica un ajuste real, para eliminar la divergencia entre el respaldo y el reloj del sistema.
