# create_queue
		AMDKFD_IOWR(0x02, struct kfd_ioctl_create_queue_args)

#### Does it require IOMMU to be enabled in bios?

## Required Inputs

```C
__u32 gpu_id;		/* to KFD */
```

### Queue type
```C
__u32 queue_type;		/* to KFD */
```
* compute - 0x0, pm4 compute commands
* sdma - 0x1, pcie optimized SDMA queue, pm4 format
* compute_aql - 0x2, aql compute commands
* sdma_xgmi - 0x3, non-pci optimized SDMA queue, pm4 format
* sdma_by_eng_id - 0x4, manually pick sdma engine for this queue, pm4 format

### User allocated ring buffer
```C
__u64 ring_base_address;	/* to KFD */
__u64 write_pointer_address;	/* to KFD */
__u64 read_pointer_address;	/* to KFD */
__u32 ring_size;		/* to KFD */
```
Size must be a power of 2 and at least 1024.
Size is in bytes but remember the ring buffer is an array of u32 values.

Buffer must be 256 bytes aligned.

#### What is the type of value the rptr and wptr are pointing to?
Perhaps it's actually indicees these pointers are dereferenced to u32 in create_queue kernel side implementation.
The size of the buffer is also in u32 and it would make sense
to do arithmetic on indicees rather than raw addresses.

#### Is the rptr and wptr guaranteed to be accessed by only one thread?

Wptr is the location commands can be written from.
So the region from `[rptr, wptr - 1]` inclusive is reserved to be read by the gpu.

The driver is going to modify the read_pointer as it consumes the commands from the buffer.
Buffer is idle when *rptr == *wptr.

#### Who is responsible for deallocating that memory and what must happen first?

#### How is this buffer synchronized with?


### Queue Percentage
`__u32 queue_percentage;	/* to KFD */`

It's actually split into two 8bit values.

* bit 0-7: queue percentage from 0 to 100.
* bit 8-15: pm4_target_xcc - XCC's id when gpu is split into multiple, only for PM4 queue

#### What does the percentage represent, what effect does it have?
I believe it's to specify how full the buffer should be before the kernel
should start executing commands from it, this way it's more efficient.

But wouldn't that mean commands don't get executed untill this percentage is reached?


### Queue Priority
`__u32 queue_priority;	/* to KFD */`

Value from 0 to 15 (0xf), max prio at 15.

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
