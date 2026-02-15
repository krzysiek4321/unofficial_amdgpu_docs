# acquire_vm
    AMDKFD_IOW(0x15, struct kfd_ioctl_acquire_vm_args)

#### What is this for? Espacially since not all kfd nodes are drm devices.
Don't know

## Required Inputs
	__u32 drm_fd;	/* to KFD */
	__u32 gpu_id;	/* to KFD */

Drm_fd must be a valid file descriptor to opened drm file.

#### Can I close the drm_fd after this ioctl?
I say you can, because the implementation uses `fget()` to increase refcount to drm_file
and `fput()` to decrese it on error or during `kfd_process_destroy_pdds()`.

#### What happens if I call it twice?
You will get EBUSY if the drm_file is different.
If it's the same file nothing happens.
