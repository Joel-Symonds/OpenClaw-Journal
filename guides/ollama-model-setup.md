# Ollama Model Setup: qwen3.6:27b

## Model Details
- **Name**: qwen3.6:27b
- **Architecture**: qwen3.5
- **Parameters**: 27.8B
- **Quantization**: Q4_K_M (4-bit)
- **Model size**: 28GB on disk, ~17GB VRAM used

## Context Window
- **Maximum capability**: 256K tokens
- **Actual loaded**: 128K tokens (auto-capped by Ollama based on VRAM)
- **Why 128K?**: Ollama automatically limits context length based on available VRAM to prevent OOM errors
- **Practical impact**: Most conversations never exceed 10-20K tokens, so 128K is more than enough

## VRAM Usage
- **Dual GPU setup**: RTX 5060 Ti 16GB x 2
- **Per GPU usage**: ~14GB VRAM each when using both GPUs
- **Headroom**: ~2GB per GPU for other tasks
- **Quantization sweet spot**: Q4_K_M gives good quality without eating all VRAM

## Configuration
Ollama auto-tunes context length based on VRAM. No manual config needed in most cases.

To manually set context length (if needed), add to the Ollama service override:

```
Environment="OLLAMA_CONTEXT_LENGTH=131072"
```

Then reload and restart:

```
sudo systemctl daemon-reload
sudo systemctl restart ollama
```

## Performance
- **Single GPU**: ~3 tokens/sec (pinned for ComfyUI isolation)
- **Dual GPU**: 15+ tokens/sec (normal mode)
- **Context impact**: 128K context uses ~17GB VRAM total

## Notes
- The model supports up to 256K context, but Ollama caps it based on VRAM
- 128K is more than enough for typical conversations
- Q4_K_M quantization is the sweet spot for 27B models on 16GB GPUs
- If you need faster inference, consider Q4_0 (lower quality, less VRAM)
- If you need better quality, consider Q5_K_M (higher quality, more VRAM)
