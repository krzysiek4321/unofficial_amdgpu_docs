# Buffer Object Metadata
For every user created buffer object metadata can be added and stored in kernel space.

This way it's easier to share certain properties for example how to interpret this buffer
between applications using the same user space driver (MESA).

The metadata doesn't impact the functionality of using the BO.

To add metadata you'd need to use `DRM_AMDGPU_GEM_METADATA`.

It alows you to store tiling used by this buffer object.

It also allows you to set whatever you want into
- flags
- custom_metadata_buffer

The custom metadata doesn't have a fixed size.
But it has a limit of at most 64 uint32 values.
Underneath it could be any size, but that's how this ioctl was designed.

To retrieve this metadata or some part of it you'd use `DRM_AMDGPU_GEM_METADATA` or `AMDKFD_IOC_GET_DMABUF_INFO`.
