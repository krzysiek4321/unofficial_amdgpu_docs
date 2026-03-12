# Mapping memory to GPU's address space

## Virtual Addresses
They are assigned in 4KiB pages, so
**when you pick a VA make sure it's PAGE_SIZE aligned**.

There is no alignment requirement based on memory size.

You should check the returned device aperture info.
Spefically gpuvm to know which VA to use for allocation.

### Reserved addresses
Bottom 0x0 - 0x10_000 (16 pages) are reserved for kernel.

GMC hole: 0x0000_8_0000_0000__000 - 0xffff_8_0000_0000__000.

Top is dependent on device address size.
48bit address for gfx103 and top is 0xffff_ffff_ffff.

From the top these are reserved for kernel:
- 2 pages for default CWSR trap handler,
- 512 pages for SEQ64,
- 512 pages for CSA.

Take note you might not get a conflict mapping memory to these adresses if they have not yet been mapped.
Except for 0x0 address, which is intentionally reserved for NULLPTR purposes.

## IOCTLs
### map_memory_to_gpu
    AMDKFD_IOWR(0x18, struct kfd_ioctl_map_memory_to_gpu_args)
```C
/* Map memory to one or more GPUs
 *
 * @handle:                memory handle returned by alloc
 * @device_ids_array_ptr:  array of gpu_ids (__u32 per device)
 * @n_devices:             number of devices in the array
 * @n_success:             number of devices mapped successfully
 *
 * @n_success returns information to the caller how many devices from
 * the start of the array have mapped the buffer successfully. It can
 * be passed into a subsequent retry call to skip those devices. For
 * the first call the caller should initialize it to 0.
 *
 * If the ioctl completes with return code 0 (success), n_success ==
 * n_devices.
 */
struct kfd_ioctl_map_memory_to_gpu_args {
	__u64 handle;			/* to KFD */
	__u64 device_ids_array_ptr;	/* to KFD */
	__u32 n_devices;		/* to KFD */
	__u32 n_success;		/* to/from KFD */
};
```
#### Outputs
`__u32 n_success` how many devicess sucessfully mapped the memory to their VA table


- EINVAL - invalid device_id present
    or invalid handle
    or n_success > n_devices
    or n_devices == 0
    or VA is aleary mapped
    or VA is 0
    or VA is not PAGE_SIZE aligned
- ENOMEM - no memory available to copy user data to or invalid handle
- EFAULT - copying data from user

### unmap_memory_from_gpu
    AMDKFD_IOWR(0x19, struct kfd_ioctl_unmap_memory_from_gpu_args)

```C
struct kfd_ioctl_unmap_memory_from_gpu_args {
	__u64 handle;			/* to KFD */
	__u64 device_ids_array_ptr;	/* to KFD */
	__u32 n_devices;		/* to KFD */
	__u32 n_success;		/* to/from KFD */
};
```

### SET_SCRATCH_BACKING_VA
    AMDKFD_IOWR(0x11, struct kfd_ioctl_set_scratch_backing_va_args)

```C
struct kfd_ioctl_set_scratch_backing_va_args {
	__u64 va_addr;	/* to KFD */
	__u32 gpu_id;	/* to KFD */
	__u32 pad;
};
```

Only used for no CP scheduling mode (KFD_SCHED_POLICY_NO_HWS).
