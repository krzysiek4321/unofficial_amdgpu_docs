# Scheduling commands to gpus with User Queues

**These are different from DRM's UserQ**

They exist to reduce ioctl communication to schedule work to the gpu.

They are scheduled to hardware pipes.

You first allocate memory for the queue, then you create it, then you wait for events
signaled by your gpu commands, meanwhile you write new commands to the ring buffer and notify the gpu by writing to the doorbell corresponding to the created queue.

You can set a mask to tell the gpu which CU you wish to have your gpu kernels to run on.

## Properties
### Ring Buffer
Size must be a power of 2 and at least 1024.
Size is in bytes but remember the ring buffer is an array of u32 values.

Buffer must be 256 bytes aligned, becuase the address is passed to the gpu shifted right by 8.

Buffer and rptr and wptr must be already mapped to buffer object (BO).
The kernel does a lookup for Virtual Address (VA) mapping.

`kfd_queue_acquire_buffers()` requires rptr and wptr to be mapped to exactly one gpu memory page (4096 bytes).
It cannot be a part of a larger allocation.
But I believe we can pack both of them and even ring buffer into one page if `size < 4096`.

#### What is the type of value the rptr and wptr are pointing to?
Perhaps it's actually indicees these pointers are dereferenced to u32 in create_queue kernel side implementation.

They must be indicees because update_queue doesn't change the addresses of these.

The size of the buffer is also in u32 and it would make sense
to do arithmetic on indicees rather than raw addresses.

#### Is the rptr and wptr guaranteed to be accessed by only one thread?
Don't know yet.

Wptr is the location commands can be written from.
So the region from `[rptr, wptr - 1]` inclusive is reserved to be read by the gpu.

The driver is going to modify the read_pointer as it consumes the commands from the buffer.
Buffer is idle when *rptr == *wptr.

### Queue Type
* compute - 0x0, pm4 compute commands
* sdma - 0x1, pcie optimized SDMA queue, pm4 format
* compute_aql - 0x2, aql compute commands
* sdma_xgmi - 0x3, non-pci optimized SDMA queue, pm4 format
* sdma_by_eng_id - 0x4, manually pick sdma engine for this queue, pm4 format

### Queue Percentage
A u32 value is actually split into two 8bit values.

* bit 0-7: queue percentage from 0 to 100.
* bit 8-15: pm4_target_xcc - XCC's id when gpu is split into multiple, only for PM4 queue

#### What does the percentage represent, what effect does it have?
I believe it's to specify how full the buffer should be before the kernel
should start executing commands from it, this way it's more efficient.

But wouldn't that mean commands don't get executed untill this percentage is reached?

### Queue Priority
`__u32 queue_priority;	/* to KFD */`

Value from 0 to 15 (0xf), max prio at 15.

### Doorbell offset
`__u64 doorbell_offset;	/* from KFD */`

For gpu's no older than gfx901 (IS_SOC15) it includes relative offset into a doorbells page.

#### How do I use this offset with mmap? What size of memory should be mapped, 1 uint32_t?

## Doorbells
There is a maximum of 1024 queues per process.
Each is assigned a doorbell.

They are automatically created with queues.

### Size
Doorbell size is device dependent.
For < gfx901 it's 4 bytes.
For gfx901+ it's 8 bytes.

So mapping `mmap()` would need to be `2 * PAGE_SIZE` in size for gfx901+ and `PAGE_SIZE` for older engines.

#### Why are doorbells 8 bytes for all newer gpu if a queue has size in u32 and `*wptr` is an index?

### Index
#### How can I tell which address from the mmap doorbells page or pages to write the new wptr to?
Is it as simple as just `idx = offset & SIZE`?

### Whais is it for?
It's purpose is to notify the gpu when we wrote new commands into a queue.
We write the new "wptr" value into a doorbell for a given queue.

### bitmap
It's 1024 bits, split into 2 512 bit parts, the seccond called **mirror**, set the same way first part is.

## Usage patterns
Todo

## Questions to the reader

### Does it require IOMMU to be enabled in bios?

### Can it be directly created from any memory in programs address space?

### Who is responsible for deallocating that memory and what must happen first?

### How is this buffer synchronized with?

## IOCTLs
### create_queue
		AMDKFD_IOWR(0x02, struct kfd_ioctl_create_queue_args)

#### Required Inputs

```C
__u32 gpu_id;		/* to KFD */
__u32 queue_type;		/* to KFD */
__u32 queue_percentage;	/* to KFD */
__u32 queue_priority;	/* to KFD */
```

##### Ring buffer
```C
__u64 ring_base_address;	/* to KFD */
__u64 write_pointer_address;	/* to KFD */
__u64 read_pointer_address;	/* to KFD */
__u32 ring_size;		/* to KFD */
```


#### Conditional Inputs

##### End Of Pipe (EOP) buffer
```C
__u64 eop_buffer_address;	/* to KFD */
__u64 eop_buffer_size;	/* to KFD */
```
Not required.
It's used to submit commands to GPU to be executed after a shader finishes and caches get flushed.
Size must be appropriate for the selected gpu.

##### Save-restore buffer
```C
__u64 ctx_save_restore_address; /* to KFD */
__u32 ctx_save_restore_size;	/* to KFD */
```
Required only for `compute*` queues.

It must be user accessible address and it must have a mapping to a bo.

Size must be `>= node.ctl_stack_size + node.wg_data_size`.

Actual BO size must be larger and equal to
`size + debug_memory_size * num_of_XCC` rounded up to `PAGE_SIZE`.

Look in `kfd_queue_ctx_save_restore_size()` to see how the values above are determined.

###### How is it used?
todo

##### SDMA engine id
`__u32 sdma_engine_id;		/* to KFD */`

Used when queue type is `sdma_by_eng_id`.
Used as a performance tweek for high end gpu's split with xGMI.
It allow to specify a preffered sdma engine to be used for this queue
which remember is tied to a specific gpu.

##### Ctl stack size
```C
__u32 ctl_stack_size;		/* to KFD */
```
Required only for queue type `compute*`.
Must be equal to selected node's ctl_stack_size.

#### Outputs
##### Queue Id
`__u32 queue_id;		/* from KFD */`

An Id unique for the process, which opened the kfd file.

##### Doorbell offset
`__u64 doorbell_offset;	/* from KFD */`

For gpu's no older than gfx901 (IS_SOC15) it includes relative offset into a doorbells page.

###### How do I use this offset with mmap? What size of memory should be mapped, 1 uint32_t?

### destroy_queue
    AMDKFD_IOWR(0x03, struct kfd_ioctl_destroy_queue_args)

#### Required Inputs
	__u32 queue_id;		/* to KFD */

### update_queue
    AMDKFD_IOW(0x07, struct kfd_ioctl_update_queue_args)

#### Required Inputs
	__u32 queue_id;		/* to KFD */
	__u32 queue_percentage;	/* to KFD */
	__u32 queue_priority;	/* to KFD */

##### Ring buffer
	__u64 ring_base_address;	/* to KFD */
	__u32 ring_size;		/* to KFD */

It accepts a null base_address to disable this queue.

You can resize the buffer or use a new one, keeping in mind size requirements.

Take note the rptr_addr and wptr_addr stay the same.

### set_cu_mask
    AMDKFD_IOW(0x1A, struct kfd_ioctl_set_cu_mask_args)

#### Inputs
	__u32 queue_id;		/* to KFD */
	__u32 num_cu_mask;		/* to KFD */
	__u64 cu_mask_ptr;		/* to KFD */

**num_cu_mask** must be multiple of 32, because its unit is bit count and mask elements are uint32

### get_queue_wave_state
    AMDKFD_IOWR(0x1B, struct kfd_ioctl_get_queue_wave_state_args)

### alloc_queue_gws
    AMDKFD_IOWR(0x1E, struct kfd_ioctl_alloc_queue_gws_args)
