# GEM objects

These correspond to blobs of memory, which can be partitioned, recognized by the gpu driver.

Each object is reference counted and automatically deleted when refcount reaches 0.

Some gem objects are created by the kernel driver.

You can see current gem objects in `/sys/kernel/debug/dri/*gpu*/amdgpu_gem_info`.

## Sharing between processes
### FLINK
An older sharing mechanism, which uses [DRM_IOCTL_GEM_FLINK](ioctl.md#gem_flink) to assign
a **unique for a gpu** integer "name" that can be used by **anybody** to import this object using
[DRM_IOCTL_GEM_OPEN](ioctl.md#gem_open).

### PRIME (aka dma-buf)
A newer more secure mechanism uses creating **dma-buf file descriptors** [DRM_IOCTL_PRIME_HANDLE_TO_FD](ioctl.md#prime_handle_to_fd)
for gem objects that need to be **passed over a unix socket** to a process which want's to import a gem object
[DRM_IOCTL_PRIME_FD_TO_HANDLE](ioctl.md#prime_fd_to_handle).

## Pinning

## Syncobjects
