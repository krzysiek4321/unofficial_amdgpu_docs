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

## /sys/module/drm/parameters/debug
Allows to enable debugging messages to show in kernel ring buffer (dmesg).

Use the following to enable all messages.
```sh
echo 0x1ff > /sys/module/drm/parameters/debug
```

Use the following to disable all messages.
```sh
echo 0x0 > /sys/module/drm/parameters/debug
```

Category info from kernel source code
```C
MODULE_PARM_DESC(debug, "Enable debug output, where each bit enables a debug category.\n"
"\t\tBit 0 (0x01)  will enable CORE messages (drm core code)\n"
"\t\tBit 1 (0x02)  will enable DRIVER messages (drm controller code)\n"
"\t\tBit 2 (0x04)  will enable KMS messages (modesetting code)\n"
"\t\tBit 3 (0x08)  will enable PRIME messages (prime code)\n"
"\t\tBit 4 (0x10)  will enable ATOMIC messages (atomic code)\n"
"\t\tBit 5 (0x20)  will enable VBL messages (vblank code)\n"
"\t\tBit 7 (0x80)  will enable LEASE messages (leasing code)\n"
"\t\tBit 8 (0x100) will enable DP messages (displayport code)");
```
