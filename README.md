# Búsqueda y Recomendación de Textos Legales

La idea es avanzar en el entendimiento y solución de un sistema de búsqueda y recomendación de documentos de texto, el cual se lo instancia en el dominio legal/jurídico utilizando el corpus de leyes de Argentina.
El principal atractivo de un sistema de recomendación reside en que ofrecen información relevante para el usuario acerca de la base de información del dominio en cuestión, sin necesidad de que el mismo tenga conocimientos sobre los artículos recomendados o la consulta que va a realizar. 
Trataremos de responder a las siguientes preguntas:
* ¿Se pueden buscar y recomendar artículos estableciendo automáticamente cuales son en un corpus de datos lo que tienen mayor similitud con el tema buscado? 
* ¿Cuáles son los procesos necesarios basados en Procesamiento de Lenguaje Natural (NLP) para buscar y brindar recomendaciones de normas de leyes, decretos, resoluciones, etc.? 
* ¿Se pueden hacer búsquedas y recomendaciones desde distintas fuentes de información, como artículos periodísticos, fragmentos de contratos, textos con sentido legal como tweets u opiniones de profesionales del ámbito legal?
Una solución a este problema es utilizar una librería llamada Doc2Vec, pero en lugar de usar directamente el paquete cerrado, iremos desglosando el problema para entenderlo.

Los documentos sobre los cuales trabajaremos fueron obtenidos del sitio InfoLEG (http://www.infoleg.gob.ar), el cual es una base de datos de documentos legislativos del Ministerio de Justicia y Derechos Humanos de la Nación, Ministerio que administra además el Sistema Argentino de Información Jurídica (SAIJ).
InfoLEG está conformada por documentos digitales tales como leyes, decretos, decisiones administrativas, resoluciones, disposiciones y todo acto que en sí mismo establezca su publicación obligatoria en la primera sección del Boletín Oficial de la República Argentina.

# Limpieza de Datos

Nuestro objetivo es pasar de una cadena de texto, a una lista de palabras o tokens limpios, que serán útiles para el procesamiento del lenguaje natural.
La limpieza de datos podemos dividirla en 3 grandes pasos:
1. Eliminar ruido
2. Tokenización
3. Normalización

**Eliminar ruido**

Los documentos a procesar vienen en distintos formatos TXT, PDF, DOC, etc. Para obtener el texto hay que quitarles todo tipo de formato y metadatos. En esta oportunidad, trabajamos con documentos TXT y PDF. 

El formato TXT no tiene metadatos, por lo que su lectura no posee mayores inconvenientes.

El formato PDF (Portable Document Format) al estar estructurado por capas, que incluye imágenes, tablas, tipos de fuentes, color. Esto hace que resulte complejo extraer el texto sin formato. Probamos con diferentes librerías, como ser PyMuPDF, pyPDF2 y PDFMiner, obteniendo mejores resultados con PDFMiner.

En ambos casos hay que tener en cuenta la codificación de los caracteres (Encoding). Un encoding es un mapa de caracteres a una representación en bits (por ejemplo 1000001). 
El ASCII es uno de los primeros estándares, pero contempla sólo los caracteres ingleses, es decir no incluye por ejemplo la letra ñ. A raíz de ésto aparecen distintas variantes, siendo el UTF-8 uno de los más utilizados.

**Tokenización**

La tokenización, también conocido como segmentación de texto o análisis léxico, es un paso que divide cadenas de texto más largas en piezas más pequeñas o tokens. Los trozos de texto más grandes pueden ser convertidos en oraciones, las oraciones pueden ser tokenizadas en palabras, etc.

La primera opción que probamos fue utilizar la función split(), pero no obtuvimos una división con sentido lingüístico, ya que por ejemplo, no separó los signos de puntuación que se encuentran al final de cada palabra, como las comas ó los puntos.

Para evitar este inconveniente utilizamos de la librería NLTK, la función nltk.tokenize.toktok.ToktokTokenizer(), la cual obtiene mejores resultados en la lengua española.

**Normalización**

La normalización generalmente se refiere a una serie de tareas relacionadas destinadas a poner todo el texto en igualdad de condiciones:
* convertir el texto en minúscula,
* eliminar signos de puntuación,
* corregir errores de ortografía,
* eliminar stop words,
* Stemming,
* Lematizar 

Uno de los inconvenientes en el tratamiento de textos es la cantidad enorme de tokens que contiene. Para atacar este problema se utilizan las siguientes técnicas:
Eliminar las palabras (tokens) muy comunes en cualquier documento, como por ejemplo “de”, “que”, “y”,  que no aportan al análisis del corpus. Esta lista de palabras se llaman stop word, y pueden ser descargadas desde internet o generadas a partir del  corpus, como veremos más adelante, lo cual resulta en una lista de stop words propia del dominio.
El stemming consiste en extraer la raíz o stem (tronco en inglés) de una palabra . Este proceso se realiza porque la raíz de una palabra puede aparecer más veces en un texto. Por ejemplo, tanto la palabra “tormenta” como “tormentas” tienen como raíz “torment”, y las palabras “tornado”, “tornados”, “tornar” y “tornen” tienen como raíz “torn”.
La lematización es un proceso lingüístico que consiste en, dada una forma flexionada (es decir, en plural, en femenino, conjugada, etc), hallar el lema correspondiente. El lema es la forma que por convenio se acepta como representante de todas las formas flexionadas de una misma palabra. Es decir, el lema de una palabra es la palabra que nos encontraríamos como entrada en un diccionario tradicional: singular para sustantivos, masculino singular para adjetivos, infinitivo para verbos. Por ejemplo, sabemos que “canto”, “cantas”, “canta”, “cantamos”, “cantáis” y “cantan” son distintas formas (conjugaciones) del mismo verbo “cantar”.	

Algunos ejemplos encontrados utilizando 2 librerías distintas:

Raw :  amplíase 	Stemmed NLTK:  ampli 		Lematized NLTK :  amplíase 		Lematized Spacy :  amplíase
Raw :  artículo 		Stemmed NLTK:  articul 		Lematized NLTK :  artículo 		Lematized Spacy :  artículo
Raw :  frutos 		Stemmed NLTK:  frut 			Lematized NLTK :  frutos 		Lematized Spacy :  fruto
Raw :  frescos 		Stemmed NLTK:  fresc 		Lematized NLTK :  fresco 		Lematized Spacy :  fresco
Raw :  expediente 	Stemmed NLTK:  expedient 		Lematized NLTK :  expediente 	Lematized Spacy :  expedientar
Raw :  nº 		Stemmed NLTK:  nº 			Lematized NLTK :  nro 			Lematized Spacy :  número

Observamos una diferencia en la lematización.
* El token frutos, la librería NLTK mantiene el plural y SpaCy si lo transforma a singular.
* El token expediente, la librería NLTK mantiene el singular y SpaCy modifica el término.
* El token nº, la librería NLTK lo transforma nro y SpaCy lo transforma a número

Como conclusiones de la limpieza de datos podemos enumerar las siguientes:

**Proceso iterativo**

Luego de trabajar en las distintas notebooks a lo largo de esta diplomatura, fuimos modificando la función de limpieza de datos permanentemente. Siempre encontramos algo para mejorar o métodos con los cuales no obteníamos los resultados esperados. 

**Diferentes librerías para leer PDFs**

Al momento de leer los PDFs, tuvimos que probar diferentes librerías debido a que no separaban correctamente el texto. 
Algunos no lograban distinguir el texto del formato que lo envolvía, por ejemplo una tabla. En otros aparecían caracteres especiales que nada tenían que ver con el texto.

**Idioma Español**

La mayoría de los trabajos consultados utilizan con documentos en inglés, el cual no tiene caracteres como la ñ o los acentos. Esto nos llevó a probar diferentes criterios para reducir el listado de tokens.
A su vez las librerías que utilizan listas de palabras, como los stop words o la lematización, no están tan completas en el lenguaje español.

**Abreviaturas y Errores de ortografía**

Un problema sobre el cual no encontramos solución fue la cantidad de errores de ortografía, las abreviaturas y diferentes criterios de abreviaturas. Por ejemplo a “número” lo encontramos sin acento, y como “nro”, “n°” y “n.”. 

**Stemming vs Lematizar**

El stemming es mucho más rápido desde el punto de vista del procesamiento que la lematización. También tiene como ventaja que reconoce relaciones entre palabras de distinta clase. Podría reconocer, por ejemplo, que picante y picar tienen como raíz pic-. En otras palabras, el stemming puede reducir el número de elementos que forman nuestros textos. Y eso, en muchos casos, es lo que buscamos.

Una desventaja del stemming es que sus algoritmos son más simples que los de lematización:
overstemming: Pueden “recortar” demasiado la raíz y encontrar relaciones entre palabras que realmente no existen.
understemming: También puede suceder que deje raíces demasiado extensas o específicas, y que tengamos más bien un déficit de raíces, en cuyo caso palabras que deberían convertirse en una misma raíz no lo hacen.

El stemming suele ser una buena solución cuando no importa demasiado la precisión y se requiere de un procesamiento eficiente.

El lematizador busca un lema para la palabra, y la devuelve tal cual si no lo halla. La lematización suele funcionar mejor cuando se necesita procesar palabras de manera similar a como lo hace un ser humano.

Nosotros optamos por utilizar el lematizador WordNetLemmatizer() de la librebrería NLTK.

**Stop word personalizado**

La primera aproximación a las stop word fue utilizando la librería NLTK, y el listado provisto por la misma, pero observamos que era insuficiente, ya que seguían apareciendo palabras muy frecuentes que no eran relevantes.
Esto nos llevó a investigar criterios para generar un listado personalizado de stop word a partir de las frecuencias de las palabras y de la relevancia de la misma en el documento.




