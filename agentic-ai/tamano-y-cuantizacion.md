# Tamaño de un Modelo: Parámetros, Cuantización y Memoria

## La unidad base: parámetros

Un modelo se mide en **parámetros** — los "pesos" que aprendió durante el entrenamiento (7B = 7 mil millones). Más parámetros generalmente significa más capacidad, pero también más pesado y más lento de correr.

## Precisión: cuánto pesa cada parámetro

Los parámetros no siempre se guardan igual — la **precisión numérica** con la que se almacenan determina cuánto ocupa un modelo en disco, independientemente de cuántos parámetros tenga.

```
tamaño en disco (GB) ≈ (parámetros × bytes por parámetro) / 1.000.000.000
```

| Precisión | Bytes/parámetro | Un modelo de 7B pesa aprox. | Calidad |
|---|---|---|---|
| FP32 (full precision) | 4 bytes | ~28 GB | Máxima — rara vez se distribuye así |
| FP16 / BF16 (half precision) | 2 bytes | ~14 GB | La que se usa para entrenar y servir en la nube |
| INT8 | 1 byte | ~7 GB | Pérdida mínima de calidad |
| **Q4 (4-bit)** — default de Ollama | ~0.5–0.6 bytes | ~4–5 GB | Pérdida perceptible, pero aceptable para la mayoría de los usos |
| Q2 (2-bit) | ~0.25–0.3 bytes | ~2–3 GB | Degradación notable — casi nunca vale la pena |

## Cuantización: el trade-off

**Cuantizar** es reducir la precisión numérica de los pesos de un modelo ya entrenado, para que ocupe menos espacio y corra más rápido — a cambio de perder algo de calidad en las respuestas. Es la razón por la que un mismo modelo puede pesar cosas muy distintas según qué versión bajes: Llama 3.3 70B pesa ~140GB en FP16, pero ~40GB en Q4 — el mismo modelo, distinta compresión.

## RAM/VRAM real al correr ≠ solo el tamaño en disco

Al cargar el modelo, la memoria ocupada arranca siendo aproximadamente el tamaño en disco de los pesos — pero durante la generación se suma el **KV cache**: memoria extra que crece con el largo del contexto que se está procesando en esa conversación puntual. Con contextos cortos es un margen chico sobre el tamaño base; con contextos largos (o varias conversaciones corriendo en paralelo) puede sumar bastante más.

```
RAM/VRAM total ≈ tamaño en disco de los pesos (cuantizados) + KV cache (crece con el contexto)
```

## Tabla comparativa — hardware típico por escala de modelo (cuantizado Q4)

| Modelo (parámetros) | Tamaño en disco (Q4) | RAM/VRAM mínima recomendada | Hardware típico |
|---|---|---|---|
| 3B – 8B | ~2–5 GB | 8 GB | Laptop o PC hogareña, sin GPU dedicada |
| 13B – 14B | ~7–9 GB | 16 GB | PC con GPU dedicada de gama media |
| 30B – 34B | ~18–20 GB | 24–32 GB | GPU dedicada de gama alta (ej. RTX 4090) |
| 70B | ~38–40 GB | 48–64 GB | Múltiples GPUs o servidor dedicado |
| 400B+ | ~200 GB+ | Múltiples GPUs de datacenter | Fuera del alcance de hardware personal |

Son órdenes de magnitud, no números exactos — varían según la implementación específica de cuantización (Q4_0, Q4_K_M, etc.) y el largo de contexto que uses.

## Por qué importa

Esto es la letra chica detrás de "self-hosteás un modelo gratis" (ver [self-hosted / open source](costos-llms.md#más-allá-de-la-api-self-hosted--open-source)): el costo no es cero, se traduce directamente en qué hardware necesitás — y eso depende de dos decisiones concretas: qué tamaño de modelo elegís, y con qué nivel de cuantización lo corrés.

---
Relacionado: [Costos de LLMs](costos-llms.md), [Comparación de Modelos](comparacion-modelos.md), [Qué es un token](que-es-un-token.md).
