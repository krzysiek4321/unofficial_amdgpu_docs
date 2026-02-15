# create_queue
		AMDKFD_IOWR(0x02, struct kfd_ioctl_create_queue_args)

## Required Inputs

```C
__u32 gpu_id;		/* to KFD */
```

### Queue type
```C
__u32 queue_type;		/* to KFD */
```
{{#include type.md}}

### User allocated ring buffer
```C
__u64 ring_base_address;	/* to KFD */
__u64 write_pointer_address;	/* to KFD */
__u64 read_pointer_address;	/* to KFD */
__u32 ring_size;		/* to KFD */
```

### Queue Percentage
`__u32 queue_percentage;	/* to KFD */`

{{#include percentage.md}}

### Queue Priority
`__u32 queue_priority;	/* to KFD */`

{{#include priority.md}}

## Conditional Inputs

### End Of Pipe (EOP) buffer
```C
__u64 eop_buffer_address;	/* to KFD */
__u64 eop_buffer_size;	/* to KFD */
```
Not required.
It's used to submit commands to GPU to be executed after a shader finishes and caches get flushed.
Size must be appropriate for the selected gpu.

### Criu save-restore buffer
```C
__u64 ctx_save_restore_address; /* to KFD */
__u32 ctx_save_restore_size;	/* to KFD */
```
Required only for `compute*` queues.
Size must be sufficiently large for a given node.

Actual size is going to be larger to account for debug buffers per each XCC and aligned to PAGE_SIZE.

### SDMA engine id
`__u32 sdma_engine_id;		/* to KFD */`

Used when queue type is `sdma_by_eng_id`.
Used as a performance tweek for high end gpu's split with xGMI.
It allow to specify a preffered sdma engine to be used for this queue
which remember is tied to a specific gpu.

### Ctl stack size
```C
__u32 ctl_stack_size;		/* to KFD */
```
Required only for queue type `compute*`.
Must be equal to selected node's ctl_stack_size.

## Outputs
### Queue Id
`__u32 queue_id;		/* from KFD */`

An Id unique for the process, which opened the kfd file.

### Doorbell offset
`__u64 doorbell_offset;	/* from KFD */`

For gpu's no older than gfx901 (IS_SOC15) it includes relative offset into a doorbells page.

#### How do I use this offset with mmap? What size of memory should be mapped, 1 uint32_t?
