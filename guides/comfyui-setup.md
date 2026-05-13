# ComfyUI Setup: Image Generation on Dual GPUs

## Overview
ComfyUI is a node-based image generation interface for Stable Diffusion, Flux, and other models. This guide documents our setup for running it alongside Ollama on dual GPUs.

## Installation
We use a Python virtual environment to keep dependencies isolated:

```bash
cd ~/comfyui/ComfyUI
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

## Launch
When running alongside Ollama, pin ComfyUI to GPU 1:

```bash
cd ~/comfyui/ComfyUI
source venv/bin/activate
python main.py --device cuda:1
```

The `--device cuda:1` flag tells ComfyUI to use only GPU 1, leaving GPU 0 free for Ollama.

## Manager
ComfyUI Manager provides node installation and model management:

```bash
cd ~/comfyui/ComfyUI/custom_nodes/comfyui-manager
git pull
```

Manager is already installed and auto-updates with the ComfyUI repo.

## Models

### Checkpoints (SDXL)
Located at `models/checkpoints/`. Our working checkpoints:

- **realismByStableYogi_v5XL** — Best for photorealistic portraits. Requires SDXL workflow with CLIP and VAE loaders.
- **sdxlUnstableDiffusers_nihilmania** — Another photorealistic option, good for variety.
- **Flux Dev (flux1-dev-fp8)** — Conversational prompting, more forgiving with prompts but can drift toward anime style without steering.
- **Flux Schnell (flux1-schnell-fp8)** — Faster inference, good for quick iterations.

### Prompting Styles

#### SDXL (realismByStableYogi, nihilmania)
Comma-separated descriptors, shorter phrases, uses negative prompts heavily:

**Positive:**
```
portrait of a 26 year old woman of mixed Japanese and Korean heritage,
shoulder length dark brown hair, slightly messy natural hair texture,
soft warm skin tones, natural features, no heavy makeup,
calm dark eyes with quiet steady expression, faint subtle smile,
wearing dark oversized sweater, warm natural window lighting,
photorealistic portrait photograph, shallow depth of field,
high detail skin texture, realistic facial features
```

**Negative:**
```
anime, cartoon, illustration, drawing, painting, 3d render, doll,
plastic, glossy, oversaturated, blurry, low quality, distorted face,
deformed, extra fingers, bad anatomy, cartoon eyes, large eyes,
blushing, chibi, anime hair, wind swept hair, fantasy, costume
```

#### Flux Dev
Free-form, conversational prompts. More forgiving but needs steering away from anime style. Add "realistic photograph" at the start and "avoid anime style, avoid illustration" at the end.

## GPU Considerations
- **GPU 0**: Ollama (pinned via CUDA_VISIBLE_DEVICES=0)
- **GPU 1**: ComfyUI (via --device cuda:1)
- Each RTX 5060 Ti has 16GB VRAM
- SDXL checkpoints use ~6-8GB VRAM
- Flux Dev uses ~14GB VRAM
- Leave headroom for attention overhead and VAE decoding

## Workflow Nodes (SDXL)
Basic SDXL portrait workflow:
1. Checkpoint loader (realismByStableYogi_v5XLFP16.safetensors)
2. CLIP text encode (positive + negative prompts)
3. Empty latent image (1024x1024 for portraits)
4. KSampler (Euler, 20-30 steps, CFG 5-7)
5. VAE decode
6. Image save

## Notes
- SDXL needs CLIP and VAE loaders; Flux handles encoding internally
- Negative prompts matter a lot for SDXL — use them to steer away from unwanted styles
- Flux Dev can produce anime-style results without negative steering
- ComfyUI Manager helps install custom nodes and browse models
- Always use venv to avoid system Python conflicts
