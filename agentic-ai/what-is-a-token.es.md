# Qué es un token

La unidad básica que un LLM procesa — **no es una palabra, ni una sílaba, ni tiene que ver con el significado**. Es un pedazo de texto (una subpalabra) que el *tokenizer* decidió tratar como una sola unidad, en base a qué tan seguido aparece ese pedazo en los datos con los que se entrenó el modelo.

## No es semántico — es estadístico

El proceso que parte el texto en tokens (tokenización, típicamente algo como *Byte-Pair Encoding*) no sabe nada de gramática ni de significado — junta los pares de caracteres que más seguido aparecen juntos en el corpus de entrenamiento, repetidamente, hasta formar un vocabulario de "pedazos" frecuentes. El resultado: palabras comunes suelen ser un solo token, palabras raras o largas se parten en varios pedazos que no necesariamente coinciden con morfemas reales.

```
"the" → 1 token (aparece muchísimo, es su propio token)
"unbelievable" → probablemente 3 tokens: "un" + "believ" + "able"
                  (coincide más o menos con morfemas acá, pero es casualidad,
                   no es que el tokenizer "entienda" que "un-" es un prefijo)
```

## ~4 caracteres por token, como regla rápida

En inglés, la aproximación típica es ~4 caracteres o ~0.75 palabras por token. **En español y otros idiomas suele ser menos eficiente** — los tokenizers se entrenan con más datos en inglés, así que un texto en español puede necesitar más tokens para decir lo mismo que su equivalente en inglés. Es la razón concreta detrás de algo que ya hablamos: escribir en inglés consume, en general, menos tokens que el mismo contenido en español.

## Cómo el modelo elige el próximo token

Generar texto es, en el fondo, predecir **un token a la vez**: dado todo el texto previo, el modelo calcula qué tan probable es cada token posible del vocabulario como "siguiente", y elige uno — después repite el proceso con ese token ya incluido en el contexto, uno por uno, hasta terminar.

```
Prompt: "El sol es..."

El modelo no "sabe" la respuesta — calcula una probabilidad (logit) para
cada palabra candidata del vocabulario, y las normaliza con softmax
para que sumen 100%:

  "amarillo"  → 0.5  (50% de probabilidad)
  "rojo"      → 0.3  (30%)
  "brillante" → 0.2  (20%)

Con temperatura baja, elige casi siempre la más probable ("amarillo").
Con temperatura alta, hay más chance de que elija una opción menos obvia.
```

**Softmax** es la función que convierte esos puntajes crudos (*logits*) en probabilidades que suman 1 — así el modelo puede "elegir" según esa distribución en vez de comparar números arbitrarios sin escala común.

**Temperatura** (típicamente 0 a 2) controla qué tan determinística o creativa es esa elección:
- **Temperatura baja (cerca de 0)**: casi siempre elige el token más probable — respuestas más consistentes y predecibles, ideal para tareas donde no querés variación (clasificación, extracción de datos).
- **Temperatura alta (cerca de 2)**: más probabilidad de elegir tokens menos obvios — respuestas más variadas/creativas, pero también más erráticas.

## Test-Time Compute — pensar más al responder, no al entrenar

Hasta acá, escalar un LLM significaba entrenarlo con más parámetros y más datos — ese cómputo se gasta **una sola vez**, durante el entrenamiento, y después el modelo responde a la misma velocidad sin importar si la pregunta es trivial o muy difícil. **Test-Time Compute** (o *inference-time scaling*) es la idea inversa: dejar que el modelo gaste más cómputo **al momento de responder**, generando un razonamiento más largo antes de dar la respuesta final, cuando la tarea lo amerita.

Es la técnica detrás de los modelos "de razonamiento" (OpenAI o1/o3, Claude con *extended thinking*): en vez de ir directo a la respuesta, el modelo genera una cadena de razonamiento interna extensa antes de contestar — la misma idea de fondo que pedirle [Chain of Thought](prompt-engineering.es.md#chain-of-thought-cot) en el prompt, pero automático, mucho más largo, y entrenado específicamente para eso (en vez de depender de que el usuario lo pida).

```
Pregunta simple: "¿Capital de Francia?"
  → poco test-time compute necesario, responde casi directo

Pregunta compleja: "Demostrá que la suma de los primeros n números impares es n²"
  → el modelo "piensa" más antes de responder: explora varios caminos
    de razonamiento, descarta algunos, converge en uno, recién ahí contesta
```

**El trade-off**: más tokens de "pensamiento" = más [costo](llm-costs.es.md) y más latencia — no tiene sentido gastarlo en preguntas triviales. La ventaja real aparece en tareas de varios pasos lógicos (matemática, debugging, planificación), donde razonar más antes de contestar cambia el resultado.

## Multimodal — tokens más allá del texto

Los modelos multimodales tokenizan y procesan más que texto — audio e imágenes también se convierten en algún tipo de "token" que el modelo puede razonar en el mismo espacio que el texto (una imagen se parte en patches que funcionan como tokens, por ejemplo). El mecanismo de fondo (convertir el input a unidades discretas, predecir la salida token por token) es el mismo, solo cambia qué representa cada token.

## Por qué importa

- **Precio**: los proveedores cobran por token, no por carácter ni por palabra — ver [Costos de LLMs](llm-costs.es.md).
- **Context window**: el límite de cuánto texto puede "ver" un modelo a la vez se mide en tokens, no en palabras — un context window de 200K tokens no son 200K palabras, son bastantes menos (según el idioma y el contenido).
- **Por qué un LLM a veces "corta raro" una palabra rara o un nombre propio**: si esa palabra nunca apareció seguido en el entrenamiento, el tokenizer la parte en varios pedazos poco intuitivos — es más frecuente con nombres propios, jerga técnica muy específica, o texto en un idioma con poca representación en el entrenamiento.

---
Relacionado: [Costos de LLMs](llm-costs.es.md), [De ML clásico a Agentic AI](from-ml-to-agentic-ai.es.md), [Prompt Engineering](prompt-engineering.es.md), [Context Engineering](context-engineering.es.md).
