---
title: "Asistente IA para la Ley de Protección de Datos 21719 con Langchain, Ollama y ChromaDB"
date: 2025-12-30
---

## Introducción

Como continuación de mi [primer](https://rcuevasa.github.io/blog/2025/12/12/langchain-ollama-experiment.html) intento por explorar la construcción de agentes IA para entornos locales, he construido una segunda aplicación intentando crear una solución práctica para un problema contingente.

La Ley de protección de datos 21.719 moderniza la normativa anterior que data de 1999, y fue publicada el 13 de Diciembre de 2024, sin embargo esta entrará en plena vigencia el 1 de Diciembre de 2026. Es decir, a empresas y organizaciones afectas les queda poco menos de un año para dar cumplimiento con la nueva Ley.

Debido a esta necesidad me surgió la idea de crear un asistente IA, que pudiera ser utilizado para ayudar a personas y empresas a responder preguntas sobre la ley e incluso a elaborar diagnósticos y planes en relación al proceso de cumplimiento de esta. En ese sentido, una posible pregunta que surge es ¿Por qué no usar un chatbot general disponible a través de internet que posiblemente será muchísimo más rápido y tal vez preciso que un modelo local? La respuesta obvia fue porque intentando dar cumplimiento a la ley de protección de datos una organización o persona podría terminar vulnerando el principal objetivo al subir datos sensibles a cualquier chat IA. He ahí donde surge la necesidad de implementar una solución RAG (Retrieval-Augmented Generation).

Entonces cuál sería el entorno adecuado, seguro y protegido que permitiera contar con el poder de un asistente IA que no solo ayude a construir diagnósticos y planes sobre la ley en si misma, sino que más importante aún incorpore al análisis el contexto concreto de la organización, vale decir, sus datos, estructuras de datos, políticas, estructura administrativa, etc.

Entonces, eso es exáctamente lo que he construido, un asistente IA para entornos locales (o bajo estricto control de seguridad de la organización), que permite cargar en una base de datos vectorial tanto la ley 21.719 como cualquier otra documentación de la organización, políticas de seguridad, políticas de clasificación de la información, definiciones administrativas como el encargado de la protección de datos, de manera de que el asistente cuente con el contexto adecuado para evaluar el cumplimiento, indicar correcciones y mejoras posibles, evaluar el nivel de cumplimiento de la organización en relación a la ley de protección de datos.

## Características de la solución

- Especializado en la ley 21.719 de Protección de Datos de Chile. 
- Utiliza modelos IA locales sobre OLLAMA para generación de respuestas en un entorno local seguro. Los modelos pueden ser cambiados a elección. 
- Sistema RAG (Retrieval-Augmented Generation) para obtener respuestas más relevantes y de mejor calidad.
- Soporte de carga de múltiples documentos en formato PDF y CSV. Específicamente pensados  para cargar documentación y muestras de datos.
- Utilización de Base Vectorial local ChromaDB para el almacenamiento de la documentación y una recuperación de información rápida y precisa.

## Principales desafíos

### Desafíos de Diseño

#### 1. Experiencia de Usuario

- Mejorar la estructura y mensajes de la interfaz

#### 2. Arquitectura Escalable

- Separar responsabilidades de gestión de estado
- Crear un sistema reutilizable para manejo de variables
- Evitar duplicación de código entre módulos

#### 3. Mantenimiento del Entorno

- Limpiar eficientemente el entorno de desarrollo
- Facilitar el restablecimiento a estado inicial
- Eliminar documentos y base vectorial sistemáticamente 

### Desafíos Técnicos

#### 4. Gestión de Variables de Entorno Dinámicas

El desafío más evidente fue la necesidad de gestionar múltiples documentos dinámicamente.

- Creación del módulo `variable_helper.py`
- Implementación de  `assure_read_variables()` para manejar diferentes formatos de variables
- Sistema para actualizar variables en el archivo `.env` dinámicamente

#### 5. Filtrado de Búsqueda Vectorial por Tipo de Documento

- Implementación de `update_filter_criteria()` para búsquedas de similaridades filtradas por documento (source metadata)
- Asignación de diferentes valores de k según tipo (15 para ley, 5 para CSV, 10 para PDF), para mantenernos dentro de los límites del modelo (32K)
- Uso de metadatos de origen para distinguir entre documentos

#### 6. Manejo de Múltiples Tipos de Documento

- Transición de un único documento legal a múltiples CSVs y PDFs para respuestas con contexto de la organización.
- Necesidad de diferentes rutas de almacenamiento ( data/laws  vs  data/documents )
- Actualización automática de la base vectorial al agregar nuevos documentos

#### 7. Persistencia del Estado entre Ejecuciones

- Necesidad de persistir qué documentos están cargados
- Dificultad para mantener el estado entre recargas de Streamlit
- Implementación de reinicios automáticos después de cargar documentos

#### 8. Procesamiento de Diversos Formatos de Datos

El soporte para CSVs introdujo desafíos adicionales:

- Diferentes métodos de carga ( PyPDFLoader  vs  CSVLoader )
- Necesidad de diferentes ponderaciones en la búsqueda vectorial sin sobrepasar los límites del modelo
- Procesamiento de datos estructurados vs texto libre

## Posibles extensiones y mejoras 
- Agregar soporte para otro tipo de documetnos (Word, Excel, etc.)
- Implementación de múltiples colecciones de ChromaDB, en reemplazo de una única colección para toda la información.
- Sistema de autenticación de usuarios
- Integración con bases de datos externas, configurables.
- Reemplazo de langchain chains por agents y tools. 

## Conclusiones

En esta ocasión, con la experiencia ganada en el anterior proyecto me fue relativamente sencillo construir los elementos relacionados con el uso de la IA, específicamente el uso de cadenas (chains) y la integración de la base de datos vectorial. Las dificultades estuvieron por el lado de implementación de las características hacia el usuario. Por ejemplo, permitir la carga de múltiples documentos, o la actualización de las variables de entorno me dieron un buen dolor de cabeza debido a mi inexperiencia en el manejo de variables globales/locales y su comportamiento. Es evidente que la aplicación está sujeta a potenciales mejoras. Lo mejor de todo es que he dado un paso interesante para mi al construir una aplicación que puede ser útil para otras personas u organizaciones en términos concretos, además del aprendizaje ganado en el proceso.

En segundo lugar, resulta interesante como el uso del llamado "vibe-coding" no sirve de mucho a la hora de intentar implementar soluciones con un  cierto grado de complejidad, hice un intento de "vibe-coding" que resultó un verdadero desastre. Luego de esa falla, debí recomenzar haciendo todo a mano y desde cero, consultando tutoriales, referencias de las api de langchain, chroma, etc. ¿Eso quiere decir que no utilizo un asistente de programación? Por supuesto que lo uso, de hecho uso dos, copilot en vscode y en vim, y crush (GLM). Pero siempre para tareas puntuales, para solución de errores, y para entender cosas que desconozco principalmente. También lo uso para escribir rápidamente el README, de donde también saco algunos elementos para este artículo.

## Código Fuente
El proyecto completo está disponible en [dpl-assist](https://github.com/rcuevasa/dpl_assist), ahí podrán encontrar las instrucciones de instalación, instalación de dependencias, etc. Cualquier consulta o sugerencia no duden en contactarme.
