# Talos custom kernel patches (yanghoeg fork)

브랜치별 base:

- `release-1.12` / `release-1.12-r6drmnext` — Talos 1.12.x base, **GCC** 빌드, 6.18.24
- `release-1.13` / `release-1.13-r6drmnext` — Talos 1.13.0 base, **LLVM (clang) + ThinLTO**, 6.18.24

ASUS X99-E WS workstation 의 R9700 RDNA4 dual GPU 32GB ReBAR + PCI bridge resource
fix 를 위한 v6.18 port patch 모음. v6.19 / pci-v7.0-changes 에 들어간 또는 진행 중인
upstream patch 를 6.18.24 stable 로 backport.

## 1.12 → 1.13 전환 시 변경사항

| 항목 | 1.12 (R6) | 1.13 |
|---|---|---|
| Toolchain | GCC | LLVM (clang) |
| TOOLCHAIN_MUSL_IMAGE | v1.12.0-10-gec7c6e8 | v1.13.0 |
| TOOLS_REV | v1.12.0-10-gbbd753d | v1.13.0 |
| LLVM_IMAGE | (없음) | ghcr.io/siderolabs/llvm |
| containerd | 2.1.7 | 2.2.3 |
| kernel/build/pkg.yaml LLVM env | (없음) | `LLVM: 1` |
| make jobs | `make -j 8` (R6 throttle) | `make -j $(nproc)` upstream → R6 cherry-pick 시 8 으로 회귀 |
| AMD GPU peer-to-peer DMA | disabled | **enabled** (559b1be) |
| dynamic SCS | enabled | disabled (ddd56d2) |

**LLVM 빌드 영향 가설**: 0001~0004 patch 자체는 source-level diff 라 toolchain 무관 적용.
단 clang 의 strict warning 으로 build error 가능성 존재 — 빌드 시도 후 확인 필요.

| Patch file | Description | Upstream | Status |
|---|---|---|---|
| `0001-PCI-prevent-shrink-bridge-window-v618-port.patch` | `dc4b4d04e1ca` ("PCI: Prevent shrinking bridge window from its required size", Ilpo Järvinen) v6.18 port. `pci_bridge_distribute_available_resources()` → `adjust_bridge_window()` shrink 방지 | v6.19 / pci-v7.0-changes merged | applied |
| `0002-amdgpu-smu14-0x33-driver-if-for-r9700.patch` | RDNA4 R9700 SMU14_0_2 driver if 0x33 (idle 350W → 138W). agd5f tree backport | agd5f staging, mainline 진행 중 | applied |
| `0003-PCI-fix-premature-removal-realloc-head-v618.patch` | "PCI: fix premature removal from realloc head" v6.18 port. `pci_bus_distribute_available_resources()` realloc list 처리 fix | v6.19 / pci-next | applied |
| `0004-PCI-try-bar-resize-without-window-release-v618.patch` | Ilpo v2 04/11 "PCI: Try BAR resize even when no window was released" v6.18 port. `pci_resize_resource()` 안 `if (list_empty(&saved))` → `if (!bridge)`. **placeholder hunk** — 빌드 직전 v6.18.24 source 의 line offset 채워야 함 | v2 series LKML 2025-11-13, 6.19 target | placeholder |

## Build target image tags

### 1.12 base (production fleet 현재 운영)

| Image tag | Patches | 용도 |
|---|---|---|
| `v1.12.7-rdna4-c1-smuonly-r3-amd64` | 0002 SMU only | Phase 2a 운영 (idle 138W) |
| `v1.12.7-rdna4-c1-smuonly-r4-amd64` | R3 + factory cmdline (`pci=realloc`, mlx5 blacklist) | 검증된 cmdline baseline |
| `v1.12.7-rdna4-c1-smuonly-r5-pci04-amd64` | R4 + 0001 + 0003 + 0004 | Phase 2b dual-GPU 32GB target |
| `kernel:v1.3-r6drmnext-amd64` | R5 + drm-next 73ec35caf | **현재 운영** (small BAR 위 32GB 인식) |

### 1.13 base (검증 진행 예정)

| Image tag (예정) | Patches | 용도 |
|---|---|---|
| `v1.13.0-rdna4-r6-llvm-amd64` | R6 patches (cherry-picked) on 1.13 LLVM toolchain | Talos 1.13 + ROCm 7.2.2 검증 |

## 빌드

```sh
cd ~/code/siderolabs/talos
make kernel REGISTRY=<registry>:5000 USERNAME=yanghoeg PLATFORM=linux/amd64 PUSH=true JOBS=10
make installer REGISTRY=<registry>:5000 USERNAME=yanghoeg PLATFORM=linux/amd64 PUSH=true
```

자세한 환경 setup 은 `bootstrap/talos/custom-kernel-build/{README,BUILDKIT}.md` (gitops repo).
