# export_dmabuf
    AMDKFD_IOWR(0x24, struct kfd_ioctl_export_dmabuf_args)

It basically uses DRM's gem_prime_export. See [PRIME_HANDLE_TO_FD](../../../drm/ioctl.md#prime_handle_to_fd).

### Inputs
	__u64 handle;		/* to KFD */
	__u32 flags;		/* to KFD */

Flags will be set for created file descriptor and are the same as for `open()` syscall.

### Outputs
	__u32 dmabuf_fd;	/* from KFD */

- EPERM - if you try to export a USERPTR memory
    or underlying BO has AMDGPU_GEM_CREATE_VM_ALWAYS_VALID flag set
