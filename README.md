# Automatización de la Generación de Resúmenes Judiciales mediante Modelos de Lenguaje Natural

### Integrantes:
- Fuentes, Tiffany.
- Martinelli, Sofía.

Ante la necesidad de los empleados del Poder Judicial de la Provincia de Córdoba de redactar manualmente resúmenes de boletines judiciales, y como parte de un esfuerzo continuo por optimizar estos procesos, se ha implementado el uso de modelos de lenguaje natural para automatizar la generación de dichos resúmenes. La técnica seleccionada fue el prompt engineering, que permite ajustar las salidas generadas por los modelos a formatos específicos sin requerir un proceso de fine-tuning, lo cual reduce significativamente los costos computacionales y facilita su implementación en entornos prácticos. 

Durante el proyecto se llevaron a cabo pruebas y evaluaciones comparativas entre modelos multilingües y modelos especializados en español para identificar el más adecuado para generar resúmenes en el ámbito judicial, y si los modelos entrenados específicamente en español logran mayor precisión y coherencia en este dominio.

## Objetivos preliminares
En la etapa inicial del proyecto, se plantearon los siguientes objetivos:

1. Desarrollar un prototipo que permita la generación automática de resúmenes de textos judiciales.
2. Comparar el rendimiento de un modelo multilingüe y uno especializado en español para la generación de resúmenes.
3. Analizar la capacidad del prompt engineering para generar resúmenes adecuados sin la necesidad de fine-tuning.

A medida que avanzamos, algunas de las hipótesis preliminares fueron revisadas y descartadas en función de los resultados obtenidos. También surgió un nuevo objetivo: optimizar el uso de LLMs (en este caso ChatGPT) para condensar los documentos con instrucciones para los sumarios y poder generar un prompt conciso para la generación del mismo.

## Resultados de los experimentos
### Flan T5, BERT, roBERTa
Para los experimentos realizados con estos tres modelos, implementamos una estrategia común: segmentamos el documento judicial en fragmentos más pequeños y, tras procesar cada uno de ellos, generamos un resumen intermedio con el fin de superar las limitaciones impuestas por el contexto reducido de estos modelos. En cada caso, solicitamos al modelo que produjera un resumen final del documento judicial en su totalidad.

Sin embargo, los resultados obtenidos no fueron satisfactorios. Los tres modelos presentaron incoherencias significativas en sus respuestas, generando información irrelevante o incluso inexistente que no se encontraba en el texto original. Por ejemplo, en el caso de BERT y roBERTa el modelo presentaba alucinaciones e incluía información que no provenía del documento original, mientras que en Flan T5 a pesar de no presentar información incorrecta, no generaba resúmenes, si no que directamente colocaba fragmentos del texto original.

En base a estos resultados y dado que roBERTa es un modelo especializado en español, descartamos la hipótesis de que los resúmenes generados por este modelo serían más precisos que los del modelo multilingüe en lo que respecta al lenguaje legal.

A continuación presentamos salidas de algunos de los modelos mencionados: 

BERT:
> Generated Summary: “The case was brought by Patricia Alejandra Farías, a.k.a. "Patricia the Pregnant”.

roBERTa:
> Resumen final: “Te ofrecemos una selección de artículos de EL PAÍS de este 9 de noviembre”.

> Resumen final: “(...) sentencia del Superior de Justicia acerca de la condena a la actora lesbiana lesbiana lesbiana”. 

Ante estos problemas de generación de información inexacta, determinamos que estos modelos no eran adecuados para nuestros objetivos y los excluimos de futuros experimentos. 

Como consecuencia de los resultados obtenidos, decidimos descartar otra de las hipótesis formuladas para estos modelos en particular: la posibilidad de mejorar su rendimiento mediante la aplicación de fine-tuning.

### Llama 3.1- 8b
Después de experimentar con otros modelos, decidimos centrar nuestros esfuerzos en el modelo Llama 3.1-8b.

El contexto del modelo es lo suficientemente amplio como para permitir el procesamiento completo del documento judicial junto con el prompt, sin necesidad de iterar o agregar resúmenes intermedios, a diferencia de los modelos utilizados anteriormente.

Para evaluar la viabilidad del modelo, seleccionamos un documento judicial específico (2023-04-25 S17 - penal) y solicitamos al modelo que generara un resumen sin proporcionar instrucciones adicionales ni información sobre el formato esperado.

El resumen generado resultó ser extenso e informal, pero contenía exclusivamente información presente en el documento judicial y capturaba adecuadamente los puntos clave del fallo. Esto demostró que el modelo era capaz de producir un resumen fiel al contenido original, aunque presenta oportunidades de mejora en términos de concisión y formalidad.

Basándonos en este primer experimento, decidimos estructurar el proceso de generación de resúmenes en tres partes clave del resumen judicial:

- Datos de la causa.
- Síntesis de la causa.
- Sumario.

Para cada una de estas partes, generamos un prompt separado. Esta segmentación nos permitió incluir ejemplos de respuestas correctas e incorrectas, especificar el formato de salida esperado y adaptar el enfoque según cada tarea. Aunque el modelo tiene un contexto suficientemente amplio, decidimos evitar posibles errores por truncamiento dividiendo el proceso en estas secciones.

Las primeras evaluaciones de los resúmenes generados se llevaron a cabo en varias etapas, combinando métodos cualitativos y cuantitativos. 

En primer lugar, realizamos una evaluación manual centrada en aspectos clave como la veracidad de los datos, la adecuación del vocabulario, así como la fluidez y la consistencia semántica del contenido. Estas consideraciones nos permiten asegurar que la salida no solo sea precisa, sino también comprensible y de fácil lectura.

Además, implementamos métricas automáticas de evaluación, como la métrica ROUGE para la sección de "Datos de la Causa", dado que la comparación directa entre el texto original y el resumen es más viable en esta sección.

#### Datos de la causa
Para extraer los datos clave de la causa judicial, implementamos el siguiente prompt:

```
Extrae solamente los siguientes datos de la causa del texto judicial que sigue. Debes obtener:
    1. Fecha
    2. Sede
    3. Dependencia
    4. Autos
    5. Resolución
    6. Jueces

    Formato del resultado (obligatorio):
    Fecha: [Fecha]
    Sede: [Sede]
    Dependencia: [Dependencia]
    Autos: [Autos]
    Resolución: [Resolución]
    Jueces: [Jueces]

    Aquí está el texto judicial:
```

Realizamos varias pruebas modificando el orden en que se presentaba el texto judicial dentro del prompt (al inicio o al final), y también resaltando la importancia de términos como "solamente" para enfocar la extracción en los datos solicitados.

El prompt que produjo los mejores resultados fue:

```
Extrae **solamente** los siguientes datos de la causa del texto judicial que sigue, en el output solo devuelve la respuesta final. Debes obtener:
    1. Sede
    2. Dependencia
    3. Autos
    4. Resolución
    5. Fecha
    6. Jueces

    Formato del resultado (obligatorio):
    Sede: [Sede]
    Dependencia: [Dependencia]
    Autos: [Autos]
    Resolución: [Resolución]
    Fecha: [Fecha]
    Jueces: [Jueces]

    Aquí está el texto judicial:
    {text}
```

Resultado generado:

```
Sede: Córdoba
Dependencia: Cámara Quinta en lo Criminal y Correccional, Sala “C”
Autos: 2018121
Resolución: Nº 17
Fecha: 25/04/2023
Jueces: Dra. María Susana Beatriz Blanc Gerzicich de Scapellato y Dr. Oscar Quintana
```

Resumen original:

```
Sede: Ciudad de Córdoba
Dependencia: Cámara en lo Criminal y Correccional de Quinta Nominación, Secretaría n.° 9, Sala Unipersonal
Autos: “Gruttadauria, César Daniel y otro p. s. a. homicidio culposo”, expediente n.° 2018121
Resolución: Sentencia n.° 17
Fecha: 25/4/2023
Jueza: María Susana Beatriz Blanc Gerzicich
```

Métricas ROUGE:
- ROUGE-1: Precisión: 0.8158, Recall: 0.5741, F1-score: 0.6739
- ROUGE-2: Precisión: 0.4865, Recall: 0.3396, F1-score: 0.4000
- ROxUGE-L: Precisión: 0.7105, Recall: 0.5000, F1-score: 0.5870

El resultado fue bastante satisfactorio en términos de precisión y coherencia. Sin embargo, aún falta probar si la inclusión de ejemplos claros, tanto del documento judicial como de los datos de la causa esperados en el resumen, podría mejorar aún más el desempeño del modelo.

#### Síntesis

En esta sección explicamos los tres enfoques utilizados para generar la síntesis del caso, implementados a través de prompts distintos:
- Prompt 1: Fue un prompt genérico, diseñado como base para los otros dos, y que proveía un resumen sin indicaciones específicas.
- Prompt 2: Incorporó un ejemplo de una síntesis correcta, en este caso, la síntesis real del fallo en cuestión, con el objetivo de guiar al modelo hacia una respuesta más precisa.
- Prompt 3: Además del ejemplo correcto, incluía un ejemplo de una síntesis inadecuada, destacando qué errores se debían evitar en la respuesta.

Cada uno de estos enfoques permitió observar diferencias notables en la precisión y calidad de los resúmenes generados. En particular, se aprecia una mejora notable en los resultados obtenidos con los prompts que incluyen ejemplos, en contraste con el que no utiliza.

#### Sumario
La generación del sumario ha sido un desafío clave debido a la complejidad inherente a este tipo de texto. Aunque existen instructivos detallados para redactar los sumarios, son documentos muy extensos (creados en 2012), y el propio equipo del Poder Judicial nos indicó que muchos de estos criterios cambian con el tiempo. Esto complica encontrar un enfoque adecuado para comprimir estas instrucciones en un formato manejable para el prompt.

Hemos utilizado ChatGPT para procesar estos documentos, extrayendo las instrucciones más relevantes para generar el prompt, pero nos encontramos con múltiples dificultades. La decisión sobre qué información incluir o excluir en el prompt no siempre es clara, ya que el contenido de los instructivos es denso y cambiante. Hay muchas formas de abordar este proceso, y estamos explorando cuál es la más eficiente.

El enfoque utilizado es el mismo que en la síntesis, usando tres prompts distintos, uno sólo con las instrucciones generadas con ayuda de ChatGPT, un segundo prompt incluyendo un ejemplo positivo, y un tercer prompt incluyendo un ejemplo positivo y uno negativo.

## Evaluación
El equipo del Poder Judicial realizará una evaluación exhaustiva de los resultados generados por los diferentes prompts. Evaluarán aspectos como coherencia, concisión, exactitud factual y adecuación del lenguaje, utilizando una escala del 1 al 7. Además, proporcionarán comentarios cualitativos sobre la estructura del resumen, indicaciones sobre cómo mejorar el formato (por ejemplo, evitar abreviaciones en nombres), y sugerencias para futuras optimizaciones.

Este feedback nos permitiría no solo elegir el mejor enfoque de generación, sino también refinar los prompts y mejorar los resultados basándonos en sus observaciones. Finalmente, estos comentarios serían la base para iterar sobre los prompts y seguir optimizando el proceso de síntesis y sumario.

## Dificultades
Entre las principales dificultades que enfrentamos se incluyen las siguientes:
1. Limitación de recursos: Inicialmente, no pudimos correr el modelo con GPU, lo que limitó el número de experimentos que pudimos realizar. Sin embargo, desde hace tres días pudimos acceder a una GPU, lo que ha acelerado significativamente los experimentos.
2. Evaluación de los resúmenes: Aunque las métricas automáticas como ROUGE son adecuadas para los datos de la causa, no resultan tan precisas para evaluar síntesis o sumarios. Esto se debe a que el modelo puede generar buenos resultados utilizando palabras diferentes a las del resumen original, afectando negativamente la métrica de ROUGE. Aquí es donde el feedback del equipo experto es esencial para identificar los mejores resúmenes.
3. Generación de prompts para el sumario: Crear un buen prompt para los sumarios ha sido complicado debido a la antigüedad de los instructivos y su naturaleza cambiante. Encontrar una fórmula que resuma estas instrucciones de manera eficiente es un reto que seguimos enfrentando.

## Planificación a lo largo del proyecto
A pesar de que surgieron algunas dificultades a lo largo del desarrollo del proyecto, la planificación inicial fue bastante precisa. 

En un principio, habíamos proyectado alcanzar avances significativos y obtener resultados preliminares para la última semana de octubre, incluyendo un análisis comparativo entre modelos y una autoevaluación de los resúmenes generados. Si bien logramos cumplir con los plazos estimados, uno de los aspectos que no cumplió con nuestras expectativas fue la cantidad de pruebas realizadas durante el período de experimentación. Esto se debió a las dificultades técnicas previamente mencionadas, particularmente la imposibilidad de ejecutar los modelos con GPU. En consecuencia, aunque respetamos los tiempos planificados, no fue posible realizar experimentos con la totalidad o una gran cantidad de archivos. 

Hubo un cambio importante respecto a la planificación que hicimos al inicio del cuatrimestre; decidimos enfocarnos en mejorar los prompts para la generación de resúmenes. Como equipo, optamos por descartar la posibilidad de realizar fine-tuning, considerando particularmente las limitaciones de tiempo y el alcance del proyecto. Además, éramos conscientes de que esta técnica requería muchos recursos, lo cual también influyó en nuestra decisión al momento de cerrar el trabajo.

Actualmente nos encontramos esperando las devoluciones de los expertos de dominio. Sin embargo, en la siguiente sección de trabajo futuro vamos a plantear una alternativa para poder corregir los resúmenes generados.

Otro aspecto relevante de la planificación fue el feedback recibido al entregar el informe preliminar, el cual incluyó varias sugerencias útiles por parte de nuestros compañeros. Dado que aún nos encontrábamos en una etapa de exploración, muchos de sus comentarios nos permitieron identificar aspectos que no habíamos considerado en profundidad, como los métodos más adecuados para generar los prompts, las estrategias para evaluar los modelos y las condiciones bajo las cuales sería necesario realizar fine-tuning. Este intercambio fue particularmente valioso, ya que nos ayudó a replantear y enriquecer nuestro enfoque de trabajo.

## Trabajo futuro

Considerando un equipo de cinco personas trabajando durante un año a tiempo completo, las siguientes líneas de investigación y desarrollo podrían implementarse para continuar avanzando en este proyecto:

### Implementación de un pipeline híbrido con múltiples LLMs
El diseño de un pipeline híbrido con múltiples modelos de lenguaje busca mejorar la calidad y especificidad de los resúmenes judiciales generados. Este enfoque combina las capacidades de modelos como Claude o Gemini para optimizar prompts con la generación final realizada por un modelo adaptado, como LLaMA.

El objetivo principal es crear un flujo de trabajo eficiente que:
- Genere prompts personalizados para resúmenes judiciales basados en entradas específicas.
- Utilice estos prompts para guiar a un modelo como LLaMA en la generación del resumen.
- Permita iterar sobre los prompts y ajustar el proceso según el feedback del usuario.

#### Flujo del pipeline
1. Input inicial: resumen deseado.

El pipeline comienza con el resumen objetivo (deseado) que se quiere generar. Este resumen se proporciona manualmente o se selecciona de un conjunto de ejemplos predefinidos.

2. Generación del prompt.

Se utiliza un modelo LLM comercial, como Claude o Gemini, para generar un prompt que guíe la generación del resumen. Este prompt debe:
- Describir claramente las características esperadas del resumen.
- Proporcionar instrucciones específicas para estructurar el texto.

Ejemplo:
- Input: Texto del fallo judicial y resumen deseado.
- Output del modelo LLM: "Por favor, genera un resumen que incluya los puntos principales del fallo judicial, asegurándote de mencionar el nombre completo de los juzgados y evitar ambigüedades en los eventos narrados."

3. Generación del resumen.

El prompt generado, junto con el texto del fallo judicial, se pasa al modelo LLaMA, que produce el resumen.

4. Incorporación de feedback.

Si el usuario elige generar un nuevo prompt, se le da la opción de proporcionar feedback.

Ejemplo de flujo interactivo:
1. Output: "Selecciona 1 si quieres generar un nuevo prompt o 2 si quieres continuar con este resumen."
    - Usuario: 1
2. Output: "Selecciona 1 si quieres agregar feedback o 2 si no."
    - Usuario: 1
3. Output: "Ingresa el feedback."
    - Usuario: "Asegúrate de incluir en el resumen el nombre completo de los juzgados."

El feedback se utiliza para ajustar el prompt generado por el modelo comercial y guiar una nueva iteración.

5. Validación del resumen

El resumen final es revisado por el usuario para asegurar que cumple con las expectativas. Si es satisfactorio, se guarda; de lo contrario, el proceso puede repetirse.

#### Ventajas del pipeline híbrido:
- Especialización de tareas: Aprovecha las fortalezas de cada modelo: los modelos comerciales generan prompts optimizados, mientras que LLaMA realiza la tarea específica de generación de texto.
- Iteración guiada: El feedback del usuario permite mejorar la calidad de los prompts y afinar el proceso iterativo.
- Flexibilidad: El pipeline puede adaptarse para incluir nuevos modelos o ajustar las etapas según las necesidades del proyecto.

#### Retos del pipeline:
- Integración de modelos: Diseñar una arquitectura que permita la comunicación eficiente entre modelos y mantenga la consistencia de los datos.
- Calidad del feedback: La efectividad del pipeline depende de la claridad y precisión del feedback proporcionado por el usuario.
- Costos computacionales: Utilizar modelos comerciales y ejecutar iteraciones puede ser costoso, especialmente si el volumen de resúmenes es alto.

### Desarrollo de un evaluador automático para resúmenes
Una opción paralela o alternativa a la evaluación manual por parte del poder judicial sería programar un evaluador automático. Este evaluador, basado en un modelo de lenguaje avanzado (como Claude), analizará los resúmenes generados y asignará puntajes en función de criterios previamente definidos.

El sistema automatizado evaluará aspectos clave como coherencia, factibilidad, precisión y relevancia de los resúmenes generados, comparándolos con el resumen original. De esta manera, se busca mantener un estándar de calidad en la generación de resúmenes y garantizar su utilidad mientras se solventan los problemas de la evaluación manual.

El evaluador recibirá como entrada:
- Resumen original: Generado a partir del documento fuente.
- Resumen generado por el modelo: El texto a ser evaluado.

En algunos casos, será necesario utilizar también el documento fuente (fallo original) para validar ciertos aspectos, como la factibilidad de las afirmaciones.

#### Definición de los aspectos evaluados:
- Coherencia: Verifica si el resumen generado contiene contradicciones internas. Por ejemplo, una oración que afirme que un sujeto realizó una acción y otra que diga que no lo hizo.
- Precisión: Evalúa si el resumen generado incluye información que no aparece en el resumen original, como personas, lugares o eventos inexistentes. La comparación es estrictamente entre ambos resúmenes.
- Factibilidad: Examina si las afirmaciones del resumen generado son plausibles y realistas en el contexto del documento fuente (fallo original). Por ejemplo, si las acciones atribuidas a un sujeto son compatibles con los hechos del caso.
- Relevancia: Mide si el resumen generado refleja adecuadamente los puntos clave del resumen original, sin omitir información esencial ni agregar detalles irrelevantes.

Es fundamental establecer definiciones claras y objetivas para cada uno de estos aspectos de evaluación. Esto permitirá que el modelo evite ambigüedades al asignar los puntajes y que la evaluación sea consistente. Por ejemplo:
- En coherencia, especificar qué constituye una contradicción entre las oraciones del resumen.
- En precisión, definir cómo identificar elementos inventados que no aparecen en el resumen original.
- En factibilidad, determinar qué se considera plausible dentro del contexto del fallo judicial.
- En relevancia, señalar los puntos clave que deben ser reflejados obligatoriamente.

Adicionalmente, este sistema podrá complementarse con los comentarios cualitativos que el equipo del Poder Judicial proporcione durante sus evaluaciones manuales, ayudando a iterar sobre los prompts y mejorar el proceso de generación de resúmenes de manera continua.

### Uso de LLMs Comerciales y APIs de Pago
Explorar el uso de modelos de lenguaje comerciales, como Claude o Gemini, que ofrecen soluciones optimizadas a través de sus APIs, presenta varias ventajas:
- Reducción de la carga computacional: Al utilizar APIs, no es necesario disponer de hardware avanzado, ya que el procesamiento se realiza en servidores externos.
- Calidad optimizada: Estos modelos suelen estar pre entrenados en vastos conjuntos de datos y cuentan con optimizaciones específicas para tareas de generación de texto, lo que puede traducirse en resúmenes más precisos y bien estructurados.
- Escalabilidad: Las soluciones basadas en APIs permiten procesar múltiples resúmenes simultáneamente sin requerir infraestructura adicional.

#### Consideraciones para el uso de APIs
- Costos: Es importante evaluar los costos asociados al uso recurrente de estas APIs, especialmente para proyectos a gran escala.
- Privacidad: Asegurarse de que los datos enviados cumplan con las normativas de protección de datos, ya que serán procesados por servidores externos.
- Personalización: Aunque los modelos comerciales son poderosos, su capacidad de adaptación a dominios específicos (como el legal) puede ser limitada sin acceso directo al entrenamiento del modelo.
- Dependencia de infraestructura externa: Al depender de la infraestructura de la empresa que provee la API, cualquier interrupción en su servicio podría afectar gravemente el funcionamiento del proyecto.

### Fine-tuning de LLaMA para resúmenes judiciales
Realizar fine-tuning sobre el modelo LLaMA utilizando un conjunto de datos especializado en textos judiciales permitiría:
- Adaptación al dominio legal: El modelo podría aprender estructuras lingüísticas y terminología específica del ámbito judicial, mejorando su capacidad para generar resúmenes relevantes y precisos.
- Control sobre el modelo: A diferencia de las APIs comerciales, el fine-tuning otorga un mayor control sobre el comportamiento del modelo y sus outputs.
- Costos a largo plazo: Aunque el proceso de fine-tuning puede ser intensivo inicialmente, a largo plazo puede ser más económico que el uso continuo de APIs de pago.

#### Retos del fine-tuning
- Requisitos computacionales: El proceso de fine-tuning requiere hardware potente, como GPUs, y puede ser costoso.
- Curación de datos: Garantizar que los datos utilizados sean representativos del dominio judicial es clave para evitar sesgos.
- Mantenimiento: Un modelo fine-tuneado puede necesitar actualizaciones periódicas para adaptarse a cambios en el lenguaje o las normativas legales.

Estas propuestas, aunque relativamente simples de implementar a nivel de código, requieren un acceso considerable a recursos de hardware para llevar a cabo entrenamientos y pruebas a gran escala. Esto subraya la importancia de contar con infraestructura adecuada para seguir explorando las capacidades de los modelos de lenguaje en el ámbito judicial.

## Conclusión
En este trabajo, se abordó la automatización de la generación de resúmenes judiciales mediante modelos de lenguaje natural, respondiendo a la necesidad de optimizar los procesos manuales llevados a cabo en el Poder Judicial de Córdoba. 

Los objetivos iniciales incluían el desarrollo de un prototipo funcional, la comparación entre modelos multilingües y especializados en español, y la evaluación de la técnica de prompt engineering como alternativa eficiente al fine-tuning. 

A lo largo del proyecto, si bien los modelos iniciales (Flan-T5, BERT y roBERTa) no lograron cumplir con las expectativas en términos de coherencia y precisión, el modelo LLaMA 3.1-8b demostró ser una herramienta prometedora. Este modelo, con un contexto más amplio, permitió segmentar y estructurar los resúmenes en tres partes clave: datos de la causa, síntesis y sumario. 

Si bien los resultados alcanzados son satisfactorios, es importante destacar que los resúmenes generados no reemplazarán el trabajo humano debido a la complejidad inherente al lenguaje jurídico y a las variaciones en los criterios de redacción. No obstante, como herramienta de apoyo, el sistema tiene un gran potencial para facilitar y agilizar la tarea de los profesionales judiciales, especialmente si se incorporan las correcciones y evaluaciones de expertos. 

En conclusión, aunque existen desafíos técnicos y conceptuales, la automatización mediante modelos de lenguaje natural representa un avance significativo en la optimización de procesos judiciales, posicionándose como una herramienta valiosa para el futuro.

## Referencias
- Kojima, T., Gu, S. S., Reid, M., Matsuo, Y., & Iwasawa, Y. (2022). Large language models are zero-shot reasoners. Proceedings of the 36th International Conference on Neural Information Processing Systems (NeurIPS 2022).
- Liu, Y., & Lapata, M. (2019). Text Summarization with Pretrained Encoders. Proceedings of the 2019 Conference on Empirical Methods in Natural Language Processing and the 9th International Joint Conference on Natural Language Processing (EMNLP-IJCNLP), 3730-3740.
- Reynolds, L., & McDonell, K. (2021). Prompt Programming for Large Language Models: Beyond the Few-Shot Paradigm. Extended Abstracts of the 2021 CHI Conference on Human Factors in Computing Systems.