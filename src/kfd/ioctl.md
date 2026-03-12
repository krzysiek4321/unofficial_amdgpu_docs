# IOCTLs
Add `AMDKFD_IOC_` to each to get C definitions.

For more info look into `kernel/include/uapi/linux/kfd_ioctl.h`

Implementation can be found in `kernel/drivers/gpu/drm/amd/amdkfd/kfd_chardev.c`

## On errors
AMDGPU driver doesn't have a clear error api.
A lot of them get propagated through internal calls, which makes it hard to know which
error values to expect.

But these errors should be a part of stable ABI.

## Uncategorized

- [GET_VERSION](ioctl/get_version.md)
- [SET_MEMORY_POLICY](./acquire_vm.md#set_memory_policy)
- [GET_CLOCK_COUNTERS](./perf.md#get_clock_counters)
- [SVM](ioctl/svm.md#svm)
- [SET_XNACK_MODE](ioctl/svm.md#set_xnack_mode)
- [CRIU_OP](./ioctl/criu.md#criu_op)
- [SMI_EVENTS](./ioctl/smi.md#smi_events)

## Query devices
- [GET_PROCESS_APERTURES](./apertures.md#get_process_apertures)
- [GET_PROCESS_APERTURES_NEW](./apertures.md#get_process_apertures_new)

## Queues
- [CREATE_QUEUE](./queues.md#create_queue)
- [UPDATE_QUEUE](./queues.md#update_queue)
- [DESTROY_QUEUE](./queues.md#destroy_queue)
- [SET_CU_MASK](./queues.md#set_cu_mask)
- [GET_QUEUE_WAVE_STATE](./queues.md#get_queue_wave_state)
- [ALLOC_QUEUE_GWS](./queues.md#alloc_queue_gws)

## Memory operations
- [ACQUIRE_VM](./acquire_vm.md#acquire_vm)
- [AVAILABLE_MEMORY](./alloc.md#available_memory)
- [ALLOC_MEMORY_OF_GPU](./alloc.md#alloc_memory_of_gpu)
- [FREE_MEMORY_OF_GPU](./alloc.md#free_memory_of_gpu)
- [MAP_MEMORY_TO_GPU](./va_mapping.md#map_memory_to_gpu)
- [UNMAP_MEMORY_FROM_GPU](./va_mapping.md#unmap_memory_from_gpu)
- [SET_SCRATCH_BACKING_VA](./va_mapping.md#set_scratch_backing_va)
- [GET_TILE_CONFIG](./tiling.md#get_tile_config)

### DMABUF
- [GET_DMABUF_INFO](./dmabuf.md#get_dmabuf_info)
- [IMPORT_DMABUF](./dmabuf.md#import_dmabuf)
- [EXPORT_DMABUF](./dmabuf.md#export_dmabuf)

## Events
- [CREATE_EVENT](./events.md#create_event)
- [DESTROY_EVENT](./events.md#destroy_event)
- [SET_EVENT](./events.md#set_event)
- [RESET_EVENT](./events.md#reset_event)
- [WAIT_EVENTS](./events.md#wait_events)

## Debug
- [SET_TRAP_HANDLER](./ioctl/dbg.md#set_trap_handler)
- [RUNTIME_ENABLE](./ioctl/dbg.md#runtime_enable)
- [DBG_TRAP](./ioctl/dbg.md#dbg_trap)

### Deprecated
- [DBG_REGISTER_DEPRECATED](./ioctl/dbg.md#dbg_register_deprecated)
- [DBG_UNREGISTER_DEPRECATED](./ioctl/dbg.md#dbg_unregister_deprecated)
- [DBG_ADDRESS_WATCH_DEPRECATED](./ioctl/dbg.md#dbg_address_watch_deprecated)
- [DBG_WAVE_CONTROL_DEPRECATED](./ioctl/dbg.md#dbg_wave_control_deprecated)
