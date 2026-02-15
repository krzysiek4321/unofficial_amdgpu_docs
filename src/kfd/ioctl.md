# Supported IOCTLS (kernel 6.17, kfd 1.18)
Add `AMDKFD_IOC_` to each to get C definitions.
For more info look into `kernel/include/uapi/linux/kfd_ioctl.h`

Implementation can be found in `kernel/drivers/gpu/drm/amd/amdkfd/kfd_chardev.c`

- [GET_VERSION](ioctl/get_version.md)
- [CREATE_QUEUE](ioctl/queue/create_queue.md)
- [UPDATE_QUEUE](ioctl/queue/update_queue.md)
- [DESTROY_QUEUE](ioctl/queue/destroy_queue.md)


## SET_MEMORY_POLICY
		AMDKFD_IOW(0x04, struct kfd_ioctl_set_memory_policy_args)

### Inputs
	__u64 alternate_aperture_base;	/* to KFD */
	__u64 alternate_aperture_size;	/* to KFD */

	__u32 gpu_id;			/* to KFD */
	__u32 default_policy;		/* to KFD */
	__u32 alternate_policy;		/* to KFD */
	__u32 misc_process_flag;        /* to KFD */

## GET_CLOCK_COUNTERS
		AMDKFD_IOWR(0x05, struct kfd_ioctl_get_clock_counters_args)

### Inputs
	__u32 gpu_id;		/* to KFD */

### Outputs
	__u64 gpu_clock_counter;	/* from KFD */
	__u64 cpu_clock_counter;	/* from KFD */
	__u64 system_clock_counter;	/* from KFD */
	__u64 system_clock_freq;	/* from KFD */

## GET_PROCESS_APERTURES
		AMDKFD_IOR(0x06, struct kfd_ioctl_get_process_apertures_args)

### Outputs
```C
struct {
    __u64 lds_base;		/* from KFD */
	__u64 lds_limit;		/* from KFD */
	__u64 scratch_base;		/* from KFD */
	__u64 scratch_limit;		/* from KFD */
	__u64 gpuvm_base;		/* from KFD */
	__u64 gpuvm_limit;		/* from KFD */
	__u32 gpu_id;		/* from KFD */
} nodes[7];
__u32 num_of_nodes;
```

## CREATE_EVENT
		AMDKFD_IOWR(0x08, struct kfd_ioctl_create_event_args)

### Inputs
```C
    #define KFD_IOC_EVENT_SIGNAL		0
    #define KFD_IOC_EVENT_NODECHANGE		1
    #define KFD_IOC_EVENT_DEVICESTATECHANGE	2
    #define KFD_IOC_EVENT_HW_EXCEPTION		3
    #define KFD_IOC_EVENT_SYSTEM_EVENT		4
    #define KFD_IOC_EVENT_DEBUG_EVENT		5
    #define KFD_IOC_EVENT_PROFILE_EVENT		6
    #define KFD_IOC_EVENT_QUEUE_EVENT		7
    #define KFD_IOC_EVENT_MEMORY		8
	__u32 event_type;		/* to KFD */
	__u32 auto_reset;		/* to KFD */
	__u32 node_id;		/* to KFD - only valid for certain
							event types */
```

### Outputs
	__u64 event_page_offset;	/* from KFD */
	__u32 event_trigger_data;	/* from KFD - signal events only */
	__u32 event_id;		/* from KFD */
	__u32 event_slot_index;	/* from KFD */


## DESTROY_EVENT
		AMDKFD_IOW(0x09, struct kfd_ioctl_destroy_event_args)

### Input
	__u32 event_id;		/* to KFD */

## SET_EVENT
		AMDKFD_IOW(0x0A, struct kfd_ioctl_set_event_args)

### Input
	__u32 event_id;		/* to KFD */

## RESET_EVENT
		AMDKFD_IOW(0x0B, struct kfd_ioctl_reset_event_args)

### Input
	__u32 event_id;		/* to KFD */

## WAIT_EVENTS
		AMDKFD_IOWR(0x0C, struct kfd_ioctl_wait_events_args)

### Inputs
	__u64 events_ptr;		/* pointed to struct
					   kfd_event_data array, to KFD */
	__u32 num_events;		/* to KFD */
	__u32 wait_for_all;		/* to KFD */
	__u32 timeout;		/* to KFD */

### Outputs
```C
    #define KFD_IOC_WAIT_RESULT_COMPLETE	0
    #define KFD_IOC_WAIT_RESULT_TIMEOUT		1
    #define KFD_IOC_WAIT_RESULT_FAIL		2
	__u32 wait_result;		/* from KFD */
```

## DBG_REGISTER_DEPRECATED
		AMDKFD_IOW(0x0D, struct kfd_ioctl_dbg_register_args)

## DBG_UNREGISTER_DEPRECATED
		AMDKFD_IOW(0x0E, struct kfd_ioctl_dbg_unregister_args)

## DBG_ADDRESS_WATCH_DEPRECATED
		AMDKFD_IOW(0x0F, struct kfd_ioctl_dbg_address_watch_args)

## DBG_WAVE_CONTROL_DEPRECATED
		AMDKFD_IOW(0x10, struct kfd_ioctl_dbg_wave_control_args)

## SET_SCRATCH_BACKING_VA
		AMDKFD_IOWR(0x11, struct kfd_ioctl_set_scratch_backing_va_args)

## GET_TILE_CONFIG
		AMDKFD_IOWR(0x12, struct kfd_ioctl_get_tile_config_args)

## SET_TRAP_HANDLER
		AMDKFD_IOW(0x13, struct kfd_ioctl_set_trap_handler_args)

## GET_PROCESS_APERTURES_NEW
		AMDKFD_IOWR(0x14, struct kfd_ioctl_get_process_apertures_new_args)

Just like [GET_PROCESS_APERTURES](#get_process_apertures) except there is no limit to the number of nodes.

## ACQUIRE_VM
		AMDKFD_IOW(0x15, struct kfd_ioctl_acquire_vm_args)

## ALLOC_MEMORY_OF_GPU
		AMDKFD_IOWR(0x16, struct kfd_ioctl_alloc_memory_of_gpu_args)

## FREE_MEMORY_OF_GPU
		AMDKFD_IOW(0x17, struct kfd_ioctl_free_memory_of_gpu_args)

## MAP_MEMORY_TO_GPU
		AMDKFD_IOWR(0x18, struct kfd_ioctl_map_memory_to_gpu_args)

## UNMAP_MEMORY_FROM_GPU
		AMDKFD_IOWR(0x19, struct kfd_ioctl_unmap_memory_from_gpu_args)

## SET_CU_MASK
		AMDKFD_IOW(0x1A, struct kfd_ioctl_set_cu_mask_args)


### Inputs
	__u32 queue_id;		/* to KFD */
	__u32 num_cu_mask;		/* to KFD */
	__u64 cu_mask_ptr;		/* to KFD */

**num_cu_mask** must be multiple of 32, because its unit is bit count and mask elements are uint32

## GET_QUEUE_WAVE_STATE
		AMDKFD_IOWR(0x1B, struct kfd_ioctl_get_queue_wave_state_args)

## GET_DMABUF_INFO
		AMDKFD_IOWR(0x1C, struct kfd_ioctl_get_dmabuf_info_args)

## IMPORT_DMABUF
		AMDKFD_IOWR(0x1D, struct kfd_ioctl_import_dmabuf_args)

## ALLOC_QUEUE_GWS
		AMDKFD_IOWR(0x1E, struct kfd_ioctl_alloc_queue_gws_args)

## SMI_EVENTS
		AMDKFD_IOWR(0x1F, struct kfd_ioctl_smi_events_args)

## SVM
        AMDKFD_IOWR(0x20, struct kfd_ioctl_svm_args)

## SET_XNACK_MODE
		AMDKFD_IOWR(0x21, struct kfd_ioctl_set_xnack_mode_args)

## CRIU_OP
		AMDKFD_IOWR(0x22, struct kfd_ioctl_criu_args)

## AVAILABLE_MEMORY
		AMDKFD_IOWR(0x23, struct kfd_ioctl_get_available_memory_args)

## EXPORT_DMABUF
		AMDKFD_IOWR(0x24, struct kfd_ioctl_export_dmabuf_args)

## RUNTIME_ENABLE
		AMDKFD_IOWR(0x25, struct kfd_ioctl_runtime_enable_args)

## DBG_TRAP
		AMDKFD_IOWR(0x26, struct kfd_ioctl_dbg_trap_args)

