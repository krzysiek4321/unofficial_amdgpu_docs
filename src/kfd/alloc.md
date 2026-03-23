# Allocating and releasing GPU aware memory
Kfd allocated memory is tied to a specific kfd node. For example cpu, gpu, npu, etc.
It can be shared between multiple kfd devices.

The kernel module is keeping track of memory via buffer objects (BOs).
To you it will return a handle, but keep in mind it is **not a gem handle**.

Allocations are always done in 4KiB pages.

You should first pick a gpu.
If you wish you can check how much roughly there is VRAM available with
[available_memory](#available_memory).
Try to allocate memory with [alloc_memory_of_gpu](#alloc_memory_of_gpu).
You can manually free this memory with [free_memory_of_gpu](#free_memory_of_gpu)
but if you will not, it will be released during process exit.

If you shared it via dmabuf it may not get released untill all holders either free it or exit themselves.

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

### Creating userptr
Instead of the kernel module allocating memory it is instead provided via the offset field.

## Attributes (multiple of)
* writable - allows GPU to write to this memory
* executable - allows GPU to execute instructions from this memory
* public - corresponds to AMDGPU_GEM_CREATE_CPU_ACCESS_REQUIRED, for VRAM resizable bar is required, but only in KFD
* no substitute - no meaning as of now
* aql queue mem - use if you want to write AQL packets there
* contiguous - asks the allocator to asign physical memory in one not fragmented block

### Caching policy
Impacts `->get_vm_pte()` function used primarily in `amdgpu_vm_update`.

It used to be very complicated for gfx9 (GC 9.*).

* uncached -> MTYPE_UC

* coherent - MTYPE_UC, except for GC 9.4.1 and 9.4.2 it's MTYPE_CC if vram and bo from this gpu or MTYPE_RW if not set
* coherent_ext - only matters for GC 9.4.3, 9.4.4 and 9.5, MTYPE_CC if mem local to numa node, MTYPE_UC otherwise or MTYPE_RW if flag not set and is BO is local to device


It can be simplified to AMDGPU_VM_MTYPE_UC and AMDGPU_VM_MTYPE_NC.

## IOCTLs
### alloc_memory_of_gpu
    AMDKFD_IOWR(0x16, struct kfd_ioctl_alloc_memory_of_gpu_args)

##### What if I set mutpltiple domain flags?
For example doorbell | mmio_remap.

It just allocated a doorbell page.

It seems domain should have been an enum and not bitflags.

##### What if I assign the same VA to multiple allocations?
Nothing yet. Only when mappping the memory to gpus the VAs get checked.
You'll get error on conflict.

```C
/* Allocation flags: memory types */
#define KFD_IOC_ALLOC_MEM_FLAGS_VRAM		(1 << 0)
#define KFD_IOC_ALLOC_MEM_FLAGS_GTT		(1 << 1)
#define KFD_IOC_ALLOC_MEM_FLAGS_USERPTR		(1 << 2)
#define KFD_IOC_ALLOC_MEM_FLAGS_DOORBELL	(1 << 3)
#define KFD_IOC_ALLOC_MEM_FLAGS_MMIO_REMAP	(1 << 4)
/* Allocation flags: attributes/access options */
#define KFD_IOC_ALLOC_MEM_FLAGS_WRITABLE	(1 << 31)
#define KFD_IOC_ALLOC_MEM_FLAGS_EXECUTABLE	(1 << 30)
#define KFD_IOC_ALLOC_MEM_FLAGS_PUBLIC		(1 << 29)
#define KFD_IOC_ALLOC_MEM_FLAGS_NO_SUBSTITUTE	(1 << 28)
#define KFD_IOC_ALLOC_MEM_FLAGS_AQL_QUEUE_MEM	(1 << 27)
#define KFD_IOC_ALLOC_MEM_FLAGS_COHERENT	(1 << 26)
#define KFD_IOC_ALLOC_MEM_FLAGS_UNCACHED	(1 << 25)
#define KFD_IOC_ALLOC_MEM_FLAGS_EXT_COHERENT	(1 << 24)
#define KFD_IOC_ALLOC_MEM_FLAGS_CONTIGUOUS	(1 << 23)
```

#### Required Inputs
	__u32 gpu_id;		/* to KFD */
	__u64 size;		/* to KFD */
	__u32 flags;

#### Conditional Inputs
	__u64 mmap_offset;	/* to KFD (userptr), from KFD (mmap offset) */
	__u64 va_addr;		/* to KFD */

#### Outputs
	__u64 handle;		/* from KFD */
	__u64 mmap_offset;	/* to KFD (userptr), from KFD (mmap offset) */

mmap_offset is used by `mmap()` on drm file except for mmio_remap where it should be used with kfd file instead.

- ENODEV - you forgot to acquire_vm first

### free_memory_of_gpu
    AMDKFD_IOW(0x17, struct kfd_ioctl_free_memory_of_gpu_args)

#### Required Inputs
	__u64 handle;		/* from KFD */

### available_memory
    AMDKFD_IOWR(0x23, struct kfd_ioctl_get_available_memory_args)

**I don't like this ioctl; or prior decisions which made it neccessary**

> Add a new KFD ioctl to return the largest possible memory size that
can be allocated as a buffer object using
kfd_ioctl_alloc_memory_of_gpu. It attempts to use exactly the same
accept/reject criteria as that function so that allocating a new
buffer object of the size returned by this new ioctl is guaranteed to
succeed, barring races with other allocating tasks.
>
> —— Daniel Phillips 2022, on behalf of AMD


#### Required Inputs
	__u32 gpu_id;		/* to KFD */

#### Outputs
	__u64 available;	/* from KFD */
Available bytes, usually from VRAM for gpus.

For VRAM the value is aligned down to 2MiB
>to avoid fragmentation caused by 4K allocations in the tail 2MB BO chunk.
>
> —— Daniel Phillips 2022, on behalf of AMD

For apus, which preffer gtt, the value is min of available types aligned down to system page size.

##### What if the kernel is configured with a page size different from 4KiB?
A lot of things break in amdgpu code.
