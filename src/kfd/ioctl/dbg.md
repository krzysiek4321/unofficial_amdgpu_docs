# Debug

## Watch points
There is a maximum of 4 watch points.

## RUNTIME_ENABLE
    AMDKFD_IOWR(0x25, struct kfd_ioctl_runtime_enable_args)

TODO: look at commit in kernel 455227c4642c5e1867213cea73a527e431779060
it somewhat explains the mechanism

Set's gpu's hardware status register TRAP_EN to true. (For gfx10 and gfx103)
Which notifies the gpu a trap handler is present.
From that point exceptions will trigger the trap handler for vmid assigned to this process.

Allows the kfd runtime to debug this process (A) via ptrace.
So you can use DBG_SET_TRAP ioctl in a debugger process (B) to debug process A.

```C
/**
// Enable modes for runtime enable
#define KFD_RUNTIME_ENABLE_MODE_ENABLE_MASK	1
#define KFD_RUNTIME_ENABLE_MODE_TTMP_SAVE_MASK	2

 * kfd_ioctl_runtime_enable_args - Arguments for runtime enable
 *
 * Coordinates debug exception signalling and debug device enablement with runtime.
 *
 * @r_debug - pointer to user struct for sharing information between ROCr and the debuggger
 * @mode_mask - mask to set mode
 *	KFD_RUNTIME_ENABLE_MODE_ENABLE_MASK - enable runtime for debugging, otherwise disable
 *	KFD_RUNTIME_ENABLE_MODE_TTMP_SAVE_MASK - enable trap temporary setup (ignore on disable)
 * @capabilities_mask - mask to notify runtime on what KFD supports
 *
 * Return - 0 on SUCCESS.
 *	  - EBUSY if runtime enable call already pending.
 *	  - EEXIST if user queues already active prior to call.
 *	    If process is debug enabled, runtime enable will enable debug devices and
 *	    wait for debugger process to send runtime exception EC_PROCESS_RUNTIME
 *	    to unblock - see kfd_ioctl_dbg_trap_args.
 *
 */
struct kfd_ioctl_runtime_enable_args {
	__u64 r_debug;
	__u32 mode_mask;
	__u32 capabilities_mask;
};
```

### r_debug
From what I can tell it's not used.

Perhaps it is used if the whole runtime_info struct (which holds r_debug) get's coppied to debugger process.

Theoretically it's a raw pointer to some user provided data.
Set to null on disable.

### Mode mask
- 0 bit: enable/disable debugging runtime
- 1 bit: ask to enable restoring ttmp's if supported

### capabilities_mask
Unused

## SET_TRAP_HANDLER
		AMDKFD_IOW(0x13, struct kfd_ioctl_set_trap_handler_args)

### Required Inputs
	__u64 tba_addr;		/* to KFD */
	__u64 tma_addr;		/* to KFD */
	__u32 gpu_id;		/* to KFD */

#### For dGPUs
Both `tba_addr` and `tma_addr` are addresses in **GPU memory space**

They must be 256 bytes aligned.

Remember to set EXECUTABLE flags for the memory.

#### For APUs
Remember to set READ | EXEC flag for the memory.

## DBG_REGISTER_DEPRECATED
		AMDKFD_IOW(0x0D, struct kfd_ioctl_dbg_register_args)

## DBG_UNREGISTER_DEPRECATED
		AMDKFD_IOW(0x0E, struct kfd_ioctl_dbg_unregister_args)

## DBG_ADDRESS_WATCH_DEPRECATED
		AMDKFD_IOW(0x0F, struct kfd_ioctl_dbg_address_watch_args)

## DBG_WAVE_CONTROL_DEPRECATED
		AMDKFD_IOW(0x10, struct kfd_ioctl_dbg_wave_control_args)

## DBG_TRAP
		AMDKFD_IOWR(0x26, struct kfd_ioctl_dbg_trap_args)

```C
/*
 * Debug operations
 *
 * For specifics on usage and return values, see documentation per operation
 * below.  Otherwise, generic error returns apply:
 *	- ESRCH if the process to debug does not exist.
 *
 *	- EINVAL (with KFD_IOC_DBG_TRAP_ENABLE exempt) if operation
 *		 KFD_IOC_DBG_TRAP_ENABLE has not succeeded prior.
 *		 Also returns this error if GPU hardware scheduling is not supported.
 *
 *	- EPERM (with KFD_IOC_DBG_TRAP_DISABLE exempt) if target process is not
 *		 PTRACE_ATTACHED.  KFD_IOC_DBG_TRAP_DISABLE is exempt to allow
 *		 clean up of debug mode as long as process is debug enabled.
 *
 *	- EACCES if any DBG_HW_OP (debug hardware operation) is requested when
 *		 AMDKFD_IOC_RUNTIME_ENABLE has not succeeded prior.
 *
 *	- ENODEV if any GPU does not support debugging on a DBG_HW_OP call.
 *
 *	- Other errors may be returned when a DBG_HW_OP occurs while the GPU
 *	  is in a fatal state.
 *
 */
enum kfd_dbg_trap_operations {
	KFD_IOC_DBG_TRAP_ENABLE = 0,
	KFD_IOC_DBG_TRAP_DISABLE = 1,
	KFD_IOC_DBG_TRAP_SEND_RUNTIME_EVENT = 2,
	KFD_IOC_DBG_TRAP_SET_EXCEPTIONS_ENABLED = 3,
	KFD_IOC_DBG_TRAP_SET_WAVE_LAUNCH_OVERRIDE = 4,  /* DBG_HW_OP */
	KFD_IOC_DBG_TRAP_SET_WAVE_LAUNCH_MODE = 5,      /* DBG_HW_OP */
	KFD_IOC_DBG_TRAP_SUSPEND_QUEUES = 6,		/* DBG_HW_OP */
	KFD_IOC_DBG_TRAP_RESUME_QUEUES = 7,		/* DBG_HW_OP */
	KFD_IOC_DBG_TRAP_SET_NODE_ADDRESS_WATCH = 8,	/* DBG_HW_OP */
	KFD_IOC_DBG_TRAP_CLEAR_NODE_ADDRESS_WATCH = 9,	/* DBG_HW_OP */
	KFD_IOC_DBG_TRAP_SET_FLAGS = 10,
	KFD_IOC_DBG_TRAP_QUERY_DEBUG_EVENT = 11,
	KFD_IOC_DBG_TRAP_QUERY_EXCEPTION_INFO = 12,
	KFD_IOC_DBG_TRAP_GET_QUEUE_SNAPSHOT = 13,
	KFD_IOC_DBG_TRAP_GET_DEVICE_SNAPSHOT = 14
};

/**
 * kfd_ioctl_dbg_trap_enable_args
 *
 *     Arguments for KFD_IOC_DBG_TRAP_ENABLE.
 *
 *     Enables debug session for target process. Call @op KFD_IOC_DBG_TRAP_DISABLE in
 *     kfd_ioctl_dbg_trap_args to disable debug session.
 *
 *     @exception_mask (IN)	- exceptions to raise to the debugger
 *     @rinfo_ptr      (IN)	- pointer to runtime info buffer (see kfd_runtime_info)
 *     @rinfo_size     (IN/OUT)	- size of runtime info buffer in bytes
 *     @dbg_fd	       (IN)	- fd the KFD will nofify the debugger with of raised
 *				  exceptions set in exception_mask.
 *
 *     Generic errors apply (see kfd_dbg_trap_operations).
 *     Return - 0 on SUCCESS.
 *		Copies KFD saved kfd_runtime_info to @rinfo_ptr on enable.
 *		Size of kfd_runtime saved by the KFD returned to @rinfo_size.
 *            - EBADF if KFD cannot get a reference to dbg_fd.
 *            - EFAULT if KFD cannot copy runtime info to rinfo_ptr.
 *            - EINVAL if target process is already debug enabled.
 *
 */
struct kfd_ioctl_dbg_trap_enable_args {
	__u64 exception_mask;
	__u64 rinfo_ptr;
	__u32 rinfo_size;
	__u32 dbg_fd;
};

/**
 * kfd_ioctl_dbg_trap_send_runtime_event_args
 *
 *
 *     Arguments for KFD_IOC_DBG_TRAP_SEND_RUNTIME_EVENT.
 *     Raises exceptions to runtime.
 *
 *     @exception_mask (IN) - exceptions to raise to runtime
 *     @gpu_id	       (IN) - target device id
 *     @queue_id       (IN) - target queue id
 *
 *     Generic errors apply (see kfd_dbg_trap_operations).
 *     Return - 0 on SUCCESS.
 *	      - ENODEV if gpu_id not found.
 *		If exception_mask contains EC_PROCESS_RUNTIME, unblocks pending
 *		AMDKFD_IOC_RUNTIME_ENABLE call - see kfd_ioctl_runtime_enable_args.
 *		All other exceptions are raised to runtime through err_payload_addr.
 *		See kfd_context_save_area_header.
 */
struct kfd_ioctl_dbg_trap_send_runtime_event_args {
	__u64 exception_mask;
	__u32 gpu_id;
	__u32 queue_id;
};

/**
 * kfd_ioctl_dbg_trap_set_exceptions_enabled_args
 *
 *     Arguments for KFD_IOC_SET_EXCEPTIONS_ENABLED
 *     Set new exceptions to be raised to the debugger.
 *
 *     @exception_mask (IN) - new exceptions to raise the debugger
 *
 *     Generic errors apply (see kfd_dbg_trap_operations).
 *     Return - 0 on SUCCESS.
 */
struct kfd_ioctl_dbg_trap_set_exceptions_enabled_args {
	__u64 exception_mask;
};

/**
 * kfd_ioctl_dbg_trap_set_wave_launch_override_args
 *
 *     Arguments for KFD_IOC_DBG_TRAP_SET_WAVE_LAUNCH_OVERRIDE
 *     Enable HW exceptions to raise trap.
 *
 *     @override_mode	     (IN)     - see kfd_dbg_trap_override_mode
 *     @enable_mask	     (IN/OUT) - reference kfd_dbg_trap_mask.
 *					IN is the override modes requested to be enabled.
 *					OUT is referenced in Return below.
 *     @support_request_mask (IN/OUT) - reference kfd_dbg_trap_mask.
 *					IN is the override modes requested for support check.
 *					OUT is referenced in Return below.
 *
 *     Generic errors apply (see kfd_dbg_trap_operations).
 *     Return - 0 on SUCCESS.
 *		Previous enablement is returned in @enable_mask.
 *		Actual override support is returned in @support_request_mask.
 *	      - EINVAL if override mode is not supported.
 *	      - EACCES if trap support requested is not actually supported.
 *		i.e. enable_mask (IN) is not a subset of support_request_mask (OUT).
 *		Otherwise it is considered a generic error (see kfd_dbg_trap_operations).
 */
struct kfd_ioctl_dbg_trap_set_wave_launch_override_args {
	__u32 override_mode;
	__u32 enable_mask;
	__u32 support_request_mask;
	__u32 pad;
};

/**
 * kfd_ioctl_dbg_trap_set_wave_launch_mode_args
 *
 *     Arguments for KFD_IOC_DBG_TRAP_SET_WAVE_LAUNCH_MODE
 *     Set wave launch mode.
 *
 *     @mode (IN) - see kfd_dbg_trap_wave_launch_mode
 *
 *     Generic errors apply (see kfd_dbg_trap_operations).
 *     Return - 0 on SUCCESS.
 */
struct kfd_ioctl_dbg_trap_set_wave_launch_mode_args {
	__u32 launch_mode;
	__u32 pad;
};

/**
 * kfd_ioctl_dbg_trap_suspend_queues_ags
 *
 *     Arguments for KFD_IOC_DBG_TRAP_SUSPEND_QUEUES
 *     Suspend queues.
 *
 *     @exception_mask	(IN) - raised exceptions to clear
 *     @queue_array_ptr (IN) - pointer to array of queue ids (u32 per queue id)
 *			       to suspend
 *     @num_queues	(IN) - number of queues to suspend in @queue_array_ptr
 *     @grace_period	(IN) - wave time allowance before preemption
 *			       per 1K GPU clock cycle unit
 *
 *     Generic errors apply (see kfd_dbg_trap_operations).
 *     Destruction of a suspended queue is blocked until the queue is
 *     resumed.  This allows the debugger to access queue information and
 *     the its context save area without running into a race condition on
 *     queue destruction.
 *     Automatically copies per queue context save area header information
 *     into the save area base
 *     (see kfd_queue_snapshot_entry and kfd_context_save_area_header).
 *
 *     Return - Number of queues suspended on SUCCESS.
 *	.	KFD_DBG_QUEUE_ERROR_MASK and KFD_DBG_QUEUE_INVALID_MASK masked
 *		for each queue id in @queue_array_ptr array reports unsuccessful
 *		suspend reason.
 *		KFD_DBG_QUEUE_ERROR_MASK = HW failure.
 *		KFD_DBG_QUEUE_INVALID_MASK = queue does not exist, is new or
 *		is being destroyed.
 */
struct kfd_ioctl_dbg_trap_suspend_queues_args {
	__u64 exception_mask;
	__u64 queue_array_ptr;
	__u32 num_queues;
	__u32 grace_period;
};

/**
 * kfd_ioctl_dbg_trap_resume_queues_args
 *
 *     Arguments for KFD_IOC_DBG_TRAP_RESUME_QUEUES
 *     Resume queues.
 *
 *     @queue_array_ptr (IN) - pointer to array of queue ids (u32 per queue id)
 *			       to resume
 *     @num_queues	(IN) - number of queues to resume in @queue_array_ptr
 *
 *     Generic errors apply (see kfd_dbg_trap_operations).
 *     Return - Number of queues resumed on SUCCESS.
 *		KFD_DBG_QUEUE_ERROR_MASK and KFD_DBG_QUEUE_INVALID_MASK mask
 *		for each queue id in @queue_array_ptr array reports unsuccessful
 *		resume reason.
 *		KFD_DBG_QUEUE_ERROR_MASK = HW failure.
 *		KFD_DBG_QUEUE_INVALID_MASK = queue does not exist.
 */
struct kfd_ioctl_dbg_trap_resume_queues_args {
	__u64 queue_array_ptr;
	__u32 num_queues;
	__u32 pad;
};

/**
 * kfd_ioctl_dbg_trap_set_node_address_watch_args
 *
 *     Arguments for KFD_IOC_DBG_TRAP_SET_NODE_ADDRESS_WATCH
 *     Sets address watch for device.
 *
 *     @address	(IN)  - watch address to set
 *     @mode    (IN)  - see kfd_dbg_trap_address_watch_mode
 *     @mask    (IN)  - watch address mask
 *     @gpu_id  (IN)  - target gpu to set watch point
 *     @id      (OUT) - watch id allocated
 *
 *     Generic errors apply (see kfd_dbg_trap_operations).
 *     Return - 0 on SUCCESS.
 *		Allocated watch ID returned to @id.
 *	      - ENODEV if gpu_id not found.
 *	      - ENOMEM if watch IDs can be allocated
 */
struct kfd_ioctl_dbg_trap_set_node_address_watch_args {
	__u64 address;
	__u32 mode;
	__u32 mask;
	__u32 gpu_id;
	__u32 id;
};

/**
 * kfd_ioctl_dbg_trap_clear_node_address_watch_args
 *
 *     Arguments for KFD_IOC_DBG_TRAP_CLEAR_NODE_ADDRESS_WATCH
 *     Clear address watch for device.
 *
 *     @gpu_id  (IN)  - target device to clear watch point
 *     @id      (IN) - allocated watch id to clear
 *
 *     Generic errors apply (see kfd_dbg_trap_operations).
 *     Return - 0 on SUCCESS.
 *	      - ENODEV if gpu_id not found.
 *	      - EINVAL if watch ID has not been allocated.
 */
struct kfd_ioctl_dbg_trap_clear_node_address_watch_args {
	__u32 gpu_id;
	__u32 id;
};

/**
 * kfd_ioctl_dbg_trap_set_flags_args
 *
 *     Arguments for KFD_IOC_DBG_TRAP_SET_FLAGS
 *     Sets flags for wave behaviour.
 *
 *     @flags (IN/OUT) - IN = flags to enable, OUT = flags previously enabled
 *
 *     Generic errors apply (see kfd_dbg_trap_operations).
 *     Return - 0 on SUCCESS.
 *	      - EACCESS if any debug device does not allow flag options.
 */
struct kfd_ioctl_dbg_trap_set_flags_args {
	__u32 flags;
	__u32 pad;
};

/**
 * kfd_ioctl_dbg_trap_query_debug_event_args
 *
 *     Arguments for KFD_IOC_DBG_TRAP_QUERY_DEBUG_EVENT
 *
 *     Find one or more raised exceptions. This function can return multiple
 *     exceptions from a single queue or a single device with one call. To find
 *     all raised exceptions, this function must be called repeatedly until it
 *     returns -EAGAIN. Returned exceptions can optionally be cleared by
 *     setting the corresponding bit in the @exception_mask input parameter.
 *     However, clearing an exception prevents retrieving further information
 *     about it with KFD_IOC_DBG_TRAP_QUERY_EXCEPTION_INFO.
 *
 *     @exception_mask (IN/OUT) - exception to clear (IN) and raised (OUT)
 *     @gpu_id	       (OUT)    - gpu id of exceptions raised
 *     @queue_id       (OUT)    - queue id of exceptions raised
 *
 *     Generic errors apply (see kfd_dbg_trap_operations).
 *     Return - 0 on raised exception found
 *              Raised exceptions found are returned in @exception mask
 *              with reported source id returned in @gpu_id or @queue_id.
 *            - EAGAIN if no raised exception has been found
 */
struct kfd_ioctl_dbg_trap_query_debug_event_args {
	__u64 exception_mask;
	__u32 gpu_id;
	__u32 queue_id;
};

/**
 * kfd_ioctl_dbg_trap_query_exception_info_args
 *
 *     Arguments KFD_IOC_DBG_TRAP_QUERY_EXCEPTION_INFO
 *     Get additional info on raised exception.
 *
 *     @info_ptr	(IN)	 - pointer to exception info buffer to copy to
 *     @info_size	(IN/OUT) - exception info buffer size (bytes)
 *     @source_id	(IN)     - target gpu or queue id
 *     @exception_code	(IN)     - target exception
 *     @clear_exception	(IN)     - clear raised @exception_code exception
 *				   (0 = false, 1 = true)
 *
 *     Generic errors apply (see kfd_dbg_trap_operations).
 *     Return - 0 on SUCCESS.
 *              If @exception_code is EC_DEVICE_MEMORY_VIOLATION, copy @info_size(OUT)
 *		bytes of memory exception data to @info_ptr.
 *              If @exception_code is EC_PROCESS_RUNTIME, copy saved
 *              kfd_runtime_info to @info_ptr.
 *              Actual required @info_ptr size (bytes) is returned in @info_size.
 */
struct kfd_ioctl_dbg_trap_query_exception_info_args {
	__u64 info_ptr;
	__u32 info_size;
	__u32 source_id;
	__u32 exception_code;
	__u32 clear_exception;
};

/**
 * kfd_ioctl_dbg_trap_get_queue_snapshot_args
 *
 *     Arguments KFD_IOC_DBG_TRAP_GET_QUEUE_SNAPSHOT
 *     Get queue information.
 *
 *     @exception_mask	 (IN)	  - exceptions raised to clear
 *     @snapshot_buf_ptr (IN)	  - queue snapshot entry buffer (see kfd_queue_snapshot_entry)
 *     @num_queues	 (IN/OUT) - number of queue snapshot entries
 *         The debugger specifies the size of the array allocated in @num_queues.
 *         KFD returns the number of queues that actually existed. If this is
 *         larger than the size specified by the debugger, KFD will not overflow
 *         the array allocated by the debugger.
 *
 *     @entry_size	 (IN/OUT) - size per entry in bytes
 *         The debugger specifies sizeof(struct kfd_queue_snapshot_entry) in
 *         @entry_size. KFD returns the number of bytes actually populated per
 *         entry. The debugger should use the KFD_IOCTL_MINOR_VERSION to determine,
 *         which fields in struct kfd_queue_snapshot_entry are valid. This allows
 *         growing the ABI in a backwards compatible manner.
 *         Note that entry_size(IN) should still be used to stride the snapshot buffer in the
 *         event that it's larger than actual kfd_queue_snapshot_entry.
 *
 *     Generic errors apply (see kfd_dbg_trap_operations).
 *     Return - 0 on SUCCESS.
 *              Copies @num_queues(IN) queue snapshot entries of size @entry_size(IN)
 *              into @snapshot_buf_ptr if @num_queues(IN) > 0.
 *              Otherwise return @num_queues(OUT) queue snapshot entries that exist.
 */
struct kfd_ioctl_dbg_trap_queue_snapshot_args {
	__u64 exception_mask;
	__u64 snapshot_buf_ptr;
	__u32 num_queues;
	__u32 entry_size;
};

/**
 * kfd_ioctl_dbg_trap_get_device_snapshot_args
 *
 *     Arguments for KFD_IOC_DBG_TRAP_GET_DEVICE_SNAPSHOT
 *     Get device information.
 *
 *     @exception_mask	 (IN)	  - exceptions raised to clear
 *     @snapshot_buf_ptr (IN)	  - pointer to snapshot buffer (see kfd_dbg_device_info_entry)
 *     @num_devices	 (IN/OUT) - number of debug devices to snapshot
 *         The debugger specifies the size of the array allocated in @num_devices.
 *         KFD returns the number of devices that actually existed. If this is
 *         larger than the size specified by the debugger, KFD will not overflow
 *         the array allocated by the debugger.
 *
 *     @entry_size	 (IN/OUT) - size per entry in bytes
 *         The debugger specifies sizeof(struct kfd_dbg_device_info_entry) in
 *         @entry_size. KFD returns the number of bytes actually populated. The
 *         debugger should use KFD_IOCTL_MINOR_VERSION to determine, which fields
 *         in struct kfd_dbg_device_info_entry are valid. This allows growing the
 *         ABI in a backwards compatible manner.
 *         Note that entry_size(IN) should still be used to stride the snapshot buffer in the
 *         event that it's larger than actual kfd_dbg_device_info_entry.
 *
 *     Generic errors apply (see kfd_dbg_trap_operations).
 *     Return - 0 on SUCCESS.
 *              Copies @num_devices(IN) device snapshot entries of size @entry_size(IN)
 *              into @snapshot_buf_ptr if @num_devices(IN) > 0.
 *              Otherwise return @num_devices(OUT) queue snapshot entries that exist.
 */
struct kfd_ioctl_dbg_trap_device_snapshot_args {
	__u64 exception_mask;
	__u64 snapshot_buf_ptr;
	__u32 num_devices;
	__u32 entry_size;
};

/**
 * kfd_ioctl_dbg_trap_args
 *
 * Arguments to debug target process.
 *
 *     @pid - target process to debug
 *     @op  - debug operation (see kfd_dbg_trap_operations)
 *
 *     @op determines which union struct args to use.
 *     Refer to kern docs for each kfd_ioctl_dbg_trap_*_args struct.
 */
struct kfd_ioctl_dbg_trap_args {
	__u32 pid;
	__u32 op;

	union {
		struct kfd_ioctl_dbg_trap_enable_args enable;
		struct kfd_ioctl_dbg_trap_send_runtime_event_args send_runtime_event;
		struct kfd_ioctl_dbg_trap_set_exceptions_enabled_args set_exceptions_enabled;
		struct kfd_ioctl_dbg_trap_set_wave_launch_override_args launch_override;
		struct kfd_ioctl_dbg_trap_set_wave_launch_mode_args launch_mode;
		struct kfd_ioctl_dbg_trap_suspend_queues_args suspend_queues;
		struct kfd_ioctl_dbg_trap_resume_queues_args resume_queues;
		struct kfd_ioctl_dbg_trap_set_node_address_watch_args set_node_address_watch;
		struct kfd_ioctl_dbg_trap_clear_node_address_watch_args clear_node_address_watch;
		struct kfd_ioctl_dbg_trap_set_flags_args set_flags;
		struct kfd_ioctl_dbg_trap_query_debug_event_args query_debug_event;
		struct kfd_ioctl_dbg_trap_query_exception_info_args query_exception_info;
		struct kfd_ioctl_dbg_trap_queue_snapshot_args queue_snapshot;
		struct kfd_ioctl_dbg_trap_device_snapshot_args device_snapshot;
	};
};
```
