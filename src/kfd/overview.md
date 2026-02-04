# AMDKFD (ROCm)
Accessed via `/dev/kfd`, which can be used with `ioctl()` or `mmap()`.
This file handles all gpus.

It's what ROCM is built on.

Uses AQL packets to communicate with the GPU.

Keep in mind, file descriptor obtained from open(/dev/kfd) cannot be shared by process's children.

Having this file descriptor you have two available api's
- [IOCTL](ioctl.md)
- [MMAP](mmap.md)
