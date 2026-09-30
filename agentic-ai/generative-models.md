# Generative Models

Models that create new content — text, images, audio, video, code — instead of assigning a label to an input. Generative AI (GenAI) is the field built around them; LLMs are the text-generating case.

| | Discriminative | Generative |
|---|---|---|
| Question it answers | "Which category is this?" | "What does data like this look like?" → produces new samples |
| Output | A label or a number | New content |
| Example | Spam classifier, fraud detection | ChatGPT, Stable Diffusion, music generation |

## Main families

| Family | How it generates | Strengths | Weaknesses | Examples |
|---|---|---|---|---|
| **GAN** (2014) | A *generator* creates samples and a *discriminator* tries to tell them apart from real ones — they train by competing | Sharp images, fast generation | Unstable training, low variety (*mode collapse*) | StyleGAN, early deepfakes |
| **VAE** (2013) | Compresses data into a *latent space* and learns to decode samples from it | Stable training, smooth latent space | Blurrier outputs | Used as a component inside Stable Diffusion |
| **Autoregressive** (transformers) | One [token](what-is-a-token.md) at a time, each conditioned on the previous ones | Text and code, follows instructions | Sequential generation, one token per step | GPT, Claude, Gemini, Llama |
| **Diffusion** (2020+) | Starts from pure noise and removes it step by step until an image appears | Highest quality and variety in images/video | Slower: many steps per image | Stable Diffusion, DALL·E 3, Imagen, Sora |

## How diffusion works

```
Training:    image ──add noise──▶ noisier ──▶ ... ──▶ pure noise
             (the model learns to predict how much noise was added at each step)

Generation:  pure noise ──denoise──▶ ... ──denoise──▶ image
             (each step is guided by the text prompt)
```

- **Forward process**: noise gets added to a real image little by little until nothing is left of it. It's a fixed formula, nothing is learned here.
- **Reverse process**: a neural network learns to undo one step — given a noisy image, predict the noise so it can be subtracted. Repeating that from pure random noise produces a brand-new image.
- **Text conditioning**: the prompt goes through a text encoder (e.g. CLIP) and guides every denoising step. The *guidance scale* controls how strictly the image follows the prompt.
- **Latent diffusion**: instead of denoising millions of pixels, it works in a compressed latent space (via a VAE) and decodes to pixels only at the end — what made Stable Diffusion able to run on a consumer GPU.

```python
from diffusers import AutoPipelineForText2Image

pipe = AutoPipelineForText2Image.from_pretrained("stabilityai/stable-diffusion-xl-base-1.0").to("cuda")

image = pipe(
    "a watercolor fox in a snowy forest",
    num_inference_steps=30,  # more denoising steps = more detail, slower
    guidance_scale=7.5,      # higher = follows the prompt more literally, less variety
).images[0]
image.save("fox.png")
```

## Specific risks

- **Deepfakes and misinformation**: realistic images, voices, and video of real people.
- **Copyright**: models trained on content whose authors didn't consent.
- **Provenance**: to tell generated content apart, there are invisible watermarks (e.g. SynthID) and signed metadata (C2PA).

The general risks of AI systems are in [Risks and Mitigations](risks-and-mitigations.md).

---
Related: [From ML to Agentic AI](from-ml-to-agentic-ai.md), [Foundation Models](foundation-models.md), [What is a Token](what-is-a-token.md#multimodal--tokens-beyond-text), [Risks and Mitigations](risks-and-mitigations.md).
