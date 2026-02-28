# Monitoring gpu state
Aside from collecting information by the applications when interracting with DRM of KFD api
there are some files available in sysfs to read and modify the gpu's or kernel module's state.

## /sys/kernel/debug/dri/

amdgpu_evict_gtt - manually triggers an eviction of GTT bos
amdgpu_evict_vram - manuall triggers an eviction of VRAM bos

## /sys/kernel/debug/kfd/

## /sys/class/kfd/kfd/

## /sys/class/drm/

## /sys/module/amdgpu/

## /sys/fs/cgroup/dmem.*
