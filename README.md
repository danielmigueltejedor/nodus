# Nodus

**A modern, experimental, Rust-first kernel focused on safety, isolation, recoverability, and contemporary hardware.**

Nodus is an experimental operating-system kernel project exploring what a desktop-oriented kernel could look like if it were designed today without decades of legacy compatibility constraints.

The goal is **not** to rewrite Linux line-for-line in Rust, nor to compete with mature production kernels on hardware support. Nodus is a research and learning project intended to explore a smaller, safer and more modular architecture while eventually providing enough Linux ABI compatibility to run existing Linux applications.

> [!WARNING]
> Nodus is at the very beginning of development. It is not yet an operating system, is not suitable for production use, and currently makes no compatibility or stability guarantees.

## Why Nodus?

Modern general-purpose kernels carry decades of compatibility requirements. That compatibility is enormously valuable, but it also constrains architecture.

Nodus deliberately starts with different assumptions:

- modern 64-bit hardware only;
- Rust-first implementation;
- memory safety by default;
- minimal privileged kernel surface;
- strong isolation between components;
- capability-based access to resources;
- restartable userspace services and, where practical, drivers;
- fast typed IPC as a first-class primitive;
- multicore systems as the normal case;
- observability and tracing designed in from the beginning;
- Linux application compatibility as a compatibility layer rather than an internal architectural constraint.

Nodus is allowed to reject old hardware and old interfaces when doing so produces a cleaner design.

## Design principles

### 1. Safe Rust by default

Most kernel code should be safe Rust. `unsafe` is expected at the hardware boundary, but it should be small, explicit, documented and auditable.

### 2. Least privilege

Possessing a process identity should not implicitly grant broad access to the system. Resources should be exposed through explicit capabilities.

### 3. Failure isolation

A failure in a driver or service should not automatically become a kernel failure. Components should be restartable wherever the hardware and architecture allow it.

### 4. Fast IPC

A modular kernel is only practical if communication is cheap. IPC is therefore a core performance target, with zero-copy/shared-memory paths where appropriate.

### 5. Modern multicore first

The design should assume SMP, heterogeneous cores, deep cache hierarchies, modern interrupt controllers and contemporary power-management requirements from the beginning.

### 6. Observable by design

Scheduling, IPC, memory, driver and fault events should be traceable through structured diagnostics rather than reconstructed after the fact from ad-hoc logs.

### 7. Compatibility belongs at the edge

Nodus should not reproduce an old kernel architecture merely to expose an old API. Compatibility layers may translate established ABIs into native Nodus primitives.

## Initial architecture

The exact architecture will evolve as the project becomes real, but the current direction is a small microkernel-inspired or hybrid core.

```text
                         Applications
                              │
                 ┌────────────┴────────────┐
                 │  Native / Linux ABI     │
                 │  compatibility layers   │
                 └────────────┬────────────┘
                              │
                       Typed / async IPC
                              │
         ┌────────────────────┼────────────────────┐
         │                    │                    │
     Filesystems          Networking           Drivers
      services             services            services
         │                    │                    │
         └────────────────────┼────────────────────┘
                              │
════════════════════════════ NODUS ════════════════════════════
                              │
         ┌────────────┬──────────────┬──────────────┐
         │ Scheduler  │ Virtual mem  │     IPC      │
         ├────────────┼──────────────┼──────────────┤
         │ Capabilities             │ Interrupts    │
         ├──────────────────────────┼───────────────┤
         │ SMP / timers / arch / hardware boundary │
         └──────────────────────────────────────────┘
                              │
                           Hardware
```

This diagram is a direction, not a frozen specification.

## Target platforms

### Initial target

- **x86_64**
- **UEFI**
- **QEMU**
- modern PC-class hardware

QEMU is the first development platform because it gives Nodus a reproducible environment before real hardware support becomes a priority.

### Later targets

Potential future targets include:

- AArch64
- selected modern AMD64 machines
- selected VirtIO-capable virtual machines

Legacy BIOS, 32-bit x86 and obsolete hardware are intentionally not initial priorities.

## Linux application compatibility

Running Linux applications is a long-term goal.

This does **not** require Nodus to use the Linux kernel internally.

The intended direction is a compatibility subsystem capable of loading Linux ELF executables and translating supported Linux userspace ABI operations into native Nodus services.

Conceptually:

```text
Linux ELF application
        │
        ▼
Linux ABI compatibility layer
        │
        ├── syscall translation
        ├── process/thread semantics
        ├── virtual memory semantics
        ├── file-descriptor compatibility
        ├── signals
        ├── sockets
        └── /proc, /dev and related compatibility where required
        │
        ▼
Native Nodus primitives and services
```

Compatibility will be implemented incrementally and tested against real applications. Full Linux ABI compatibility is a major undertaking and is intentionally a later milestone.

The project does not initially aim to provide Linux kernel module or Linux driver ABI compatibility.

## Roadmap

The roadmap describes technical milestones, not release dates.

### Phase 0 — Foundations

- [ ] Write architecture decision records
- [ ] Define supported Rust toolchain and target specification
- [ ] Establish `no_std` kernel crate
- [ ] Set up reproducible builds
- [ ] Set up QEMU development workflow
- [ ] Set up serial logging
- [ ] Add CI for formatting, linting, tests and builds
- [ ] Document rules for `unsafe` code

**Exit condition:** a reproducible kernel image can boot under QEMU and print structured diagnostics.

### Phase 1 — Boot and machine bring-up

- [ ] UEFI boot path
- [ ] Parse firmware memory map
- [ ] Early physical memory management
- [ ] Page-table management
- [ ] Kernel virtual address space
- [ ] Kernel heap allocator
- [ ] Exception handling
- [ ] Interrupt handling
- [ ] APIC / timer support
- [ ] Panic and crash diagnostics

**Exit condition:** Nodus boots reliably, manages its own memory, handles exceptions and receives timer interrupts.

### Phase 2 — SMP and scheduling

- [ ] Bring up additional CPU cores
- [ ] Per-CPU state
- [ ] Kernel tasks
- [ ] Preemptive scheduler
- [ ] Synchronization primitives
- [ ] Timers and sleep
- [ ] Initial scheduler tracing
- [ ] Stress tests for concurrency

**Exit condition:** multiple tasks run preemptively across multiple virtual CPU cores.

### Phase 3 — Userspace

- [ ] Ring 3 / userspace execution
- [ ] Native executable format strategy
- [ ] Initial ELF loader
- [ ] Process address spaces
- [ ] Threads
- [ ] Native syscall entry
- [ ] Process lifecycle
- [ ] Basic exception delivery

**Exit condition:** Nodus can launch and isolate multiple userspace programs.

### Phase 4 — Capabilities and IPC

- [ ] Kernel object model
- [ ] Capability handles
- [ ] Capability transfer
- [ ] Synchronous IPC baseline
- [ ] Async/completion-based IPC
- [ ] Shared-memory transport
- [ ] Zero-copy paths where useful
- [ ] IPC benchmarking suite
- [ ] Permission and isolation tests

**Exit condition:** isolated userspace services can communicate without receiving ambient authority.

### Phase 5 — Userspace services and drivers

- [ ] Service manager
- [ ] Driver model
- [ ] Driver crash detection
- [ ] Driver/service restart infrastructure
- [ ] VirtIO block driver
- [ ] VirtIO network driver
- [ ] VirtIO input
- [ ] Basic framebuffer/display path
- [ ] Device discovery abstraction

**Exit condition:** useful I/O is provided by isolated services rather than hard-coded kernel subsystems where practical.

### Phase 6 — Storage and networking

- [ ] VFS design
- [ ] Initial native/simple filesystem support
- [ ] Persistent storage
- [ ] TCP/IP userspace service
- [ ] Sockets API
- [ ] DNS client
- [ ] Basic network tooling
- [ ] Filesystem and network fault recovery experiments

**Exit condition:** userspace programs can persist data and communicate over a network.

### Phase 7 — Native system environment

- [ ] Init/system service model
- [ ] Standard native runtime/library
- [ ] Device and service discovery APIs
- [ ] Time and randomness services
- [ ] Process inspection
- [ ] Structured tracing API
- [ ] Basic shell and utilities
- [ ] Package/application execution model experiments

**Exit condition:** Nodus can support a small self-hosted userspace environment.

### Phase 8 — Linux ABI compatibility

- [ ] Linux x86_64 ELF loading
- [ ] Linux syscall dispatch layer
- [ ] File descriptor semantics
- [ ] Linux virtual-memory operations
- [ ] Linux threads / futex primitives
- [ ] Signals
- [ ] Pipes and polling
- [ ] Unix sockets
- [ ] TCP/UDP socket compatibility
- [ ] Required `/proc`, `/dev` and pseudo-filesystem interfaces
- [ ] Dynamic linker / glibc compatibility investigation
- [ ] musl-based compatibility target
- [ ] Automated Linux ABI conformance tests

Progression target:

```text
static hello world
        ↓
simple musl binaries
        ↓
CLI applications
        ↓
complex dynamically linked applications
        ↓
graphical Linux applications
```

**Exit condition:** a meaningful subset of unmodified Linux x86_64 userspace binaries can execute on Nodus.

### Phase 9 — Modern desktop foundations

- [ ] Modern input stack
- [ ] Display/compositor interfaces
- [ ] Audio service
- [ ] Power management
- [ ] Suspend/resume experiments
- [ ] GPU userspace architecture research
- [ ] Desktop-oriented scheduling policies
- [ ] Low-latency / gaming policy experiments

**Exit condition:** enough native infrastructure exists to experiment with a real graphical desktop.

### Phase 10 — Real hardware

- [ ] Define first reference machine(s)
- [ ] ACPI support needed by selected hardware
- [ ] NVMe
- [ ] USB / xHCI
- [ ] PCIe device management
- [ ] Selected Ethernet hardware
- [ ] Selected Wi-Fi hardware
- [ ] Selected audio hardware
- [ ] IOMMU support
- [ ] Hardware-backed isolation

Hardware will be added deliberately. Supporting every old device is not a project goal.

### Phase 11 — Hardening and performance

- [ ] Formalize the threat model
- [ ] Fuzz syscall and IPC surfaces
- [ ] Fuzz parsers and executable loaders
- [ ] Audit all `unsafe` blocks
- [ ] Add kernel sanitization/debug configurations where possible
- [ ] Benchmark IPC latency
- [ ] Benchmark context switches
- [ ] Benchmark scheduler latency
- [ ] Benchmark memory allocation
- [ ] Benchmark networking
- [ ] Performance regression CI
- [ ] Fault-injection testing
- [ ] Recovery testing

## Research areas

Nodus is also a place to experiment. Areas of particular interest include:

- capability-oriented security;
- restartable userspace drivers;
- typed IPC;
- async kernel interfaces;
- zero-copy I/O;
- heterogeneous-core scheduling;
- low-latency desktop and gaming scheduling;
- transactional system updates;
- component-level recovery without rebooting;
- memory-safe driver APIs;
- tracing-first kernel architecture;
- Linux ABI compatibility without Linux internals;
- reducing the amount of code executing at the highest privilege level.

Experiments are allowed to fail. The project should prefer measured results over architectural dogma.

## Non-goals

At least initially, Nodus is **not** trying to:

- support every PC ever made;
- support 32-bit x86;
- support legacy BIOS;
- preserve historical kernel APIs for their own sake;
- be POSIX internally simply because Unix was;
- provide Linux kernel module compatibility;
- immediately replace Linux as the Nodalix production kernel;
- claim production security or reliability before those properties are demonstrated.

## Project philosophy

Nodus should be willing to ask:

> If we were designing this interface today, with modern hardware, Rust, multicore CPUs and decades of operating-system research available, would we still design it this way?

Compatibility is valuable, but it should live above clean native primitives wherever possible.

The first priority is to build something understandable, measurable and correct. Features come after foundations.

## Status

**Stage: pre-alpha / initial bring-up**

The immediate objective is intentionally small:

```text
QEMU
  ↓
UEFI
  ↓
Nodus
  ↓
serial output
  ↓
memory management
  ↓
interrupts
  ↓
scheduler
  ↓
userspace
  ↓
IPC
```

Everything after that grows from those foundations.

## Contributing

Nodus is currently an experimental project in its earliest design phase. Contribution guidelines will evolve once the architecture and development workflow stabilize.

For now, architectural discussions should explain the problem being solved, alternatives considered, expected trade-offs and how the result can be measured.

---

**Nodus** — experiment first, measure everything, keep the privileged core small.
