# Talos custom kernel patches (yanghoeg fork, release-1.12 base = 6.18.24)

ASUS X99-E WS workstation 의 R9700 RDNA4 dual GPU 32GB ReBAR + PCI bridge resource
fix 를 위한 v6.18 port patch 모음. v6.19 / pci-v7.0-changes 에 들어간 또는 진행 중인
upstream patch 를 6.18.24 stable 로 backport.

| Patch file | Description | Upstream | Status |
|---|---|---|---|
| `0001-PCI-prevent-shrink-bridge-window-v618-port.patch` | `dc4b4d04e1ca` ("PCI: Prevent shrinking bridge window from its required size", Ilpo Järvinen) v6.18 port. `pci_bridge_distribute_available_resources()` → `adjust_bridge_window()` shrink 방지 | v6.19 / pci-v7.0-changes merged | applied |
| `0002-amdgpu-smu14-0x33-driver-if-for-r9700.patch` | RDNA4 R9700 SMU14_0_2 driver if 0x33 (idle 350W → 138W). agd5f tree backport | agd5f staging, mainline 진행 중 | applied |
| `0003-PCI-fix-premature-removal-realloc-head-v618.patch` | "PCI: fix premature removal from realloc head" v6.18 port. `pci_bus_distribute_available_resources()` realloc list 처리 fix | v6.19 / pci-next | applied |
| `0004-PCI-try-bar-resize-without-window-release-v618.patch` | Ilpo v2 04/11 "PCI: Try BAR resize even when no window was released" v6.18 port. `pci_resize_resource()` 안 `if (list_empty(&saved))` → `if (!bridge)`. **placeholder hunk** — 빌드 직전 v6.18.24 source 의 line offset 채워야 함 | v2 series LKML 2025-11-13, 6.19 target | placeholder |

## Build target image tags

| Image tag | Patches | 용도 |
|---|---|---|
| `v1.12.7-rdna4-c1-smuonly-r3-amd64` | 0002 SMU only | Phase 2a 운영 (idle 138W) |
| `v1.12.7-rdna4-c1-smuonly-r4-amd64` | R3 + factory cmdline (`pci=realloc`, mlx5 blacklist) | 검증된 cmdline baseline |
| `v1.12.7-rdna4-c1-smuonly-r5-pci04-amd64` | R4 + 0001 + 0003 + 0004 | Phase 2b dual-GPU 32GB target |

## 빌드

```sh
cd ~/code/siderolabs/talos
make kernel REGISTRY=<registry>:5000 USERNAME=yanghoeg PLATFORM=linux/amd64 PUSH=true JOBS=10
make installer REGISTRY=<registry>:5000 USERNAME=yanghoeg PLATFORM=linux/amd64 PUSH=true
```

자세한 환경 setup 은 `bootstrap/talos/custom-kernel-build/{README,BUILDKIT}.md` (gitops repo).
