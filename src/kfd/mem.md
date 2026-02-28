# Memory
Kfd allocated memory is tied to a specific kfd node. For example cpu, gpu, npu, etc.
It can be shared between multiple kfd devices.

## Types (one of)
* userptr - user-allocated memory mapped for GPU access
* vram - gpu dedicated memory
* gtt - gpu accessible system memory
* doorbell - specially mapped memory region for mmio when using queues
* mmio_remap

If you pick multiple you might get an error or one of the selected will be used.
Just pick one.

### Can this be changed after a BO has been created?
Yes it can, although it's not straitforward to do. It's done internally with `ttm_bo_validate`.
Which then uses the appropriate memory manager depending on memory placement for example vram_mgr.

## Attributes (multiple of)
* writable
* executable
* public
* no substitute
* aql queue mem
* coherent
* uncached
* coherent_ext
* contiguous - asks the allocator to asign physical memory in one not fragmented block

## Evictions, moves and pinning
### When can eviction happen?

You can manually trigger an eviction in via /sys/kernel/debug/dri/0/amdgpu_evict_*.

### What does an eviction change or impact?

### When can a move happen?

### What is pinning?

Pinning means to lock pages in memory along with keeping them at a fixed
offset. It is required when a buffer can not be moved, for example, when
a display buffer is being scanned out.

### Which memory is automatically pinned?

In KFD only MMIO_REMAP and DOORBELL memory get's pinned (to GTT) during [alloc_memory_of_gpu](./ioctl/mem/alloc_memory_of_gpu.md)
and unpinned during [free_memory_of_gpu](./ioctl/mem/free_memory_of_gpu.md).

Special MES proc_ctx_bo and gang_ctx_bo also get pinned to GTT during first queue creation.

Kfd creates a kernel reserved IB bo for every process, which also gets pinned to the GTT.
DGPU's CWSR buffer also gets pinned to GTT.

Also when providing a signal page in [create_event](./ioctl/event.md#create_event) provided BO gets pinned to GTT.

It's invalid to try to pin/unpin USERPTR memory.

## Virtual Addresses
They are assigned in 4k byte pages, so
**when you pick a VA make sure it's PAGE_SIZE aligned**.

There is no alignment requirement based on memory size.

You should check the returned device aperture info.
Spefically gpuvm to know which VA to use for allocation.

### Reserved addresses
Bottom 0x0 - 0x10_000 (16 pages) are reserved for kernel.

GMC hole: 0x0000_8_0000_0000__000 - 0xffff_8_0000_0000__000.

Top is dependent on device address size.
48bit address for gfx103 and top is 0xffff_ffff_ffff.

From the top these are reserved for kernel:
- 2 pages for default CWSR trap handler,
- 512 pages for SEQ64,
- 512 pages for CSA.

Take note you might not get a conflict mapping memory to these adresses if they have not yet been mapped.
Except for 0x0 address, which is intentionally reserved for NULLPTR purposes.

## Interoperability with DRM's GEM subsystem
Maybe???

## Sharing via DMA-BUF

