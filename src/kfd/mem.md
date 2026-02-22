# Memory
Kfd allocated memory is tied to a specific kfd node. For example cpu, gpu, npu, etc.
It can be shared between multiple kfd devices.

## Types (one of)
* userptr - user-allocated memory mapped for GPU access
* vram - gpu dedicated memory
* gtt - gpu accessible system memory
* doorbell - specially mapped memory region for mmio when using queues
* mmio_remap

## Attributes (multiple of)
* writable
* executable
* public
* no substitute
* aql queue mem
* coherent
* uncached
* coherent_ext
* contiguous

## Interoperability with DRM's GEM subsystem
Maybe???

## Sharing via DMA-BUF

## Pinning?
In drm memory can be pinned.

It impact how much memory get's reported as avilable in kfd.

Is there a concept of pinning for kfd operations?
