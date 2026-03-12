# Sharing memory between processes

## IOCTLs
### get_dmabuf_info
    AMDKFD_IOWR(0x1C, struct kfd_ioctl_get_dmabuf_info_args)

Only VRAM and GTT bos are supported.

It doesn't return all flags, only domain and PUBLIC.

Size is memory size in bytes.

Metadata size and layout is entirely up to user space application
which set it with [`GEM_METADATA`](../drm/ioctl.md#gem_metadata) ioctl.

- EINVAL if failed to find a kfd device the process have access to (via cgroup)
    or metadata_size is too small
- ENOMEM if out of memory
- EFAULT if failed to copy data back to user
- some errror if provided dmabuf_fd is incorrect

### import_dmabuf
    AMDKFD_IOWR(0x1D, struct kfd_ioctl_import_dmabuf_args)


### export_dmabuf
    AMDKFD_IOWR(0x24, struct kfd_ioctl_export_dmabuf_args)

It basically uses DRM's gem_prime_export. See [PRIME_HANDLE_TO_FD](../drm/ioctl.md#prime_handle_to_fd).

#### Inputs
	__u64 handle;		/* to KFD */
	__u32 flags;		/* to KFD */

Flags will be set for created file descriptor and are the same as for `open()` syscall.

#### Outputs
	__u32 dmabuf_fd;	/* from KFD */

- EPERM - if you try to export a USERPTR memory
    or underlying BO has AMDGPU_GEM_CREATE_VM_ALWAYS_VALID flag set
