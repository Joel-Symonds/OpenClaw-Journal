# GPU Split: Ollama + ComfyUI on Dual GPUs

## Problem
Running Ollama (LLM inference) and ComfyUI (image generation) on the same machine with dual GPUs. Both apps try to use all available GPUs, causing:
- **VRAM conflicts** when both try to load models simultaneously
- **Performance drop** when Ollama is pinned to one GPU (27B model runs ~3 tokens/sec on single GPU vs snappy on two)

## Solution
Pin each app to its own GPU using environment variables and launch flags.

## Trade-off
- **Isolation**: Ollama pinned to GPU 0, ComfyUI on GPU 1 → no VRAM conflicts, but Ollama runs slower
- **Performance**: Ollama on both GPUs → fast, but ComfyUI can't run without VRAM fighting
- **Best of both**: Pin when running ComfyUI, unpin when not

## Setup

### Ollama (GPU 0) - When Running ComfyUI
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

### Normal Mode (Both GPUs for Ollama)
Remove the `CUDA_VISIBLE_DEVICES` override:

```bash
sudo tee /etc/systemd/system/ollama.service.d/override.conf << 'INNEREOF'
# Other Ollama settings remain
INNEREOF

sudo systemctl daemon-reload
sudo systemctl restart ollama
```

## Performance Comparison
- **Single GPU (pinned)**: ~3 tokens/sec (acceptable for isolation)
- **Dual GPU (normal)**: snappy, responsive (15+ tokens/sec)
- **Recommendation**: Pin only when running ComfyUI, otherwise let Ollama use both

## Result
- **GPU 0**: Ollama running qwen3.6:27b (~14GB VRAM used)
- **GPU 1**: ComfyUI free for image generation (~14GB VRAM available)
- No VRAM conflicts when pinned

## Notes
- Each GPU gets ~16GB VRAM (RTX 5060 Ti in this setup)
- Ollama's 27B model uses ~14GB on a single GPU, but benefits from two
- ComfyUI can load SDXL checkpoints (~6-8GB) without touching Ollama's space when pinned
- The pinning is temporary — remove it when you're done with ComfyUI

## Verification
Check GPU usage:
```bash
nvidia-smi --query-gpu=index,temperature.gpu,memory.used,memory.total --format=csv
```

You should see both GPUs active with separate memory usage.
