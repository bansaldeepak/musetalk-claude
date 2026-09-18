---
name: project-musetalk-benchmark
description: "MuseTalk performance benchmark project — environment, progress, and execution plan status"
metadata: 
  node_type: memory
  type: project
  originSessionId: dd202c88-6bff-4156-afc1-7c9d41394de8
  modified: 2026-08-31T16:49:25.843Z
---

## Project: MuseTalk Inference Pipeline Benchmarking & Optimization

Executing the plan in `/home/ubuntu/avatar-setup/musetalk-analysis/BENCHMARK-TASKS.md` (2558 lines).

**Repos:**
- `/home/ubuntu/avatar-setup/MuseTalk/` — working copy, branch `perf/optimization-benchmarks`. All code changes go here.
- `/home/ubuntu/avatar-setup/Taiboli-Musetalk/` — reference ONLY, NO edits.
- `/home/ubuntu/avatar-setup/musetalk-analysis/` — analysis docs, the BENCHMARK-TASKS.md plan lives here.

**Why:** Optimize the MuseTalk real-time lip-sync pipeline's inference performance on an L40S GPU.

**How to apply:** Every task produces a results markdown in `results/`, every commit needs human approval before push. Quality gate = PSNR/SSIM vs ground truth frames.

## Environment (established)

- GPU: NVIDIA L40S (44.4 GB VRAM), Driver 595.71
- Python 3.12.3, PyTorch 2.11.0+cu128, cuDNN 9.2.0
- mmcv 2.2.0 (lite, no C++ ops), mmpose 1.3.2, mmdet 3.3.0
- diffusers 0.30.2, transformers 5.15.1
- Venv at `.venv/`, activate with `source .venv/bin/activate`
- PYTHONPATH must include `musetalk/utils` for face_detection import

## Completed Tasks

### TASK-001 + TASK-000 (combined) — commit d080acb
- Environment setup on cu128 stack (Python 3.12 forced combining these)

### TASK-000B — commit 2a25504
- torch.compile mode="default": 1.59x UNet speedup

### TASK-002 — commit fde3418
- Benchmark harness, baseline: 10.9 FPS, noise floor: fully deterministic

### TASK-003 — commit 284fa3a
- UNet scales 13.4x at batch=16, VAE decode flat (memory-bound)

### TASK-004 — commit 4339f77
- Removed tensorflow/tensorboard from requirements.txt

### TASK-005 — commit 7aaab52
- Safetensors: model load 12.9s→6.2s (2.1x), bit-identical output

### TASK-008 — commit 84dfd50
- Mask caching: per-frame blending 49.7ms→15.9ms (3.1x)
- Persistent disk cache: 25.7s precompute→5ms warm load (5285x)

### TASK-009 — commit 3e12fa1
- S3FD downscaled to 0.5x: 43.6→12.7ms/frame (3.4x)
- Separated DWPose/S3FD passes: total face detect 17.5s→9.3s (1.87x)

### TASK-010 — pending commit
- Batch VAE encode (bs=2): 3.1s→2.1s cold (1.49x)
- Persistent latent cache: 9.6s→6ms warm (520x)
- Lazy model loading: DWPose/S3FD deferred, saves 5.4s on warm start
- Side effect: freed GPU memory speeds up UNet+VAE decode (42.5s→20.5s)
- Cumulative warm-start: 94.3s→62.8s (2.2x vs baseline 138s, 23.9 FPS)

## Key Findings

- **Blending IS the bottleneck** (22.5s = 35.9% of warm-start pipeline)
- **Persistent caching** eliminates coord detect + mask precompute + VAE encode on warm start (~55s→12ms)
- **GPU memory pressure matters**: not loading DWPose on warm start cut UNet+VAE decode by 50%
- **VAE encode resists batching**: memory-bandwidth-bound, batch=2 gives only 1.5x
- S3 bucket (taiboli-non-prod-ml-weights) is read-only — no write permission for cache upload

## What's Next

1. TASK-011B: Defer idle-frame precompute to background
2. TASK-005B/C/D/E: S3/EBS tasks (deferred — read-only S3 access)
3. Blending optimization (CPU morphology → GPU?)

## Git Status

Branch: `perf/optimization-benchmarks`
Latest pushed: `3e12fa1` (TASK-009)
TASK-010 changes ready, pending human approval for commit+push.
