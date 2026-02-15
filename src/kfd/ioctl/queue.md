# User Queues

**These are different from DRM's UserQ**

They exist to reduce ioctl communication to schedule work to the gpu.

They are scheduled to hardware pipes.

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
{{#include queue/type.md}}

### Queue Percentage
A u32 value is actually split into two 8bit values.

{{#include queue/percentage.md}}

#### What does the percentage represent, what effect does it have?
I believe it's to specify how full the buffer should be before the kernel
should start executing commands from it, this way it's more efficient.

But wouldn't that mean commands don't get executed untill this percentage is reached?

### Queue Priority
`__u32 queue_priority;	/* to KFD */`

{{#include queue/priority.md}}

### Doorbell offset
`__u64 doorbell_offset;	/* from KFD */`

For gpu's no older than gfx901 (IS_SOC15) it includes relative offset into a doorbells page.

#### How do I use this offset with mmap? What size of memory should be mapped, 1 uint32_t?

## Usage patterns
Todo

## Questions to the reader

### Does it require IOMMU to be enabled in bios?

### Can it be directly created from any memory in programs address space?

### Who is responsible for deallocating that memory and what must happen first?

### How is this buffer synchronized with?
