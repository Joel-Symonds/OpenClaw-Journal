# GPU Split: Ollama + ComfyUI on Dual GPUs

## Problem
Running Ollama (LLM inference) and ComfyUI (image generation) on the same machine with dual GPUs. Both want to use all available VRAM, causing conflicts and slowdowns.

## Solution
Pin each app to its own GPU using environment variables and launch flags.

## Setup

### Ollama (GPU 0)
Add `CUDA_VISIBLE_DEVICES=0` to the Ollama service override:

```bash
sudo tee -a /etc/systemd/system/ollama.service.d/override.conf << 'INNEREOF'
Environment="CUDA_VISIBLE_DEVICES=0"
INNEREOF

sudo systemctl daemon-reload
sudo systemctl restart ollama
```

This locks Ollama to GPU 0 only. It can't see GPU 1.

### ComfyUI (GPU 1)
Launch ComfyUI with the `--device cuda:1` flag:

```bash
cd ~/comfyui/ComfyUI
source venv/bin/activate
python main.py --device cuda:1
```

This tells ComfyUI to use GPU 1 only.

## Result
- **GPU 0**: Ollama running qwen3.6:27b (~14GB VRAM used)
- **GPU 1**: ComfyUI free for image generation (~14GB VRAM available)
- No VRAM conflicts, no offloading needed, no model shuffling

## Notes
- Each GPU gets ~16GB VRAM (RTX 5060 Ti in this setup)
- Ollama's 27B model uses ~14GB, leaving headroom
- ComfyUI can load SDXL checkpoints (~6-8GB) without touching Ollama's space
- If you have more GPUs, adjust the device numbers accordingly

## Verification
Check GPU usage:
```bash
nvidia-smi --query-gpu=index,temperature.gpu,memory.used,memory.total --format=csv
```

You should see both GPUs active with separate memory usage.
