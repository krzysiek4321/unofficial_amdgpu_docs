# map_memory_to_gpu
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
### Inputs
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
