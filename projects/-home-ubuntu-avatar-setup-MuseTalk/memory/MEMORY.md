Memories also committed to `documentation/memory/` (index: README.md there) in repo so they survive server/session loss.

- [Repo Setup](reference-upstream-remote.md) — Single repo: Panya-Labs/MuseTalk, origin direct, no upstream needed
- [Repo Layout](reference-repo-layout.md) — File paths, Docker container, key API facts
- [GPU Server Ops](reference-gpu-server-ops.md) — fetch-env.sh, start-musetalk.sh, /run/avatar.env, ECR lifecycle
- [Warm Start v2](project-warm-start-v2.md) — 69s→14.3s warm start, 774ms greeting, MERGED to main
- [Benchmark Project](project-musetalk-benchmark.md) — Optimization progress, baseline results, next tasks
- [Commit Workflow](feedback-commit-workflow.md) — Human approval required for every commit and push
- [Environment Quirks](project-env-quirks.md) — Python 3.12 patches: mmcv stubs, torch.load, version bumps
- [Cold Start Optimization](project-cold-start-optimization.md) — S3 trimming, burst credits, EBS vs NVMe, 15s Python floor, next: reduce Python startup
- [Perf Parallel Container](project-perf-parallel-container.md) — dev-hd box facts, perf_check.sh one-command run, 2026-09-18 cold-start + quality baseline measurements
