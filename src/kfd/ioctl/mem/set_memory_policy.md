# set_memory_policy
    AMDKFD_IOW(0x04, struct kfd_ioctl_set_memory_policy_args)
It may be pointless depending on the gpu generation.
At least for now. There has been a small change in version 1.18 (2025).

## Required Inputs
	__u32 gpu_id;			/* to KFD */

### Alternate aperture base
	__u64 alternate_aperture_base;	/* to KFD */
	__u64 alternate_aperture_size;	/* to KFD */

Only used with gfx7 and gfx8.

### Cache policy
	__u32 default_policy;		/* to KFD */
	__u32 alternate_policy;		/* to KFD */

* KFD_IOC_CACHE_POLICY_COHERENT 0
* KFD_IOC_CACHE_POLICY_NONCOHERENT 1

For gfx9+ doesn't matter.
But for gfx7 and gfx8 it does get passed to the gpu.

### Misc flag
	__u32 misc_process_flag;        /* to KFD */

Only for gfx9.5

* KFD_PROC_FLAG_MFMA_HIGH_PRECISION (1 << 0)
