---
name: project-warm-start-v2
description: "Warm start optimization results — 69s→14.3s, greeting 1610ms→774ms, merged to Panya-Labs/MuseTalk main"
metadata: 
  node_type: memory
  type: project
  modified: 2026-09-09T09:51:11.573Z
  originSessionId: dd202c88-6bff-4156-afc1-7c9d41394de8
---

## Warm Start Optimization v2 — MERGED

All work merged to `Panya-Labs/MuseTalk` main as squash commit `4ef5581` (PR #38).
Docs/memory merged as `6bf9da6` (PR #39).

### Results (validated with live Docker restarts + WebRTC sessions)

- **Warm start (page cache warm):** 69s → 14.3s (4.8x)
- **Warm start (page cache cold):** ~37s (EBS throughput bottleneck)
- **Per-session greeting:** 1610ms → 774ms (2.1x) via render service thread
- **Cold start (no avatar cache):** ~7m38s

### Key technical findings

- cuDNN benchmark plans are **per-thread** (~450ms/shape autotuning per new thread)
- Persistent render service thread eliminates per-session warmup
- base_part_inter cached as uint8 .npy (279MB), max pixel error ±0.5
- UNet safetensors file on host at `/opt/models/musetalkV15/unet.safetensors`
- TF removal in requirements.txt but still installed in base image — needs base image tag fix (user handling)

### Remaining warm start bottlenecks

| Phase | Time | Next step |
|-------|------|-----------|
| Python imports (TF present) | ~4s | TF removal in base image (~2s saved) |
| Model load (UNet + VAE + Whisper) | ~6.7s | EBS provisioning / persistent process |
| Runtime geometry | ~0.5s | Already optimized |
| Warmup (cuDNN) | ~3.2s | Persistent process (plans lost on restart) |
| **Total** | **~14.3s** | **Target: 2-3s** |

**Why:** Fast startup = better developer iteration, faster scaling, better student first impression.
**How to apply:** See STARTUP-PERF.md for detailed next steps. Each optimization should be a separate branch — merge if results are good, discard if bad.
