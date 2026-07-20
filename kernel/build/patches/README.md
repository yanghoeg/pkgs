# Talos kernel patches (yanghoeg fork)

Base: `Pkgfile:linux_version` — 현재 `6.19.0-drm-next-73ec35caf`
(agd5f amd-staging-drm-next 스냅샷, 2026-04-28).

대상 하드웨어: ASUS X99-E WS + E5-2699v4, Radeon AI PRO R9700 (gfx1201) ×2,
ConnectX-5 (mlx5). 패치 대부분이 이 조합 고유의 문제를 다룬다.

| Patch | 무엇 | Upstream 상태 |
|---|---|---|
| `0001-PCI-prevent-shrink-bridge-window` | 브리지 윈도우 부당 축소 방지 (Ilpo Järvinen, `dc4b4d04e1ca`) | mainline v7.1-rc1. **6.18.x LTS 에는 없음** |
| `0002-PCI-fix-premature-removal-realloc-head` | `realloc_head` 조기 제거 버그 (Ilpo, `1ee4716a5a28`) | mainline v7.1-rc1 + **6.18.33 백포트됨**. siderolabs/pkgs PR #1536 (2026-05-07 머지) |
| `0004-drm-amdgpu-skip-shutdown-...` | `amdgpu_pci_shutdown` 스킵 — RDNA4 FW 가 poweroff 중 suspend 에서 hang | 자체 저작, not upstreamable (workaround) |
| `0005-mlx5-skip-shutdown-...` | mlx5 shutdown 스킵 — CX5 FW 가 SR-IOV VF 있을 때 poweroff hang | 자체 저작, not upstreamable (workaround) |
| `0006-PCI-add-pcie_acs_override-boot-parameter` | PLX ACS 우회로 GPU P2P 허용 | 자체 저작. 메인테이너가 오랫동안 거부해온 계열 — **영구 out-of-tree** |
| `0007-PCI-p2pdma-add-Broadwell-EP-to-whitelist` | Broadwell-EP 를 p2pdma whitelist 에 추가 | 자체 저작, 미제출 |
| `0008-drm-amdgpu-gmc12-widen-DMA-mask-44-to-48-bit` | gfx1201 DMA mask 44→48bit — X99 가 BAR 를 44bit 위에 매핑해 P2P 거부됨 | 자체 저작, 미제출 |

## 베이스를 바꿀 때 (gotcha)

이 목록은 **베이스 커널에 종속**이다. 리베이스 전 반드시 `patch --check` 부터.

- **6.18.x LTS 로 내려가면 `0003-PCI-try-bar-resize-without-window-release` 를 되살려야 한다.**
  6.19 drm-next 는 `drivers/pci/setup-bus.c` 의
  `pbus_reassign_bridge_resources()` 게이트를 `list_empty(&saved)` → `!bridge` 로
  이미 바꿔놨지만 6.18.x 에는 그 변경이 없다. 이 패치가 없으면
  **dual-32GB BAR 리사이즈가 실패한다.** 삭제 사유는 커밋 `a48af98` 참조.
- **`0002` 는 6.18.33 이후 베이스에서 제거해야 한다** — 중복 적용.
- `0001-amdgpu-smu14-0x33` (SMU14_0_2 driver-if 0x2E→0x33) 은 6.19 에서 심볼이
  `swsmu/inc/smu_v14_0.h` → `swsmu/smu14/smu_v14_0_2_ppt.c` 로 이동해 드롭했다.
  증상(idle +220W, 팬 무응답)이 실측되지 않아 복원 안 함 —
  베이스 변경 시 `rocm-smi --showpower --showfan` 으로 재확인할 것.

## Build

```sh
cd ~/code/siderolabs/talos
make kernel REGISTRY=<registry>:5000 USERNAME=yanghoeg PLATFORM=linux/amd64 PUSH=true JOBS=10
make installer REGISTRY=<registry>:5000 USERNAME=yanghoeg PLATFORM=linux/amd64 PUSH=true
```

⚠ **커널만 바꾸면 끝이 아니다** — 모듈이 initramfs(`rootfs.sqsh`) 안에 있어
hybrid initramfs 재조립이 필요하다. 그리고 `make` 는 `.config` 를 syncconfig 로
다시 쓰므로 **의존성 미충족 심볼이 조용히 떨어진다** — 빌드된 vmlinuz 의
ikconfig 로 검증해야 한다.

절차: gitops repo `bootstrap/talos/custom-kernel-build/README.md` §7, §7-b
