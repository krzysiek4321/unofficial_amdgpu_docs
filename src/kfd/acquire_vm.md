# Preparing for memory operations
Before we can do memory operations we need to first [acquire_vm](#acquire_vm).

If you have an older gpu (before gfx10) you might also want to [set_memory_policy](#set_memory_policy).
For newer gpus you'd make use of allocation flags.

## IOCTLs
### acquire_vm
    AMDKFD_IOW(0x15, struct kfd_ioctl_acquire_vm_args)

##### What is this for?
Don't know

It turns a GFX VM into a Compute VM, but why would you want to do that?

Maybe to not have to create a new vm again if you already have a Drm vm you will not need
anymore.

Turns out it's required before allocating gpu memory.

Also initializes CWSR for the process.

It changes slighly how drm ioctls behave.
Grep for `is_compute_context`.
In gem_open when importing a gem it now also calls `amdgpu_amdkfd_bo_validate_and_fence()`, which might error.
Also when handling VM fault it slightly changes logic.

#### Required Inputs
	__u32 drm_fd;	/* to KFD */
	__u32 gpu_id;	/* to KFD */

Drm_fd must be a valid file descriptor to an opened amdgpu drm file.

##### Can I close the drm_fd after this ioctl?
I say you can, because the implementation uses `fget()` to increase refcount to drm_file
and `fput()` to decrese it on error or during `kfd_process_destroy_pdds()`.

##### What happens if I call it twice?
You will get EBUSY if the drm_file is different.
If it's the same file nothing happens.

### set_memory_policy
    AMDKFD_IOW(0x04, struct kfd_ioctl_set_memory_policy_args)
It may be pointless depending on the gpu generation.
At least for now. There has been a small change in version 1.18 (2025).

#### Required Inputs
	__u32 gpu_id;			/* to KFD */

##### Alternate aperture base
	__u64 alternate_aperture_base;	/* to KFD */
	__u64 alternate_aperture_size;	/* to KFD */

Only used with gfx7 and gfx8.

##### Cache policy
	__u32 default_policy;		/* to KFD */
	__u32 alternate_policy;		/* to KFD */

* KFD_IOC_CACHE_POLICY_COHERENT 0
* KFD_IOC_CACHE_POLICY_NONCOHERENT 1

For gfx9+ doesn't matter.
But for gfx7 and gfx8 it does get passed to the gpu.

##### Misc flag
	__u32 misc_process_flag;        /* to KFD */

Only for gfx9.5

* KFD_PROC_FLAG_MFMA_HIGH_PRECISION (1 << 0)

