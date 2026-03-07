# SMI

Creates an opened file descriptor for listening to system events
specific to this process or all processess.

You can read from the fd to get events in text form
One event per line.
Starting with a hex value without 0x prefix for event type.
After a space you'd use a corresponing format to sscanf
based on the type to decode the event.

You can write to the fd to set a filter which events you
wish to receive.
Notice the filter is a 64bit value split into 8 bytes using
system native endianess.
Where bit at position X means that events with type X will be reported.

You can poll the fd to wait until events are available to read.

Underneath it uses a FIFO buffer 8192 bytes in size.


## SMI_EVENTS
    AMDKFD_IOWR(0x1F, struct kfd_ioctl_smi_events_args)

```C
/*
 * KFD SMI(System Management Interface) events
 */
enum kfd_smi_event {
	KFD_SMI_EVENT_NONE = 0, /* not used */
	KFD_SMI_EVENT_VMFAULT = 1, /* event start counting at 1 */
	KFD_SMI_EVENT_THERMAL_THROTTLE = 2,
	KFD_SMI_EVENT_GPU_PRE_RESET = 3,
	KFD_SMI_EVENT_GPU_POST_RESET = 4,
	KFD_SMI_EVENT_MIGRATE_START = 5,
	KFD_SMI_EVENT_MIGRATE_END = 6,
	KFD_SMI_EVENT_PAGE_FAULT_START = 7,
	KFD_SMI_EVENT_PAGE_FAULT_END = 8,
	KFD_SMI_EVENT_QUEUE_EVICTION = 9,
	KFD_SMI_EVENT_QUEUE_RESTORE = 10,
	KFD_SMI_EVENT_UNMAP_FROM_GPU = 11,
	KFD_SMI_EVENT_PROCESS_START = 12,
	KFD_SMI_EVENT_PROCESS_END = 13,

	/*
	 * max event number, as a flag bit to get events from all processes,
	 * this requires super user permission, otherwise will not be able to
	 * receive event from any process. Without this flag to receive events
	 * from same process.
	 */
	KFD_SMI_EVENT_ALL_PROCESS = 64
};

/* The reason of the page migration event */
enum KFD_MIGRATE_TRIGGERS {
	KFD_MIGRATE_TRIGGER_PREFETCH,		/* Prefetch to GPU VRAM or system memory */
	KFD_MIGRATE_TRIGGER_PAGEFAULT_GPU,	/* GPU page fault recover */
	KFD_MIGRATE_TRIGGER_PAGEFAULT_CPU,	/* CPU page fault recover */
	KFD_MIGRATE_TRIGGER_TTM_EVICTION	/* TTM eviction */
};

/* The reason of user queue evition event */
enum KFD_QUEUE_EVICTION_TRIGGERS {
	KFD_QUEUE_EVICTION_TRIGGER_SVM,		/* SVM buffer migration */
	KFD_QUEUE_EVICTION_TRIGGER_USERPTR,	/* userptr movement */
	KFD_QUEUE_EVICTION_TRIGGER_TTM,		/* TTM move buffer */
	KFD_QUEUE_EVICTION_TRIGGER_SUSPEND,	/* GPU suspend */
	KFD_QUEUE_EVICTION_CRIU_CHECKPOINT,	/* CRIU checkpoint */
	KFD_QUEUE_EVICTION_CRIU_RESTORE		/* CRIU restore */
};

/* The reason of unmap buffer from GPU event */
enum KFD_SVM_UNMAP_TRIGGERS {
	KFD_SVM_UNMAP_TRIGGER_MMU_NOTIFY,	/* MMU notifier CPU buffer movement */
	KFD_SVM_UNMAP_TRIGGER_MMU_NOTIFY_MIGRATE,/* MMU notifier page migration */
	KFD_SVM_UNMAP_TRIGGER_UNMAP_FROM_CPU	/* Unmap to free the buffer */
};

#define KFD_SMI_EVENT_MASK_FROM_INDEX(i) (1ULL << ((i) - 1))
#define KFD_SMI_EVENT_MSG_SIZE	96

struct kfd_ioctl_smi_events_args {
	__u32 gpuid;	/* to KFD */
	__u32 anon_fd;	/* from KFD */
};
```
