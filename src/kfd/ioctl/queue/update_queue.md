# update_queue
		AMDKFD_IOW(0x07, struct kfd_ioctl_update_queue_args)

## Required Inputs
### Queue Identifier
	__u32 queue_id;		/* to KFD */

### Ring buffer
	__u64 ring_base_address;	/* to KFD */
	__u32 ring_size;		/* to KFD */

It accepts a null base_address to disable this queue.

You can resize the buffer or use a new one, keeping in mind size requirements.

Take note the rptr_addr and wptr_addr stay the same.

### Percentage
	__u32 queue_percentage;	/* to KFD */

{{#include percentage.md}}

### Priority
	__u32 queue_priority;	/* to KFD */

{{#include priority.md}}
