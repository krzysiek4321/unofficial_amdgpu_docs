# Kernel Fusion Driver (AMDKFD)
Accessed via `/dev/kfd`, which can be used with `ioctl()` or `mmap()`.
This file handles all gpus.

It's what ROCm is built on.

Keep in mind, file descriptor obtained from open(/dev/kfd) cannot be shared between processes.

Having this file descriptor you have two available api's
- [IOCTL](ioctl.md)
- [MMAP](mmap.md)
