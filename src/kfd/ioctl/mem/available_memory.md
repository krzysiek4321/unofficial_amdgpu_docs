# available_memory
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


## Required Inputs
	__u32 gpu_id;		/* to KFD */

## Outputs
	__u64 available;	/* from KFD */
Available bytes, usually from VRAM for gpus.

For VRAM the value is aligned down to 2MiB
>to avoid fragmentation caused by 4K allocations in the tail 2MB BO chunk.
>
> —— Daniel Phillips 2022, on behalf of AMD

For apus, which preffer gtt, the value is min of available types aligned down to system page size.

#### What if the kernel is configured with a page size different from 4KiB?
