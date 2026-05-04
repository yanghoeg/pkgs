# Talos kernel patches (yanghoeg fork)

Branch base:

- `release-1.12` / `release-1.12-r9700` — Talos 1.12.x base, **GCC** build, 6.18.24
- `release-1.13` / `release-1.13-r9700` — Talos 1.13.0 base, **LLVM (clang) + ThinLTO**, 6.18.24

Single vendor patch in this fork. PCI / BAR-related work is upstream
or in-flight on LKML — pull those yourself from the appropriate branch
(`pci-next`, `linus/master`, or Ilpo Järvinen's posted series) if your
hardware needs them.

## 1.12 → 1.13 toolchain notes

| 항목 | 1.12 | 1.13 |
|---|---|---|
| Toolchain | GCC | LLVM (clang) |
| TOOLCHAIN_MUSL_IMAGE | v1.12.0-10-gec7c6e8 | v1.13.0 |
| TOOLS_REV | v1.12.0-10-gbbd753d | v1.13.0 |
| LLVM_IMAGE | (없음) | ghcr.io/siderolabs/llvm |
| containerd | 2.1.7 | 2.2.3 |
| kernel/build/pkg.yaml LLVM env | (없음) | `LLVM: 1` |
| AMD GPU peer-to-peer DMA | disabled | **enabled** (559b1be) |
| dynamic SCS | enabled | disabled (ddd56d2) |

The single SMU patch is source-level diff so toolchain-agnostic. clang
strict warning 으로 build error 가능성은 빌드 시 확인.

## Patch

| Patch file | Description | Upstream | Status |
|---|---|---|---|
| `0001-amdgpu-smu14-0x33-driver-if-for-r9700.patch` | RDNA 4 R9700 SMU14_0_2 driver if 0x33. Idle power 350 W → 138 W per GPU | agd5f staging, mainline in progress | applied |

## Build

```sh
cd ~/code/siderolabs/talos
make kernel REGISTRY=<registry>:5000 USERNAME=yanghoeg PLATFORM=linux/amd64 PUSH=true JOBS=10
make installer REGISTRY=<registry>:5000 USERNAME=yanghoeg PLATFORM=linux/amd64 PUSH=true
```

Detailed setup: `bootstrap/talos/custom-kernel-build/{README,BUILDKIT}.md` (gitops repo).
