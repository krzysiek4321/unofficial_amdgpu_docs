# GEM objects

These correspond to blobs of memory, which can be partitioned, recognized by the gpu driver.

The gem object may be placed in one of the available domains managed by respective managers.
Like vram_mgr and gtt_mgr. But they use drm_buddy allocator to assign available pages
in respecitve domain to objects.

Each object is reference counted and automatically deleted when refcount reaches 0.

Some gem objects are created by the kernel driver.

You can see current gem objects in `/sys/kernel/debug/dri/*gpu*/amdgpu_gem_info`.

## VM
A VM manages many BO.
That involves keeping and updating a page tables.
These updates can be done either by CPU or SDMA.

For systems without resizable (large) BAR - SDMA is prefered.

### VM ib pools, what do these do?
- immediate
- delayed

### Update interface
- map_table
- prepare
- update
- commit

## Verifying BO parameters
During creation a lot of things can happen and you are not guaranteed to get the parameters
you set.

You should use [AMDGPU_IOCTL_GEM_METADATA](ioctl.md#gem_metadata) to verify the specific flags
you care about.

## Parent
When using flag VM_ALWAYS_VALID the special root bo is created for amdgpu_drm file's VM
and asigned as parent to the new BO.

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
