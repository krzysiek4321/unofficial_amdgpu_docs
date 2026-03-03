# Memory
Kfd allocated memory is tied to a specific kfd node. For example cpu, gpu, npu, etc.
It can be shared between multiple kfd devices.

## Types (one of)
* userptr - user-allocated memory mapped for GPU access
* vram - gpu dedicated memory
* gtt - gpu accessible system memory managed by kernel module
* doorbell - specially mapped memory region for mmio when using queues
* mmio_remap - special memory page designed for direct Memory Mapped Io operations on device

If you pick multiple you might get an error or one of the selected will be used.
Just pick one.

### Can this be changed after a BO has been created?
Yes it can, although it's not straitforward to do. It's done internally with `ttm_bo_validate`.
Which then uses the appropriate memory manager depending on memory placement for example vram_mgr.

## Attributes (multiple of)
* writable - allows GPU to write to this memory
* executable - allows GPU to execute instructions from this memory
* public - corresponds to AMDGPU_GEM_CREATE_CPU_ACCESS_REQUIRED, for VRAM resizable bar is required, but only in KFD
* no substitute - no meaning as of now
* aql queue mem - use if you want to send AQL packets there
* contiguous - asks the allocator to asign physical memory in one not fragmented block

### Caching policy
Impacts `->get_vm_pte()` function used primarily in `amdgpu_vm_update`.

It used to be very complicated for gfx9 (GC 9.*).

* uncached -> MTYPE_UC

* coherent - MTYPE_UC, except for GC 9.4.1 and 9.4.2 it's MTYPE_CC if vram and bo from this gpu or MTYPE_RW if not set
* coherent_ext - only matters for GC 9.4.3, 9.4.4 and 9.5, MTYPE_CC if mem local to numa node, MTYPE_UC otherwise or MTYPE_RW if flag not set and is BO is local to device


It can be simplified to AMDGPU_VM_MTYPE_UC and AMDGPU_VM_MTYPE_NC.

## VMID
There is a total of 16 VMIDs per VMHUB.
VMID 0 is reserved for the system.

Since gfx10
VMID 1..7 used for graphics
VMID 8..15 used for kfd

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

