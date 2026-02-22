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
