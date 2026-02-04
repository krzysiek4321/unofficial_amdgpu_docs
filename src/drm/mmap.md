# mmap

Provide which gem object you wish to map in **offset**.

The object might not be mappable.

Once the right object is fount it's mmap function is called.
See `amdgpu_gem_object_mmap()` in `kernel/drivers/gpu/drm/amd/amdgpu/amdgpu_gem.c`.

Remember gem objects are reference counted.
