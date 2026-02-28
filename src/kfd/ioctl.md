# Supported IOCTLS (kernel 6.17, kfd 1.18)
Add `AMDKFD_IOC_` to each to get C definitions.

For more info look into `kernel/include/uapi/linux/kfd_ioctl.h`

Implementation can be found in `kernel/drivers/gpu/drm/amd/amdkfd/kfd_chardev.c`

You can get **gpu_id** from `/sys/class/kfd/kfd/topology/nodes/*/gpu_id` or with `GET_PROCESS_APERTURES`.

## On errors
AMDGPU driver doesn't have a clear error api.
A lot of them get propagated through internal calls, which makes it hard to know which
error values to expect.

But these errors should be a part of stable ABI.

## IOCTLs

- [GET_VERSION](ioctl/get_version.md)

- [GET_PROCESS_APERTURES](ioctl/apertures.md#get_process_apertures)
- [GET_PROCESS_APERTURES_NEW](ioctl/apertures.md#get_process_apertures_new)

- [CREATE_QUEUE](ioctl/queue/create_queue.md)
- [UPDATE_QUEUE](ioctl/queue/update_queue.md)
- [DESTROY_QUEUE](ioctl/queue/destroy_queue.md)

- [AVAILABLE_MEMORY](ioctl/mem/available_memory.md)
- [ACQUIRE_VM](ioctl/mem/acquire_vm.md)
- [ALLOC_MEMORY_OF_GPU](ioctl/mem/alloc_memory_of_gpu.md)

- [CREATE_EVENT](ioctl/event.md#create_event)
- [DESTROY_EVENT](ioctl/event.md#destroy_event)
- [SET_EVENT](ioctl/event.md#set_event)
- [RESET_EVENT](ioctl/event.md#reset_event)
- [WAIT_EVENTS](ioctl/event.md#wait_events)


## GET_CLOCK_COUNTERS
		AMDKFD_IOWR(0x05, struct kfd_ioctl_get_clock_counters_args)

### Inputs
	__u32 gpu_id;		/* to KFD */

### Outputs
	__u64 gpu_clock_counter;	/* from KFD */
	__u64 cpu_clock_counter;	/* from KFD */
	__u64 system_clock_counter;	/* from KFD */
	__u64 system_clock_freq;	/* from KFD */

## SET_SCRATCH_BACKING_VA
		AMDKFD_IOWR(0x11, struct kfd_ioctl_set_scratch_backing_va_args)

```C
struct kfd_ioctl_set_scratch_backing_va_args {
	__u64 va_addr;	/* to KFD */
	__u32 gpu_id;	/* to KFD */
	__u32 pad;
};
```

Only used for no CP scheduling mode.

## GET_TILE_CONFIG
		AMDKFD_IOWR(0x12, struct kfd_ioctl_get_tile_config_args)

## SET_CU_MASK
		AMDKFD_IOW(0x1A, struct kfd_ioctl_set_cu_mask_args)

### Inputs
	__u32 queue_id;		/* to KFD */
	__u32 num_cu_mask;		/* to KFD */
	__u64 cu_mask_ptr;		/* to KFD */

**num_cu_mask** must be multiple of 32, because its unit is bit count and mask elements are uint32

## GET_QUEUE_WAVE_STATE
		AMDKFD_IOWR(0x1B, struct kfd_ioctl_get_queue_wave_state_args)

## ALLOC_QUEUE_GWS
		AMDKFD_IOWR(0x1E, struct kfd_ioctl_alloc_queue_gws_args)

## SET_XNACK_MODE
		AMDKFD_IOWR(0x21, struct kfd_ioctl_set_xnack_mode_args)
