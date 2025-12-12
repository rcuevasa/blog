---
title: "Usando chains, tools y agentes localmente con LangChain y Ollama"
date: 2025-12-12
---

## Introducción

Hace algún tiempo tuve intenciones de jugar con LangChain y Ollama para aprender sobre la implementación de soluciones usando modelos IA. Un par de semanas atrás instalé Ollama y comencé a revisar la api, y a hacer pruebas, principalmente acerca del tiempo de respuesta para validar que podía utilizarlo experimentalmente sin que las esperas fueran muy grandes.

Este artículo relata sucintamente la experiencia del desarrollo de un proyecto experimental con LangChain adaptado para usar modelos de IA locales a través de Ollama, en lugar de servicios como el de OpenAI. Lo que inició como un sencillo proceso de aprendizaje, siguiendo un tutorial de youtube, se transformó en un desafío técnico que requirió un poco de investigación y bastante experimentación para superar las limitaciones del ecosistema local.

## Contexto del Proyecto

El objetivo del proyecto era entender el uso de cadenas (chains) de LangChain e implementar agentes con herramientas (tools). Para esto decidí utilizar un tutorial de YouTube de septiembre de 2023, por lo que hubo que adaptar el código para utilizar Ollama con distintos modelos locales de IA. El tutorial utiliza el servicio OpenAI.

### Principales Características

- **Generador de nombres para mascotas**: Utiliza cadenas de LangChain para generar nombres creativos basados en especie o tipo de animal y color.
- **Interfaz Streamlit**: Web simple para interactuar con el generador.
- **Soporte de modelos locales**: Funciona con modelos locales en ejecución.
- **Experimentación con agentes**: Intentos iniciales de crear agentes con herramientas (tools).

## El Desafío Principal: Adaptándose a Ollama

El primer desafío fue reemplazar las llamadas realizadas a OpenAI con las llamadas necesarias a Ollama:

```python
llm = OpenAI(temperature=0.7, openai_api_key=openai_api_key)

prompt_template_name = PromptTemplate(
        input_variables = [...],
        template = "..."
    )

name_chain = LLMChain(llm=llm, prompt=prompt_template_name, output_key="pet_name")

response = name_chain({'animal_type': animal_type, 'pet_color': pet_color})
```

La llamada inicial más simple posible fue creada usando directamente ollama sin pasar por LangChain. Cabe desstacar que el método `generate` asume que la conexión es local sin necesidad de pasar argumentos.

```python
# Define prompt template
prompt_template = PromptTemplate(
        input_variables = [...],
        template = "..."
    )

response = ollama.generate(model="qwen3", template='prompt_template', format="json")
```
Luego, la incorporación de LangChain requirió algunas modificaciones. El principal desafío surgió de la fecha del tutorial y los cambios operados en el framework de LangChain. Inicialmente intenté usar los mismos ejemplos o similares con poco éxito, y fui comprendiendo la vorágine de transformaciones que han sufrido estos framewoeks en los últimos dos años. El tutorial es de Diciembre de 2023. Varias clases y métodos utilziados en el tutorial estaban obsoletos o habían sufrido modificaciones requiriendo revisar la documentación y la adaptación del código.

```
# Define prompt template
prompt_template = PromptTemplate(
     input_variables=[...],
     template='...',
)

model = OllamaLLM(
    model="...",
    base_url="http://localhost:11434",
)
    
output_parser = JsonOutputParser()

chain = prompt_template | model | output_parser
return response = chain.invoke({...})
```

El segundo y principal desafío surgió de la fecha del tutorial y las versiones utilizadas entonces. Desde septiembre de 2023 hasta el presente (Diciembre 2025), el framework LangChain/Ollama ha experimentado cambios sustanciales. Varias clases y métodos utilizados en el tutorial estaban obsoletas o habían sufrido modificaciones, requiriendo investigación constante y adaptación del código.

## El Obstáculo Crítico: Implementación de Herramientas

El desafío más significativo fue la falta del método `bind_tools()` en la implementación de Ollama, un componente esencial para crear agentes funcionales. Mientras que con OpenAI este proceso es directo:

```python
# Con OpenAI sería algo así para la fecha del tutorial
llm = OpenAI(temperature=0.7, openai_api_key=openai_api_key)
tools = load_tools([...], llm=llm)
agent = initialize_agent(tools)
```

Con Ollama, este enfoque simplemente no funcionaba, requiriendo una solución alternativa.

## La Solución: Implementación Directa con la API de Ollama

Después de una extensa investigación y experimentación, encontré una solución al problema del uso de las tools mediante el uso directo de la API de chat de Ollama, pero sin pasar por LangChain:

```python
def tool_a()
def tool_b()
available_tools = {...}

mytools = [...]

messages = [...]

result: ChatResponse = chat(
    model='...',
    messages=messages,
    tools=mytools
)
```

Este enfoque permitió superar las limitaciones de la integración LangChain-Ollama, implementando manualmente el ciclo de la herramienta:

```python
if result.message.tool_calls:
    for tool in result.message.tool_calls:
        if function_to_call := available_tools.get(tool.function.name):
            print('Llamando a función:', tool.function.name)
            print('Argumentos:', tool.function.arguments)
            output = function_to_call(**tool.function.arguments)
            print('Salida de función:', output)
            messages.append(result.message)
            messages.append({'role': 'tool', 'content': str(output), 'tool_name': tool.function.name})
```

## Lecciones Aprendidas

### 1. La Documentación Puede Quedar Rápidamente Desactualizada

El ecosistema de IA evoluciona vertiginosamente. Los tutoriales de hace solo meses pueden volverse obsoletos rápidamente.

### 2. Las Limitaciones de Framework Exigen Soluciones Alternativas

Cuando una implementación estándar no funciona, como en el caso de la ausencia de la inmplementación de `bind_tools()`, es necesario profundizar en el entendimiento de la API subyacente y buscar soluciones directas.

### 3. La Experimentación es Clave

El proyecto requirió probar múltiples estrategias y enfoques hasta encontrar una compatible con el objetivo.

## Conclusión

Este proyecto demuestra que incluso un tutorial simple puede presentar desafíos significativos cuando se adapta a diferentes tecnologías. La perseverancia y la investigación fueron fundamentales para superar los obstáculos presentados al usar Ollama con un modelo IA local en lugar de OpenAI.

La solución final, aunque más compleja que la implementación estándar con OpenAI, proporciona una base funcional para construir agentes con herramientas en un entorno local, ofreciendo privacidad y control sin depender de servicios externos.

## Código Fuente

El proyecto completo está disponible en [langchain-test](https://github.com/rcuevasa/langchain-test/) y muestra la evolución desde la implementación básica hasta la solución funcional de agentes con herramientas. El proyecto no ha sido concluido aún, aun cuando, ya cumplió su objetivo inicial. El proyecto tiene dos branches, main que contiene la etapa inicial del desarrollo del generador de nombres de mascotas. El segundo branch se llama test-tools, donde el objetivo fue responder una pregunta asociada a una fecha, en particular la pregunta fue "¿Quién es el actual presidente de los EE.UU.?". 
