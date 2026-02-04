# Supported IOCTLS (kernel 6.17, kfd 1.18)
Add `AMDKFD_IOC_` to each to get C definitions.
For more info look into `kernel/include/uapi/linux/kfd_ioctl.h`

Implementation can be found in `kernel/drivers/gpu/drm/amd/amdkfd/kfd_chardev.c`

## GET_VERSION
Returns version of amdkfd driver.

## CREATE_QUEUE
		AMDKFD_IOWR(0x02, struct kfd_ioctl_create_queue_args)

## DESTROY_QUEUE
		AMDKFD_IOWR(0x03, struct kfd_ioctl_destroy_queue_args)

## SET_MEMORY_POLICY
		AMDKFD_IOW(0x04, struct kfd_ioctl_set_memory_policy_args)

## GET_CLOCK_COUNTERS
		AMDKFD_IOWR(0x05, struct kfd_ioctl_get_clock_counters_args)

## GET_PROCESS_APERTURES
		AMDKFD_IOR(0x06, struct kfd_ioctl_get_process_apertures_args)

## UPDATE_QUEUE
		AMDKFD_IOW(0x07, struct kfd_ioctl_update_queue_args)

## CREATE_EVENT
		AMDKFD_IOWR(0x08, struct kfd_ioctl_create_event_args)

## DESTROY_EVENT
		AMDKFD_IOW(0x09, struct kfd_ioctl_destroy_event_args)

## SET_EVENT
		AMDKFD_IOW(0x0A, struct kfd_ioctl_set_event_args)

## RESET_EVENT
		AMDKFD_IOW(0x0B, struct kfd_ioctl_reset_event_args)

## WAIT_EVENTS
		AMDKFD_IOWR(0x0C, struct kfd_ioctl_wait_events_args)

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
		AMDKFD_IOWR(0x14,		\
			struct kfd_ioctl_get_process_apertures_new_args)

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

