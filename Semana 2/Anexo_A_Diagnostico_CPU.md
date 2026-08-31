# Anexo A - Reporte de diagnóstico: consumo de CPU en un controlador de campo

**Fecha del diagnóstico:** 24 de agosto de 2026
**Origen:** documento interno de diagnóstico, entregado al equipo.

> **Versión con datos omitidos.** Este anexo reproduce el contenido técnico del reporte interno omitiendo el identificador del nodo, direcciones de red, rutas de código y nombres de servicio, conforme al acuerdo de confidencialidad. Lo que se conserva es el problema, el razonamiento, los hallazgos y las verificaciones hechas.

---

## Alcance

Diagnóstico del consumo elevado de CPU reportado en un controlador de campo. Ventana de observación de 12:07 a 12:30, hora local, del 24 de agosto de 2026.

No se aplicó ningún cambio al equipo. No se reinició ningún servicio, no se modificó configuración, no se instaló nada persistente. La única escritura fue una herramienta de perfilado de solo lectura, que no altera el proceso observado.

## Evidencia recolectada

| Medición | Valor observado |
|---|---|
| Uptime del equipo | 10 días 20:27 |
| Load average (1/5/15 min) | 8.01 / 6.70 / 4.94 |
| CPU del componente de visión | 266.7% → 276.8% (de 400% disponibles) |
| CPU del componente de semáforos | 5.0% – 5.1% |
| Distribución de CPU por núcleo | usuario 65% / sistema 5% / libre 30% |
| Cambios de contexto | 34,900 – 51,400 /s |
| Temperatura | 60.9 °C |
| Throttling térmico | ninguno, histórico ni actual |
| Frecuencia del procesador | nominal completa |
| FPS del lazo principal de video | 49.4 – 49.7, estable |
| Memoria residente del proceso principal | ~429 MB |
| Hilos del proceso principal | 11 nombrados, más un grupo rotativo |

Se descartó causa térmica, sin throttling, con frecuencia nominal y temperatura normal, el consumo no es consecuencia de degradación de hardware. Es carga de software.

### Perfilado del proceso

Dos corridas independientes, la segunda con todas las pestañas de interfaz cerradas. Normalizado a núcleos de CPU consumidos:

| Función | Corrida 1 (24 s) | Corrida 2 (44 s) | Núcleos C1 | Núcleos C2 |
|---|---|---|---|---|
| Lectura de video | 16.99 s | 32.05 s | 0.708 | 0.728 |
| Evaluación de salud de la imagen | 14.75 s | 30.30 s | 0.615 | 0.689 |
| Envío de estado al panel de control | 8.01 s | 14.71 s | 0.334 | 0.334 |
| Overhead del grupo de trabajo | 4.33 s | 6.91 s | 0.180 | 0.157 |
| Apertura de fuentes de video | 2.01 s | 3.61 s | 0.084 | 0.082 |
| Codificación y serialización | 1.98 s | 3.25 s | 0.082 | 0.074 |

El uso del intérprete de Python durante el perfilado fue bajo, entre 7% y 9%, con actividad total entre 201% y 239%. Ese uso bajo indica que el trabajo se hace mayormente en librerías de procesamiento de imagen que liberan el intérprete, es cómputo real y no un ciclo vacío esperando.

La diferencia entre el 239% medido por el perfilador y el 276% medido a nivel de sistema se explica por tres procesos auxiliares de decodificación de video, hijos del proceso principal, que pertenecen al mismo grupo de control y por tanto suman al total del servicio, pero no son visibles para el perfilador.

### Estado de red del proceso

Una de las cuatro cámaras del equipo nunca completaba la conexión, quedaba en espera indefinida del protocolo de transmisión. Sin respuesta a ping, sin entrada en la tabla de direcciones locales, con error de ruta cada aproximadamente 5 segundos en el registro. El equipo no responde en la red.

La conexión hacia el panel de control central mostró acumulación sostenida de datos sin enviar durante la muestra, mientras que las conexiones hacia las cámaras alcanzables no mostraban acumulación.

## Hallazgo principal: lazo principal sin control de velocidad

El lazo principal de lectura de video corre sin ninguna pausa cuando hay regiones de detección configuradas, limitado solo por lo que el hardware puede procesar. La lógica que decide la pausa del lazo depende de un límite de cuadros por segundo que por defecto vale cero, así que el lazo cae siempre en la rama sin pausa cuando hay regiones activas.

Por cada vuelta y por cada cámara se ejecuta, en orden: lectura del cuadro más reciente, con una copia completa del cuadro, evaluación de salud de la imagen, y envío del estado al panel de control, que incluye codificación, serialización y armado del mensaje. A unas 50 vueltas por segundo. Los tres consumidores principales del perfil son exactamente estos tres pasos, en el mismo orden de costo.

Las rutas de reposo tampoco son gratuitas, sin regiones y sin clientes conectados, el tiempo de espera por defecto fija un piso de 50 cuadros por segundo incluso en estado ocioso.

## Hallazgos secundarios

**Una de las dos formas de leer video no tiene control de tasa.** Decodifica cada cuadro que entrega la cámara a su velocidad nativa, y a todos les aplica un cambio de tamaño y un cálculo de brillo promedio para detectar congelamiento de imagen. La otra forma de lectura sí tiene ajuste de velocidad. La asimetría es clara en cómo se construyen los lectores, uno recibe parámetros de demanda, el otro no recibe ninguno. Consecuencia, todo el mecanismo que ajusta el trabajo según demanda por fuente aplica solo a una de las dos formas de lectura.

**Evaluación de salud sin límite de tasa ni caché.** Ejecuta una comparación de patrones sobre la región de búsqueda por cada patrón, en cada llamada, sin límite de tasa. Los patrones se vuelven a calcular en cada invocación en vez de guardarse por resolución de imagen. Es por naturaleza un chequeo de estado lento, no requiere evaluarse a la tasa del lazo de video.

**Backpressure hacia el panel de control central.** Acumulación sostenida de datos sin enviar, creciendo durante la muestra y desapareciendo después la conexión. En dos horas de registro hubo dos eventos de cierre y reconexión, no un ciclo continuo. Se corrige aquí una afirmación previa de que la conexión estuviera reventando en bucle, no lo está. El envío de estado hacia el panel sí filtra correctamente si hay clientes conectados y limita su propia tasa, que aparezca alto en el perfil confirma que hay un consumidor activo, no que se estén desperdiciando envíos.

**Cámara inalcanzable.** El intento de apertura de esa fuente falla y el lazo aplica una espera fija de 2 segundos. El costo directo es bajo.

**Archivo de configuración residual.** Un archivo versionado en el repositorio del componente de visión referencia un segmento de red dado de baja. La configuración viva proviene de un archivo distinto fuera del repositorio, así que este es residuo y no afecta la operación, pero sigue presente.

**Anomalía en la métrica de CPU del registro.** El registro imprime un valor de CPU muy por debajo de lo que miden las herramientas del sistema operativo para el mismo proceso, del orden de veinte veces menor. No se identificó el mecanismo de la discrepancia. Se registra como anomalía abierta, no como defecto confirmado.

## Dirección del puente entre visión y semáforos

La comunicación es unidireccional, visión publica el estado de los detectores y el módulo de semáforos lo consume, mediante un envío periódico con intervalo mínimo fijo y sin reenviar cargas idénticas. El lazo de visión nunca se bloquea esperando al módulo de semáforos.

Implicación para cualquier mitigación, reducir la tasa del lazo de visión no bloquea el control de semáforos. Sí reduce la frecuencia con la que se actualiza el estado de detectores, pero cualquier tasa por encima de unos pocos cuadros por segundo mantiene margen sobre el intervalo mínimo del envío periódico.

## Hipótesis pendientes de confirmar

**H1 — Las regiones de detección están amarradas a la cámara inalcanzable.** El indicador de si hay regiones activas se calcula únicamente comprobando que exista una región configurada para alguna cámara declarada, sin verificar que esa fuente esté viva. Si las regiones configuradas apuntan a la cámara inalcanzable, el lazo se mantiene sin pausa aunque ninguna cámara con imagen real tenga regiones y por tanto no corra inferencia real.

Consistente con tres observaciones independientes, el conteo de detectores activos en la interfaz aparece en cero, no hay ningún rastro de inferencia en el perfil, y el lazo corre libre. Es la explicación más económica, pero no está verificada. La verificación es de solo lectura, comparar el archivo de configuración activo contra las regiones declaradas.

**H2 — El panel de control no drena al ritmo que visión produce.** La acumulación observada sugiere que el consumidor del lado del panel no lee tan rápido como visión escribe. No se inspeccionó ese lado, por lo que no se puede atribuir la causa. Relacionado con la restricción de un solo proceso trabajador del lado del panel, pero sin evidencia directa.

## Mitigación propuesta, no aplicada

Fijar el límite de cuadros por segundo del lazo principal corta la ejecución sin pausa sin tocar código, mediante una variable de entorno del servicio. Requiere recargar la configuración del gestor de servicios y reiniciar el componente. Reversible eliminando la configuración adicional.

Verificación posterior al reinicio obligatoria, un reinicio que retorna código de éxito no garantiza que el proceso sobrevivió. Hay que confirmar que el servicio sigue activo, que el proceso sigue vivo, y que el registro muestra actividad reciente con la tasa esperada.

**Riesgos.** El cambio reduce la tasa de actualización del estado de cámaras y detectores. Con detección actualmente en cero el impacto operativo aparente es nulo, pero eso mismo es síntoma de H1 sin resolver, no conviene tratar la mitigación como cierre. Tampoco corrige la falta de control de tasa en la forma de lectura sin ajuste, que seguirá decodificando a tasa nativa, solo reduce las veces que el lazo consume esos cuadros.

## Continuidad

| # | Pendiente | Prioridad |
|---|---|---|
| C1 | Verificar H1, cruce entre regiones configuradas y cámaras vivas | Alta, condiciona el resto |
| C2 | Restaurar o dar de baja formalmente la cámara inalcanzable | Alta, falla de campo |
| C3 | La forma de lectura sin ajuste de tasa no participa del sistema de demanda | Alta, causa estructural |
| C4 | Evaluación de salud sin límite de tasa, patrones sin caché por resolución | Media |
| C5 | Backpressure hacia el panel de control, requiere inspección del otro lado | Media |
| C6 | Anomalía de la métrica de CPU en el registro | Media |
| C7 | Archivo de configuración residual con segmento de red dado de baja | Baja, residuo |
| C8 | Tasa de creación de hilos de los lectores sin cuantificar | Baja |
| C9 | Decidir si el límite de fps es mitigación temporal o valor de operación | Abierta |

## Correcciones al diagnóstico durante la sesión

Se registran para que el reporte refleje el proceso real y no solo la conclusión:

1. Se atribuyó inicialmente el consumo a una lectura que mostraba un valor bajo para el componente de visión. Ese valor es el promedio sobre los más de diez días de vida del proceso, no el instantáneo. La medición válida es la del monitor de grupos de control en tiempo real.
2. Se planteó que cerrar la interfaz reduciría el consumo. La medición lo desmintió, el consumo subió ligeramente.
3. Se atribuyó una caída observada en la carga del sistema a que unos procesos auxiliares habían terminado. Incorrecto, seguían vivos. La caída no quedó explicada.
4. Se afirmó que el envío de estado armaba mensajes sin destinatario. Incorrecto, el filtro por clientes conectados está bien implementado y el consumidor real es el panel de control, no el navegador.
5. Se planteó que la conexión hacia el panel de control reventaba en bucle. El conteo del registro, dos eventos en dos horas, lo descarta.
