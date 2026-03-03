# get_dmabuf_info
    AMDKFD_IOWR(0x1C, struct kfd_ioctl_get_dmabuf_info_args)

Only VRAM and GTT bos are supported.

It doesn't return all flags, only domain and PUBLIC.

Size is memory size in bytes.

Metadata size and layout is entirely up to user space application
which set it with [`GEM_METADATA`](../../../drm/ioctl.md#gem_metadata) ioctl.

- EINVAL if failed to find a kfd device the process have access to (via cgroup)
    or metadata_size is too small
- ENOMEM if out of memory
- EFAULT if failed to copy data back to user
- some errror if provided dmabuf_fd is incorrect
