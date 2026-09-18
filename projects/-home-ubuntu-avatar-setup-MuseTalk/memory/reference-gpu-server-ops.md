---
name: reference-gpu-server-ops
description: "GPU server ops: fetch-env.sh from SSM, start-musetalk.sh from ECR, /run/avatar.env, container lifecycle"
metadata: 
  node_type: memory
  type: reference
  originSessionId: dd202c88-6bff-4156-afc1-7c9d41394de8
  modified: 2026-09-02T07:36:13.604Z
---

## Container lifecycle

- `sudo /opt/gpu-server/scripts/fetch-env.sh` — fetches all params from AWS SSM `/${ENVIRONMENT}/` path, writes to `/run/avatar.env` (28 params including Cloudflare TURN, Azure OpenAI, Taiboli keys)
- `sudo /opt/gpu-server/scripts/fetch-env.sh --refresh` — fetches + restarts containers using the env file
- `sudo /opt/gpu-server/scripts/start-musetalk.sh --restart` — ECR login, pull latest `musetalk:dev`, recreate container with `--env-file /run/avatar.env`
- ECR image: `755348349838.dkr.ecr.us-east-1.amazonaws.com/musetalk:dev`

## Key details

- `/run/avatar.env` is tmpfs — wiped on reboot. Must run `fetch-env.sh` after every instance start.
- Container uses `--network host` (no port mapping needed)
- Volumes: `/opt/models:/app/models`, `/opt/musetalk-data/video:/app/data/video`
- `unet.safetensors` lives on host at `/opt/models/musetalkV15/unet.safetensors` — persists across container recreations
- Empty env file → ICE servers return `[]` → WebRTC "Could not reach connection service" error
- `docker restart` keeps env vars (baked at creation). Container recreation (`docker rm + run`) needs `--env-file` again.

## TF removal note (2026-09-02)

TF removed from requirements.txt and `TRANSFORMERS_NO_TF=1` set in Dockerfile, but TF 2.15.0 still installed in base image venv. Need explicit `pip uninstall` in Dockerfile or base image rebuild without TF.
