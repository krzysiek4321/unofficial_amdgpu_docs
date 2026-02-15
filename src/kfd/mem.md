# Memory
Kfd allocated memory is tied to a specific kfd node. For example cpu, gpu, npu, etc.
It can be shared between multiple kfd devices.

## Interoperability with DRM's GEM subsystem
Maybe???

## Sharing via DMA-BUF

## Pinning?
In drm memory can be pinned.

It impact how much memory get's reported as avilable in kfd.

Is there a concept of pinning for kfd operations?
