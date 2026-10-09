# AGENTS.md

This repository is a **research and documentation lab** on Linux process
performance and virtualization — not a software project. There is no build,
test, lint, or CI. Currently only `README.md` and `kb/context.md` exist.

## Current scope

- Compare execution models for a **generic CPU-bound process with a hard
  per-invocation time budget**: native on shared cores, native on
  dedicated/isolated cores, container (default and latency-tuned), and VM
  (pinned vCPU).
- The study targets **generic Linux**, not any specific distribution or
  enterprise release, and is not tied to a domain workload. The trading framing
  that appeared in `README.md` and `kb/context.md` was the original motivation
  and is being removed — do not reintroduce it or RHEL-specific framing.
- Because there is no real workload, **define and commit to one concrete anchor
  benchmark before measuring**. A generic study with no fixed workload and no
  defined metric is unfalsifiable.

## Measurement discipline (do not drop these)

- Define the measured interval, start/end points, and clock source for every
  experiment.
- Report latency/jitter **distributions** (p50/p99/p99.9/max), not means;
  include throughput; state sample size, duration, and warm-up.
- Separate facts, assumptions, hypotheses, and measurements.

## Read before producing content

- `README.md` — mission, methodology, experiment-record template, working
  principles.
- `kb/context.md` — acting persona and output expectations.

## Host environment

- Experiments run on this **Fedora 44** ThinkPad (AMD Ryzen 5 PRO 4650U,
  6C/12T, ~7 GiB RAM). Results are host- and kernel-specific; qualify them
  rather than presenting them as general Linux behavior, and record the kernel
  version.
- Machine-wide hardware/power facts live in `~/.config/opencode/AGENTS.md`; do
  not duplicate them. Verify volatile state (kernel, free space, services)
  rather than trusting it.

## Local virtualization tooling (verify before relying)

- KVM available: `/dev/kvm` present, `kvm_amd nested=1`.
- Present: `podman` (5.x), `qemu-system-x86_64`, `virsh`.
- Absent: `docker`, `virt-install`, `perf`, `numactl`, `cyclictest`.
- Installing tools needs root; show the user the command rather than running
  `sudo` yourself.

## Repository shape

- The layout in README §"Suggested repository structure" is aspirational; none
  of those directories exist yet. Create them as experiments land.
- Keep experiments small, independent, and reproducible.
