# Modelos Generativos

Modelos que crean contenido nuevo — texto, imágenes, audio, video, código — en vez de asignarle una etiqueta a un input. La IA generativa (GenAI) es el campo construido alrededor de ellos; los LLMs son el caso que genera texto.

| | Discriminativo | Generativo |
|---|---|---|
| Pregunta que responde | "¿De qué categoría es esto?" | "¿Cómo son los datos de este tipo?" → produce muestras nuevas |
| Output | Una etiqueta o un número | Contenido nuevo |
| Ejemplo | Clasificador de spam, detección de fraude | ChatGPT, Stable Diffusion, generación de música |

## Familias principales

| Familia | Cómo genera | Fortalezas | Debilidades | Ejemplos |
|---|---|---|---|---|
| **GAN** (2014) | Un *generador* crea muestras y un *discriminador* intenta distinguirlas de las reales — se entrenan compitiendo | Imágenes nítidas, generación rápida | Entrenamiento inestable, poca variedad (*mode collapse*) | StyleGAN, los primeros deepfakes |
| **VAE** (2013) | Comprime los datos en un *espacio latente* y aprende a decodificar muestras desde ahí | Entrenamiento estable, espacio latente suave | Resultados más borrosos | Se usa como componente dentro de Stable Diffusion |
| **Autorregresivo** (transformers) | Un [token](what-is-a-token.es.md) a la vez, cada uno condicionado por los anteriores | Texto y código, sigue instrucciones | Generación secuencial, un token por paso | GPT, Claude, Gemini, Llama |
| **Diffusion** (2020+) | Arranca de ruido puro y lo va sacando paso a paso hasta que aparece una imagen | La mejor calidad y variedad en imagen/video | Más lento: muchos pasos por imagen | Stable Diffusion, DALL·E 3, Imagen, Sora |

## Cómo funciona diffusion

```
Entrenamiento:  imagen ──sumar ruido──▶ más ruidosa ──▶ ... ──▶ ruido puro
                (el modelo aprende a predecir cuánto ruido se sumó en cada paso)

Generación:     ruido puro ──quitar ruido──▶ ... ──quitar ruido──▶ imagen
                (cada paso va guiado por el prompt de texto)
```

- **Proceso forward**: se le suma ruido a una imagen real de a poco hasta que no queda nada de ella. Es una fórmula fija, acá no se aprende nada.
- **Proceso reverse**: una red neuronal aprende a deshacer un paso — dada una imagen con ruido, predecir el ruido para poder restarlo. Repitiendo eso desde ruido aleatorio puro sale una imagen completamente nueva.
- **Condicionamiento por texto**: el prompt pasa por un encoder de texto (ej. CLIP) y guía cada paso de limpieza. El *guidance scale* controla qué tan al pie de la letra sigue la imagen al prompt.
- **Latent diffusion**: en vez de limpiar millones de píxeles, trabaja en un espacio latente comprimido (vía un VAE) y recién al final decodifica a píxeles — lo que permitió que Stable Diffusion corriera en una GPU de consumo.

```python
from diffusers import AutoPipelineForText2Image

pipe = AutoPipelineForText2Image.from_pretrained("stabilityai/stable-diffusion-xl-base-1.0").to("cuda")

image = pipe(
    "a watercolor fox in a snowy forest",
    num_inference_steps=30,  # más pasos de limpieza = más detalle, más lento
    guidance_scale=7.5,      # más alto = sigue el prompt más literal, menos variedad
).images[0]
image.save("fox.png")
```

## Riesgos específicos

- **Deepfakes y desinformación**: imágenes, voces y videos realistas de personas reales.
- **Copyright**: modelos entrenados con contenido cuyos autores no dieron su consentimiento.
- **Procedencia**: para distinguir contenido generado existen marcas de agua invisibles (ej. SynthID) y metadata firmada (C2PA).

Los riesgos generales de los sistemas de IA están en [Riesgos y Mitigaciones](risks-and-mitigations.es.md).

---
Relacionado: [De ML clásico a Agentic AI](from-ml-to-agentic-ai.es.md), [Foundation Models](foundation-models.es.md), [Qué es un token](what-is-a-token.es.md#multimodal--tokens-más-allá-del-texto), [Riesgos y Mitigaciones](risks-and-mitigations.es.md).
