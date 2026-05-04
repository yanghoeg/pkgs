# Talos kernel patches (yanghoeg fork, release-1.12 base = 6.18.24)

Single vendor patch in this fork. PCI / BAR-related work is upstream
or in-flight on LKML — pull those yourself from the appropriate branch
(`pci-next`, `linus/master`, or Ilpo Järvinen's posted series) if your
hardware needs them.

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
