# De ML clásico a Agentic AI

Cómo se llegó de "entrenar un modelo para una tarea puntual" a "un LLM que planea y ejecuta tareas de varios pasos solo" — el contexto necesario para entender por qué agentic es lo que es hoy, y no un invento sin historia atrás.

## 1. Machine Learning clásico — un modelo, una tarea

Se entrena un modelo con datos etiquetados para que aprenda a resolver **una tarea específica** (clasificar spam, predecir un precio, agrupar clientes). El modelo no "entiende" nada fuera de eso — un modelo que predice precios de casas no sirve para clasificar emails, hay que entrenar uno nuevo desde cero.

```python
# ejemplo del patrón clásico: entrenar UN modelo para UNA tarea puntual
from sklearn.linear_model import LogisticRegression

model = LogisticRegression()
model.fit(X_train, y_train)  # aprende de datos etiquetados, específicos de este problema
model.predict(X_new)          # solo sabe hacer esto — clasificar spam, en este ejemplo
```

## 2. Deep Learning — el modelo aprende sus propias features (2012+)

Antes de esto, alguien tenía que diseñar a mano qué características del dato importaban (*feature engineering*). El momento bisagra fue **AlexNet** (2012, ImageNet) — una red neuronal profunda que aprendía las features por sí sola a partir de las imágenes en bruto, superando por lejos a los métodos anteriores. Seguía siendo "un modelo, una tarea" (visión, texto, etc. por separado), pero ya no hacía falta diseñar las features a mano.

## 3. Transformers — la arquitectura base de todo lo que vino después (2017)

El paper *"Attention Is All You Need"* (2017) introdujo una arquitectura que procesa secuencias completas de una vez (en vez de una palabra a la vez, como las RNNs anteriores), prestando "atención" a qué partes del input importan más para cada parte del output. Es la arquitectura detrás de prácticamente todo LLM actual.

## 4. Modelos preentrenados — un modelo base, muchos usos (2018-2020)

BERT y GPT (versión original) cambiaron el patrón de "un modelo por tarea" a **preentrenar un modelo grande una sola vez** sobre texto masivo (sin etiquetas, *self-supervised*), y después ajustarlo (*fine-tuning*) con poco esfuerzo para tareas específicas. El conocimiento general del lenguaje quedaba en el modelo base; el fine-tuning solo lo especializaba.

## 5. LLMs y la era del prompting (2020, mainstream en 2022)

GPT-3 (2020) mostró algo nuevo: con un modelo lo suficientemente grande, ya no hacía falta fine-tuning para resolver una tarea nueva — alcanzaba con **describirla en el prompt** (*few-shot*/*zero-shot learning*). El lanzamiento de ChatGPT (noviembre 2022) fue el momento en que esto se volvió mainstream fuera del mundo técnico.

```
Prompt: "Clasificá este email como spam o no spam: '¡Ganaste un premio, hacé click acá!'"
→ el mismo modelo que escribe código, traduce, y resume, ahora también clasifica —
  sin haber sido entrenado específicamente para esto, solo con la instrucción en el prompt
```

## 6. RAG — darle al LLM información que no tiene (2023+)

Un LLM solo "sabe" lo que vio en su entrenamiento — no conoce datos privados de una empresa, ni eventos posteriores a su fecha de corte. **Retrieval-Augmented Generation**: antes de responder, se busca información relevante (típicamente en una base de datos vectorial) y se la agrega al prompt como contexto, para que el modelo responda basado en eso en vez de inventar.

## 7. Tool Use / Function Calling — el LLM puede hacer, no solo hablar (2023+)

Hasta acá, un LLM solo devolvía texto. Con *function calling*, el modelo puede decidir "para responder esto, necesito llamar a esta función" (buscar en una API, consultar una DB, mandar un email) — el LLM elige qué herramienta usar y con qué argumentos, el código de la aplicación la ejecuta de verdad y le devuelve el resultado.

## 8. Agentic AI — planear, actuar, observar, repetir (2023-2024+)

La unión de todo lo anterior en un **loop**: el LLM no responde una sola vez — planea los pasos necesarios, ejecuta una acción (usando tool use), observa el resultado, decide el siguiente paso, y repite hasta completar la tarea o decidir que terminó. Es la diferencia entre "preguntarle algo a un chatbot" y "pedirle que resuelva un problema de punta a punta". Este mismo patrón es, literalmente, cómo funciona esta conversación: cada vez que edito un archivo del repo, reviso el resultado, y decido el siguiente paso, es un agente ejecutando ese loop.

```
Usuario: "Agregá un archivo nuevo y linkealo desde el índice"
  → Agente planea: 1) crear archivo, 2) editar índice, 3) verificar que no rompió nada
  → Agente ejecuta paso 1 (tool: escribir archivo)
  → Agente observa: ¿salió bien? sí → sigue al paso 2
  → Agente ejecuta paso 2 (tool: editar índice)
  → Agente observa, verifica, reporta que terminó
```

---
Relacionado: ver el resto de `agentic-ai/` para el detalle de cada pieza (RAG, tool use, arquitectura de agentes) a medida que se van agregando.
