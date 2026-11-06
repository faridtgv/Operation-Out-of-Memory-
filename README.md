# MiniMax H3 out of memory: 7 fixes (ComfyUI workflows)

These are the ComfyUI workflows from the Beyond the Frame video **"Operation: Out of Memory"**. Each workflow is one MiniMax H3 graph with the video's fixes built in. Every fix is a coloured group, and the red nodes are the changes.

| Workflow | Use it when |
|---|---|
| `OOM_H3_seven_fixes_ClipProj.json` | You run out of memory before the first frame. A 4B encoder plus a projection replaces the 32B encoder (15.7 GB → about 5–8 GB). |
| `OOM_H3_seven_fixes_stock_CLIP.json` | The stock 32B encoder fits on your card, or you don't want to install ClipProj. |

## What's inside

| Chapter | Problem | Fix | Where |
|---|---|---|---|
| 01 · The buy-in | Out of memory before the first frame | ClipProj Loader in place of CLIPLoader | node |
| 02 · The tasting | Flicker or black frames | Pruned model, not full INT8 / BF16 | node |
| 03 · The engine | Suddenly 4× slower after an update | Note down your PyTorch/CUDA versions and roll back if an update slows you down | note |
| 04 · The warehouse | Out of memory with memory to spare | `--disable-dynamic-vram` | note |
| 05 · The bad line | Crackling audio | Update ComfyUI, plain euler sampler, no EasyCache | node + note |
| 06 · The disguise | Reference drift (second watch, wrong tie) | A dossier prompt: describe it precisely and repeat it | prompt |
| 07 · The package | Crash during VAE Decode | VRAM-Cleanup (unload all models) right before decode, 209 frames | node |

## Custom nodes

- **ComfyUI-ClipProj** (ClipProj workflow only): https://github.com/nicolab28/ComfyUI-ClipProj
  - Text encoder, any Qwen3-VL-4B → `models/text_encoders`: `qwen3vl_4b_fp8_scaled.safetensors` (5.2 GB) or `qwen3vl_4b_bf16.safetensors` (8.9 GB), https://huggingface.co/Comfy-Org/Krea-2/tree/main/text_encoders
  - Projection → `models/clip_projections`: https://huggingface.co/NicoLab28/ClipProj-MiniMax-H3
- **Comfyui-Memory_Cleanup** (VRAM-Cleanup node): https://github.com/LAOGOU-666/Comfyui-Memory_Cleanup
- **ComfyUI-VideoHelperSuite** (Video Combine, saves the mp4)

Both workflows also need the MiniMax H3 core nodes (built into current ComfyUI), the pruned diffusion model, and the video and audio VAEs: https://huggingface.co/Comfy-Org/MiniMax-H3

## Before you run

1. In the loaders, re-select your own model, encoder, projection and VAE files.
2. In the ClipProj Loader, set `device` to your GPU.
3. The test size is 608×352. Raise it for final renders.
4. Frame counts must fit `n % 17 == 5` (5, 22 … 124 ≈ 5 s … 209 ≈ 8.7 s … 243 ≈ 10 s). On 16 GB, crashes started above ~209 frames, so render short clips and join them in the edit.
5. Tiled VAE Decode does **not** help with H3, because the H3 VAE already tiles internally. Use the VRAM-Cleanup node.

`make_oom_workflows.py` regenerates both JSON files.

Numbers and fixes reflect the reports as of November 2026. ComfyUI changes fast, so check the linked GitHub threads in the video description.
