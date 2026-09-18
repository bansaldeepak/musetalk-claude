---
name: cold-start-optimization
description: "Cold start optimization project — S3 download tuning, EBS vs NVMe benchmarks, network burst findings, 15s Python floor"
metadata: 
  node_type: memory
  type: project
  originSessionId: dd202c88-6bff-4156-afc1-7c9d41394de8
  modified: 2026-09-09T14:03:15.086Z
---

## Cold start optimization (branch: perf/s3-avatar-cache)

Full benchmarks documented in `documentation/optimization-benchmarks/COLD-START-OPTIMIZATION.md`.

### Key findings (2026-09-09)

**Data trimming:** Excluded 7.8 GB of unused models from S3 sync (v1 model, unet.pth superseded by safetensors, syncnet). Download reduced from 12 GB → 4.2 GB models + 5.9 GB avatar cache = 10.1 GB total.

**g6e.xlarge network burst:** "Up to 25 Gbps" is burstable — first ~3.5 GB at ~700 MB/s, then drops to ~270 MB/s baseline. This is WHY 10 GB always takes ~28s regardless of tuning. Not fixable without larger instance.

**s5cmd tuning:** `--numworkers 256 cp --concurrency 32 --part-size 32` gives best single-file speed (727 MB/s vs 432 MB/s default). Use `cp` not `sync` for cold start (sync adds 10-15s LIST overhead on empty dirs).

**EBS reads cap at 125 MB/s** regardless of parallelism. EBS→NVMe copy = 80s for 10 GB. S3→NVMe = 28s (faster to go via S3 than local EBS!).

### Container start times

| Scenario | Time |
|----------|------|
| NVMe warm restart | **20s** |
| EBS warm page cache restart | **28s** |
| EBS cold page cache (first boot) | **49s** |
| S3→NVMe + start | **48s** |

### Python startup floor: 15s

5s container + 4s imports + 7s model load (safetensors→GPU) + 0.5s cache + 3s GPU warmup. This is the NEXT optimization target. Ideas: lazy model load, persistent process, torch.compile caching, quantization.

**Why:** User wants cold + warm start both under 10s to potentially eliminate EBS.

**How to apply:** Any future work on startup time should read the benchmark doc first. The 15s Python floor and 28s network floor are hard limits at current instance size. Reducing data size (especially the two 2.7 GB numpy frame arrays) would help most.

### Branch state

3 commits on `perf/s3-avatar-cache`, pushed. ECR image not yet rebuilt — current dev image predates s5cmd install and entrypoint changes. VPC Gateway Endpoint created but not verified routing.

### Related: [[project-musetalk-benchmark]], [[reference-gpu-server-ops]]
