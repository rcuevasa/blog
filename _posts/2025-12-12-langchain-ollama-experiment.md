---
title: "Usando chains, tools y agentes localmente con LangChain y Ollama"
date: 2025-12-12
---

## Introducción

Este artículo relata la experiencia técnica durante el desarrollo de un proyecto experimental de LangChain adaptado para usar modelos locales a través de Ollama, en lugar del servicio OpenAI. Lo que comenzó como un simple seguimiento de tutorial se transformó en un desafío técnico que requirió investigación profunda y experimentación para superar las limitaciones específicas del ecosistema local.

## Contexto del Proyecto

El proyecto tenía como objetivo entender las cadenas (chains) de LangChain y eventualmente implementar agentes con herramientas (tools). Siguiendo un tutorial de YouTube de septiembre de 2023, se adaptó el código para utilizar Ollama con modelos locales como qwen, deepseek-r1 y llama3.1.

### Características Principales

- **Generador de nombres para mascotas**: Utiliza cadenas de LangChain para generar nombres creativos basados en tipo y color
- **Interfaz Streamlit**: Web simple para interactuar con el generador
- **Soporte de modelos locales**: Funciona con modelos locales en ejecución
- **Experimentación con agentes**: Intentos iniciales de crear agentes con herramientas

## El Desafío Principal: Adapting a Ollama

El primer obstáculo significativo surgió de la desactualización del tutorial. Desde septiembre de 2023 hasta la fecha actual, el framework LangChain ha experimentado cambios sustanciales:

```python
# Ejemplo de código inicial que presentaba problemas
from langchain_ollama import OllamaLLM
from langchain.agents import create_agent

model = OllamaLLM(
    model="deepseek-r1",
    base_url="http://localhost:11434",
    stream=False
)
```

Varias clases y métodos utilizados en el tutorial fueron deprecados o modificados, requiriendo investigación constante y adaptación del código.

## El Obstáculo Crítico: Implementación de Herramientas

El desafío más significativo fue la falta del método `bind_tools()` en la implementación de Ollama, un componente esencial para crear agentes funcionales. Mientras que con OpenAI este proceso es directo:

```python
# Con OpenAI sería algo así
agent = create_agent(model, tools)
agent.invoke({"messages": [{"role": "user", "content": "question"}]})
```

Con Ollama, este enfoque simplemente no funcionaba, requiriendo una solución alternativa.

## La Solución: Implementación Directa con la API de Ollama

Después de extensiva investigación y experimentación, se encontró una solución mediante el uso directo de la API de chat de Ollama:

```python
from ollama import ChatResponse, chat
from datetime import date

def current_date() -> str:
    """Obtener la fecha actual."""
    return str(date.today())

def tool_wikipedia(query: str) -> str:
    """Buscar una consulta en wikipedia."""
    wikipedia.set_lang("en")
    search_results = wikipedia.summary(query)
    return search_results

available_tools = {
    'tool_wikipedia': tool_wikipedia,
    'current_date': current_date,
}

mytools = [tool_wikipedia, current_date]

messages = [
    {
        "role": "system", 
        "content": "Eres un asistente útil con herramientas disponibles."
    },
    {
        "role": "user", 
        "content": "¿Cuál es la fecha actual?"
    },
    {
        "role": "user",
        "content": "¿Quién es el presidente de Estados Unidos? Usa la herramienta de wikipedia para averiguarlo."
    }
]

result: ChatResponse = chat(
    model='qwen3',
    messages=messages,
    tools=mytools
)
```

Este enfoque permitió superar las limitaciones de la integración LangChain-Ollama, implementando manualmente el ciclo de herramienta:

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

El ecosistema de IA evoluciona rápidamente. Los tutoriales de hace solo meses pueden volverse obsoletos rápidamente.

### 2. Las Limitaciones de Framework Exigen Soluciones Alternativas

Cuando una implementación estándar no funciona, como en el caso de `bind_tools()`, es necesario profundizar en la API subyacente y buscar soluciones directas.

### 3. La Experimentación es Clave

El proyecto requirió probar múltiples modelos (deepseek-r1, qwen3) y enfoques hasta encontrar uno compatible con las herramientas necesarias.

## Conclusión

Este proyecto demuestra que incluso un tutorial simple puede presentar desafíos técnicos significativos cuando se adapta a diferentes tecnologías. La perseverancia y la investigación fueron fundamentales para superar los obstáculos particulares de usar Ollama en lugar de OpenAI.

La solución final, aunque más compleja que la implementación estándar con OpenAI, proporciona una base funcional para construir agentes con herramientas en un entorno local, ofreciendo privacidad y control sin depender de servicios externos.

## Código Fuente

El proyecto completo está disponible en [repositorio del proyecto] y muestra la evolución desde la implementación básica hasta la solución funcional de agentes con herramientas.

---

*Este artículo refleja la experiencia técnica real durante el desarrollo del proyecto LangChain con Ollama, destacando que la adaptación de tecnologías requiere no solo conocimiento técnico sino también creatividad para encontrar soluciones alternativas.*
