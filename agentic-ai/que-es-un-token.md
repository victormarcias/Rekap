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

## Por qué importa

- **Precio**: los proveedores cobran por token, no por carácter ni por palabra — ver [Costos de LLMs](costos-llms.md).
- **Context window**: el límite de cuánto texto puede "ver" un modelo a la vez se mide en tokens, no en palabras — un context window de 200K tokens no son 200K palabras, son bastantes menos (según el idioma y el contenido).
- **Por qué un LLM a veces "corta raro" una palabra rara o un nombre propio**: si esa palabra nunca apareció seguido en el entrenamiento, el tokenizer la parte en varios pedazos poco intuitivos — es más frecuente con nombres propios, jerga técnica muy específica, o texto en un idioma con poca representación en el entrenamiento.

---
Relacionado: [Costos de LLMs](costos-llms.md), [De ML clásico a Agentic AI](historia-de-ml-a-agentic.md).
