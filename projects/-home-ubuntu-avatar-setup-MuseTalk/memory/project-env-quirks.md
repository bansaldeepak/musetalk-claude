---
name: project-env-quirks
description: Environment workarounds and patches applied for Python 3.12 + PyTorch 2.11 on this server
metadata: 
  node_type: memory
  type: project
  originSessionId: dd202c88-6bff-4156-afc1-7c9d41394de8
  modified: 2026-08-25T10:17:45.188Z
---

## Patches Applied (in MuseTalk working copy)

These workarounds were necessary because Python 3.12 + PyTorch 2.11 + CUDA 12.8 diverges from the README's Python 3.10 + torch 2.0.1 spec.

**Why:** Only Python 3.12 available on this server; the stock stack is incompatible.

**How to apply:** If the venv gets recreated or packages are upgraded, these patches may need to be reapplied.

### 1. torch.load weights_only=False
All `torch.load()` calls patched with `weights_only=False` (legacy .pth checkpoints contain numpy objects).
- `musetalk/utils/preprocessing.py` — added torch.serialization.add_safe_globals
- All other .py files — sed-patched torch.load calls

### 2. mmcv C++ ops stub
`.venv/lib/python3.12/site-packages/mmcv/ops/__init__.py` — replaced with dynamic placeholder that returns _Placeholder class for CamelCase names, _placeholder_fn for others. Prevents import crashes; actual ops raise NotImplementedError if called.

### 3. mmcv ext_loader patch
`.venv/lib/python3.12/site-packages/mmcv/utils/ext_loader.py` — `load_ext()` catches ImportError and returns namedtuple of stubs instead of crashing.

### 4. mmdet version check relaxed
`.venv/lib/python3.12/site-packages/mmdet/__init__.py` — `mmcv_maximum_version = '2.3.0'` (was '2.2.0', rejected mmcv 2.2.0).

### 5. mmengine checkpoint patch
`.venv/lib/python3.12/site-packages/mmengine/runner/checkpoint.py` — torch.load calls patched with weights_only=False.

### 6. setuptools downgraded
`setuptools<71` (70.3.0) to avoid pkg_resources removal in Python 3.12.

### 7. TensorFlow NOT installed
Incompatible with Python 3.12. Confirmed not needed for inference (validates TASK-004 premise).

### 8. transformers upgraded
From 4.39.2 → 5.15.1 (old version incompatible with huggingface_hub 1.28.0).
