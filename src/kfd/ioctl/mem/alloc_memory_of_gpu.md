# alloc_memory_of_gpu
    AMDKFD_IOWR(0x16, struct kfd_ioctl_alloc_memory_of_gpu_args)

#### What if I set mutpltiple domain flags?
For example doorbell | mmio_remap.

It just allocated a doorbell page.

It seems domain should have been an enum and not bitflags.

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

## Required Inputs
	__u32 gpu_id;		/* to KFD */
	__u64 size;		/* to KFD */
	__u32 flags;

## Conditional Inputs
	__u64 mmap_offset;	/* to KFD (userptr), from KFD (mmap offset) */
	__u64 va_addr;		/* to KFD */

## Outputs
	__u64 handle;		/* from KFD */
