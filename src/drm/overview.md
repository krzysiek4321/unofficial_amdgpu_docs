# DRM
It's a commond api for doing graphics on linux.

Some parts of it are intentionally driver specific.

## Files
Accessed via **/dev/dri/card(N)** or **/dev/dri/renderD(M)**. Each device should get it's own file in **/dev/dri**.

## Differences
The **card(N)** file is reserved for so called DRM master. Only one master can be using a device at a time.
You can switch VT (chvt command) to have multiple masters running, but only one will have access to the gpu at a time.
Compositor is ususally the master.

The **renderD(M)** file is used by client applications, but ususally need to first obtain permission - called auth token - from master to
do things with the device. It has a restricted set of available operations comparred to the above.

## Reference counted
Opening these files return a reference counted object for this process, which means opening the files multiple times or dupplicating these file descriptors still reference the same object.

## Message format
Commands here use PM4 format.

## Source
More info in `kernel/drivers/gpu/drm/amd/amdgpu/amdgpu_drv.c`.
