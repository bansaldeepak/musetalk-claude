---
name: reference-upstream-remote
description: "Panya-Labs/MuseTalk is the single working repo — origin direct, deployed to ECR"
metadata: 
  node_type: memory
  type: reference
  modified: 2026-09-09T09:50:35.820Z
  originSessionId: dd202c88-6bff-4156-afc1-7c9d41394de8
---

## Repo consolidation (2026-09-09)

Previously there were two repos: `bansaldeepak/Realtime-Avatar` (clone) and `Panya-Labs/MuseTalk` (origin).
Now consolidated to a **single repo**: `Panya-Labs/MuseTalk`.

- Working directory: `/home/ubuntu/avatar-setup/MuseTalk`
- Origin: `https://github.com/Panya-Labs/MuseTalk.git`
- No upstream remote needed — origin IS the deployed repo
- `bansaldeepak/Realtime-Avatar` is legacy — do not push there

Standard workflow:
```bash
git checkout -b feat/my-branch
# ... work ...
git push origin feat/my-branch
gh pr create --base main --head feat/my-branch
```

Private repo — fork setting was off on GitHub.
