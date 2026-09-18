---
name: reference-repo-layout
description: Key file paths and run commands for Panya-Labs/MuseTalk (single production repo)
metadata: 
  node_type: memory
  type: reference
  modified: 2026-09-09T09:50:46.682Z
  originSessionId: dd202c88-6bff-4156-afc1-7c9d41394de8
---

## Repo Structure

```
/home/ubuntu/avatar-setup/
└── MuseTalk/                       # THE repo (Panya-Labs/MuseTalk, origin direct)
    ├── main_server.py              # Production WebRTC server (~5k lines, FROZEN — no new code)
    ├── musetalk/                   # Model code (mostly vendored upstream)
    │   ├── models/
    │   │   ├── vae.py              # VAE + decode_latents_tensor + TAESD
    │   │   └── unet.py             # UNet (safetensors preferred, .pth fallback)
    │   └── utils/
    │       ├── preprocessing.py    # Lazy DWPose/S3FD loading (our change)
    │       ├── blending.py         # get_image_prepare_material
    │       └── face_parsing/       # BiSeNet face parsing
    ├── documentation/
    │   ├── optimization-benchmarks/  # All benchmark results + analysis docs
    │   └── architecture/           # rendering.md, voice-and-protocol.md, operations.md
    ├── memory/                     # Session memory files (committed to repo)
    ├── STARTUP-PERF.md             # Warm/cold start analysis and plan
    ├── AGENTS.md                   # Repo orientation for AI agents
    ├── benchmark.py                # Production benchmark harness
    ├── tests/                      # test_relay_protocol.py, test_mic_gate.py, test_blank_avatar.py
    ├── docker-entrypoint.sh        # Page cache pre-warming, launch main_server.py
    ├── Dockerfile / Dockerfile.base # GPU image layers
    └── scripts/convert_unet_safetensors.py
```

## Docker Container (production)

```bash
docker exec musetalk <command>
docker logs musetalk

# Host-mounted volumes (persist across container recreation)
# /opt/models/musetalkV15/unet.safetensors (3.2GB)
# /opt/models/musetalkV15/unet.pth (3.2GB, fallback)
# /opt/models/sd-vae/ (320MB, .safetensors)
# /opt/models/whisper/ (145MB)
# /opt/models/dwpose/ (389MB)
# /opt/musetalk-data/video/ (avatar source videos)

# Container-internal cache (lost on container recreation, rebuilt on cold start)
# /app/results/v15/avatars/avator_1/runtime/ (geometry, idle, alpha, base_part)
```

## Key API Facts

- UNet input: latent [batch, 8, 32, 32], conditioning [batch, 50, 384]
- Single-step inpainting (timestep=0), NOT iterative diffusion
- Production blend: `(alpha * face_inter) + base_part` — numpy, NOT BiSeNet+morphology
- Production decode: `decode_latents_tensor()` keeps data on GPU as uint8 tensor
- Render service thread: pre-warmed at startup, all sessions share it via submit_render()
