# Linux Process Performance & Virtualization

> A hands-on research lab for understanding Linux execution and measuring the performance implications of containers and virtual machines for latency-sensitive CPU-bound workloads.

## Mission

How do native processes, containers, and virtual machines compare in execution
performance for a CPU-bound process with a hard per-invocation time budget?

This repository investigates that question from first principles: Linux process
execution, kernel behavior, CPU and memory architecture, isolation mechanisms,
virtualization, and measurement methodology.

There is no single domain workload. The study therefore defines one concrete
**anchor workload** — a fixed-work, CPU-bound task with a bounded execution-time
budget — and measures how each execution model affects its latency distribution
and jitter.

**The goal is not to prove that virtualization is good or bad. It is to
determine which execution model is appropriate for which workload, under which
conditions, and supported by what evidence.**

## Core questions

- What determines the execution time and variability of a native Linux process?
- How do scheduling, context switches, CPU migration, interrupts, cache locality, NUMA, memory allocation, and page faults affect latency?
- Which Linux mechanisms do containers use, and which affect the application's execution path?
- What additional mechanisms are introduced by a virtual machine and hypervisor?
- Under controlled conditions, how do native processes, containers, and VMs compare in latency, jitter, throughput, and resource use?
- Which optimizations—CPU affinity, CPU isolation, NUMA placement, huge pages, locked memory, interrupt affinity, or busy polling—help a given workload, and what trade-offs do they introduce?
- How do shared versus dedicated/isolated CPU cores change the execution-time distribution and jitter of the anchor workload?
- What can benchmark results establish, and what uncertainty remains?

## Scope

### 1. Linux execution fundamentals

Processes and threads; user mode and kernel mode; system calls; scheduling; context switches; CPU affinity and migration; virtual memory; page faults; TLBs; CPU caches; synchronization; interrupts; NUMA; memory allocation.

### 2. Performance engineering

CPU topology and SMT; frequency scaling and power states; isolated CPUs; scheduling policies; memory locking; huge pages; interrupt distribution; busy polling; kernel and networking behavior; observability with tools such as `perf`, `taskset`, `chrt`, `numactl`, `/proc`, and `/sys`.

### 3. Containers

Linux namespaces and cgroups; capabilities and seccomp; container runtimes; CPU and memory controls; filesystem isolation; networking; image and runtime overhead; the distinction between container execution and orchestration platforms.

### 4. Virtual machines

Hypervisors and hardware-assisted virtualization; KVM; guest and host scheduling; vCPUs; memory translation; virtual interrupts; virtual I/O; device passthrough; CPU and NUMA placement; isolation and operational trade-offs.

### 5. Distribution and version differences

Kernel versions, distribution defaults, cgroup versions, container runtimes, and tooling availability differ across Linux systems. General upstream Linux functionality must be distinguished from behavior specific to a particular kernel or distribution version. Record the kernel and distribution version for every experiment.

### 6. Workloads and architecture

Single-threaded and multi-threaded execution; CPU-bound versus I/O-bound behavior; inter-process communication; network paths; logging, monitoring, and supporting services. Evaluate components independently rather than assuming the entire system has one latency requirement.

## A crucial distinction: execution time of what?

Before interpreting any result, define the measured interval.

- **Process execution time:** time spent performing the workload's computation.
- **Application-path latency:** time across a defined sequence of application operations, potentially including queues and synchronization.
- **Request-to-response latency:** time from receiving an input to producing an output.
- **End-to-end latency:** time across the full pipeline, including the components explicitly included in that measurement.

These are different metrics. Every experiment should state its start and end points, clock and timing method, included operations, and whether networking or queueing is part of the result.

## Experimental methodology

Comparisons should use equivalent workloads and, where possible, the same physical host. Keep application binaries, input data, compiler options, workload intensity, and measurement methods consistent. Record every intentional difference between configurations.

Before comparing execution models, fix a single **anchor workload**: a fixed-work, CPU-bound task with a bounded per-invocation execution-time budget. Record its exact work unit so every configuration runs identical work.

Compare these execution models where the environment allows:

1. Native process on bare metal.
2. Container with default resource settings.
3. Container with latency-oriented CPU and memory placement.
4. VM with a baseline configuration.
5. VM with relevant CPU, memory, and device optimizations.
6. A hybrid architecture that assigns different components to different execution models.

Collect metrics appropriate to the experiment:

- Latency distribution: p50, p90, p99, p99.9, and maximum observed latency.
- Throughput and CPU utilization.
- Context switches, CPU migrations, page faults, and other relevant counters.
- Hardware, firmware, BIOS settings, kernel, distribution version, runtime, hypervisor, and configuration details.
- Number of observations, test duration, workload shape, warm-up procedure, and environmental conditions.

Use histograms or quantile summaries rather than relying on a single average. Report sample size and test duration. Treat the maximum as the largest value observed during a specific test—not as proof of a hard upper bound. Avoid attributing a performance difference to containerization or virtualization until other plausible causes have been examined.

## Suggested repository structure

```text
.
├── README.md
├── docs/
│   ├── linux-execution.md
│   ├── cpu-memory-and-numa.md
│   ├── containers.md
│   ├── virtual-machines.md
│   ├── latency-measurement.md
│   ├── distribution-and-versions.md
│   └── architecture-decisions.md
├── experiments/
│   ├── 01-native-baseline/
│   ├── 02-scheduling-and-affinity/
│   ├── 03-memory-and-numa/
│   ├── 04-container-comparison/
│   ├── 05-vm-comparison/
│   └── 06-networking-and-ipc/
├── benchmarks/
│   ├── workloads/
│   ├── scripts/
│   └── results/
├── references/
│   └── bibliography.md
└── reports/
    └── findings-template.md
```

This is a suggested layout, not a requirement to create every directory immediately. Keep experiments small and independently reproducible; add structure as the investigation grows.

## Experiment record template

Each experiment should document:

1. **Question:** What are we trying to learn?
2. **Hypothesis:** What do we expect, and why?
3. **Environment:** Hardware, firmware, kernel/distribution version, software versions, and privileges.
4. **Workload:** Program, input, compiler/build options, workload intensity, and warm-up.
5. **Configuration:** Exact commands and settings, including CPU affinity, cgroups, NUMA placement, and virtualization settings.
6. **Measurement:** Timing boundaries, clock source, tools, sample count, duration, and reported statistics.
7. **Results:** Raw data or a link to it, summaries, plots where useful, and anomalies.
8. **Interpretation:** What the evidence supports and what it does not establish.
9. **Reproduction:** Steps required for another person to rerun the test.
10. **Limitations and next steps:** Confounding factors, unresolved questions, and the next experiment.

Never publish results without enough context for someone else to understand how they were produced.

## Working principles

- Start with mechanisms and build toward architectural conclusions.
- Separate facts, assumptions, hypotheses, and measurements.
- Prefer primary documentation, kernel documentation, source code, peer-reviewed research, and reproducible experiments.
- Check the documentation for the exact kernel and distribution version, and the relevant runtime or hypervisor versions.
- Separate isolation overhead from contention, configuration, networking, orchestration, and operational effects.
- Treat latency, tail latency, throughput, resource efficiency, security, isolation, and maintainability as separate dimensions.
- Make conclusions conditional on the workload and configuration tested.
- Treat bare metal as a baseline to measure, not as an automatic guarantee of deterministic performance.
- Treat containers and VMs as distinct technologies with different mechanisms and trade-offs.
- Do not extrapolate from a synthetic microbenchmark to production performance without explaining the limits.

## Learning path

| Phase | Focus | Exit criterion |
|---|---|---|
| 1 | Native Linux execution | Explain and observe scheduling, memory, and CPU behavior in a native process. |
| 2 | Measurement foundations | Build a repeatable latency benchmark and report distributions responsibly. |
| 3 | CPU and memory tuning | Measure the effects of affinity, migration, NUMA, and relevant memory behavior. |
| 4 | Containers | Explain the isolation mechanisms and compare a container against a native baseline. |
| 5 | Virtual machines | Explain the guest/host execution path and compare a VM against the same baseline. |
| 6 | Workload-level testing | Include realistic IPC and networking where relevant to the target system. |
| 7 | Architecture assessment | Recommend native, containerized, virtualized, or hybrid placement by component, with evidence and caveats. |

## References

Start with primary sources and record the version consulted:

- [Linux kernel documentation](https://docs.kernel.org/)
- [Linux man-pages project](https://man7.org/linux/man-pages/)
- [perf documentation](https://perf.wiki.kernel.org/)
- [Open Container Initiative (OCI) specifications](https://opencontainers.org/)
- [KVM documentation](https://www.linux-kvm.org/page/Documents)

Prefer primary upstream documentation for mechanisms. Where behavior is version-specific, record the exact kernel and distribution version consulted.

Maintain a source list in [`references/bibliography.md`](references/bibliography.md), including title, author or organization, version/date, URL, and the claim or experiment it informs.

## Definition of done

The investigation is successful when it produces:

- A clear mental model of native Linux execution and the relevant hardware interactions.
- Reproducible comparisons of native processes, containers, and VMs under documented conditions.
- An explanation of observed latency and tail behavior, including limitations and confounding factors.
- A workload-specific assessment of where each execution model is appropriate.
- An evidence-based recommendation, including unresolved risks and the additional measurements needed before a production migration.

**Final principle:** measure the workload that matters, understand the mechanisms behind the measurements, and make architectural decisions from evidence rather than general claims.
