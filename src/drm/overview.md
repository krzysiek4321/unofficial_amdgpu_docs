# DRM
It's a commond api for doing graphics on linux.

Some parts of it are intentionally driver specific.

## Files / clients
Opening **/dev/dri/card%d** gives a unique DRM_MINOR_PRIMARY client.
Opening **/dev/dri/renderD%d** gives a unique DRM_MINOR_RENDER client.
Opening **/dev/dri/accel%d** gives a unique DRM_MINOR_ACCEL client.

Each gpu should get a primary and render files.

You'll most likely want to use the RENDER client.

If you need to have multiple file descriptors to a drm file simply dupplicate them with `dup()`.

## Permission structure
`drm_ioctl_permit()` is used to determine if the user have sufficient permisions to invoke an IOCTL.

These are the relevand flags set for IOCTLs:
- DRM_ROOT_ONLY - only allow when capable(CAP_SYS_ADMIN)
- DRM_AUTH - only allow authenticated render clients.
- DRM_MASTER - only allow current master
- DRM_RENDER_ALLOW - unless set, render clients not allowed

You can see currently existing drm_file and info if they are master or authenticated
in corresponding drm debugfs `/sys/kernel/debug/dri/*/clients`.

### MASTER
There can be at most only one, set for a device, at a time.
You might get master status by opening a **primary client** or using `SET_MASTER` ioctl on a **primary client** after the previous master closed or used `DROP_MASTER` ioctl.

## Reference counted
Opening these files return a reference counted object for this process, which means opening the files multiple times or dupplicating these file descriptors still reference the same object.

## Message format
Commands here use PM4 format.

## Source
More info in `kernel/drivers/gpu/drm/amd/amdgpu/amdgpu_drv.c`.
