# Events
These signals can be created in response to firmware messages via `->interrupt_wq()` or by kernel module
in certain situations.

It then searches for all event with the specific type, populates the appropriate data in all of them and marks them
for waiters.

Be aware these events are hard to tie to specific gpu actions or commands.

`kfd_signal_poison_consumed_event()` will send SIGBUS to the process.


## Types
These are userspace exposed types.

```C
#define KFD_IOC_EVENT_SIGNAL		0
#define KFD_IOC_EVENT_NODECHANGE		1
#define KFD_IOC_EVENT_DEVICESTATECHANGE	2
#define KFD_IOC_EVENT_HW_EXCEPTION		3
#define KFD_IOC_EVENT_SYSTEM_EVENT		4
#define KFD_IOC_EVENT_DEBUG_EVENT		5
#define KFD_IOC_EVENT_PROFILE_EVENT		6
#define KFD_IOC_EVENT_QUEUE_EVENT		7
#define KFD_IOC_EVENT_MEMORY		8
```

### Actually used types
These are types actually used in kernel module code with known data layout in [WAIT_EVENTS](#wait_events).

#### SIGNAL, DEBUG_EVENT
These are the kfd's version of **fences**.

Signaled with `kfd_signal_event_interrupt()` in kernel, generally it's either CP_END_OF_PIPE, SDMA_TRAP or SQ_INTERRUPT_MSG.

#### HW_EXCEPTION
Signaled with `kfd_signal_hw_exception_event()`, on BAD_OPCODE

#### MEMORY
Signaled with `kfd_signal_vm_fault_event()`, on GFX_PAGE_INV_FAULT and GFX_MEM_PROT_FAULT

## Special event id = 0
Created by the kernel so please don't destroy it.

It is used for a fast path to ignore bogus events
that are sent by the Command Processor (CP) without a context ID (a partial event id).

## Waiting for events
When using [WAIT_EVENTS](#wait_events) event waiters are created for each event_id submitted in the ioctl.

You can mark if you want this to return when all the events are signalled or at least one.

The event waiters are then woken up dynamically.

## Event age
A u64 property.
- 0 - reserved, should not be used
- 1 - default, used during event creation
- 2... - used by set_event, by incrementing previous age and wrapping back to 2

## Signal page
It's 4096 * 8 bytes in size.
So 4096 u64 values.

Value of -1 means **unsignalled**.

It's only used by SIGNAL and DEBUG events.

It's alloced either by the user on the GPU in GTT domain and passed in [CREATE_EVENT](#create_event) or automatically created in cpu kernel space
but the kernel is going to see it as if there is only 256 slots.

The underlying BO also gets pinned to GTT.

`Page[event_id] = ...`

The signal**er** will **write 1** into slots he wishes to signal before sending an interrupt to the process.

Can be [mmaped](../mmap.md#2---events).

## Signal events

### How can I tell the gpu to signal a partiluar event_id?
The signal page must be manually created in GTT domain and VA mapped.
For these to work.

Generally grep for `ring_emit_fence` and `INT_SEL`.

#### From RDNA code
Depending on gpu generation it passes 8, 23 or 24 bits from event_id.
```
v_mov_b32 v0, $ADDR_LOW(SIGNAL_PAGE + event_id)
v_mov_b32 v1, $ADDR_HI(SIGNAL_PAGE + event_id)
v_mov_b32 v2, 1
v_mov_b32 v3, 0
global_store_dwordx2 v[0:1], v[2:3], off

s_waitcnt 0

s_mov m0, $EVENT_ID
s_sendmsg sendmsg(MSG_INTERRUPT)
```

#### From SDMA commands written to ring buffer
It passes 28 bits from event_id.

```C
    // SDMA v5.2
	amdgpu_ring_write(ring, SDMA_PKT_HEADER_OP(SDMA_OP_FENCE) |
			  SDMA_PKT_FENCE_HEADER_MTYPE(0x3)); /* Ucached(UC) */
	amdgpu_ring_write(ring, lower_32_bits(signal_page + event_id));
	amdgpu_ring_write(ring, upper_32_bits(signal_page + event_id));
	amdgpu_ring_write(ring, lower_32_bits(1));

    amdgpu_ring_write(ring, SDMA_PKT_HEADER_OP(SDMA_OP_FENCE) |
              SDMA_PKT_FENCE_HEADER_MTYPE(0x3));
    amdgpu_ring_write(ring, lower_32_bits(signal_page + event_id + 4));
    amdgpu_ring_write(ring, upper_32_bits(signal_page + event_id + 4));
    amdgpu_ring_write(ring, upper_32_bits(0));

    /* generate an interrupt */
    amdgpu_ring_write(ring, SDMA_PKT_HEADER_OP(SDMA_OP_TRAP));
    amdgpu_ring_write(ring, SDMA_PKT_TRAP_INT_CONTEXT_INT_CONTEXT(event_id));
```

#### From compute ring buffer
It passes 28 bits from event_id.

Notice it writes event_id into `signal_page[event_id]`, because there is no mechanism to
provide a separate argument for the interrupt.

##### Via PACKET3_EVENT_WRITE_EOP
```C
    void* addr = signal_page + event_id;
	amdgpu_ring_write(ring, PACKET3(PACKET3_EVENT_WRITE_EOP, 4));
	amdgpu_ring_write(ring, (EOP_TCL1_ACTION_EN |
				 EOP_TC_ACTION_EN |
				 EOP_TC_WB_ACTION_EN |
				 EVENT_TYPE(CACHE_FLUSH_AND_INV_TS_EVENT) |
				 EVENT_INDEX(5) |
				 (exec ? EOP_EXEC : 0)));
	amdgpu_ring_write(ring, addr & 0xfffffffc);
	amdgpu_ring_write(ring, (upper_32_bits(addr) & 0xffff) |
			  DATA_SEL(2) | INT_SEL(2));
	amdgpu_ring_write(ring, event_id);
	amdgpu_ring_write(ring, 0);
```
##### Via PACKET3_RELEASE_MEM
Since gfx9
```C
    void* addr = signal_page + event_id;
	amdgpu_ring_write(ring, PACKET3(PACKET3_RELEASE_MEM, 6));
	amdgpu_ring_write(ring, (PACKET3_RELEASE_MEM_GCR_SEQ |
				 PACKET3_RELEASE_MEM_GCR_GL2_WB |
				 PACKET3_RELEASE_MEM_GCR_GLM_INV | /* must be set with GLM_WB */
				 PACKET3_RELEASE_MEM_GCR_GLM_WB |
				 PACKET3_RELEASE_MEM_CACHE_POLICY(3) |
				 PACKET3_RELEASE_MEM_EVENT_TYPE(CACHE_FLUSH_AND_INV_TS_EVENT) |
				 PACKET3_RELEASE_MEM_EVENT_INDEX(5)));
	amdgpu_ring_write(ring, (PACKET3_RELEASE_MEM_DATA_SEL(2) |
				 PACKET3_RELEASE_MEM_INT_SEL(2)));
	amdgpu_ring_write(ring, lower_32_bits(addr));
	amdgpu_ring_write(ring, upper_32_bits(addr));
	amdgpu_ring_write(ring, event_id);
	amdgpu_ring_write(ring, 0);
	amdgpu_ring_write(ring, 0);
```

## IOCTLs

### CREATE_EVENT
		AMDKFD_IOWR(0x08, struct kfd_ioctl_create_event_args)

#### Inputs
```C
__u32 event_type;		/* to KFD */
__u32 auto_reset;		/* to KFD */
__u32 node_id;		/* to KFD - only valid for certain
                        event types */
__u64 event_page_offset;	/* to KFD - only for dGPU
bits 31:0 - BO handle to be used as signal_page for signal events
bits 63:32 - gpu_id
*/
```
auto_reset automatically resets events without waiters.

The BO must be created in GTT domain.
Also please make sure to make it large enough (4096 * 8 bytes).

You are passing ownership of this BO here and freeing it is not allowed.

Also you have to make sure you pass a BO only once during first `CREATE_EVENT` call.

But you can also leave it empty and the memory will be allocated in kernel space,
but it will not be accessible to the gpu.

##### What is node_id for?

##### Is using smaller size going to produce a kernel module bug?

#### Outputs
```C
__u64 event_page_offset;	/* from KFD*/
__u32 event_trigger_data;	/* from KFD - signal events only */
__u32 event_id;		/* from KFD */
__u32 event_slot_index;	/* from KFD - the same as event_id */
```

You can use `event_page_offset` in [mmap](../mmap.md#2---events).

- ENOSPC - no slot available in signal_page
- ENOMEM - no memory to allocate singnal_page or no memory to copy user data into
- EINVAL - signal_page is already set
    or gpu not found
    or provided bo is invalid
    or there is a problem with BO flags, check dmesg output

### DESTROY_EVENT
		AMDKFD_IOW(0x09, struct kfd_ioctl_destroy_event_args)

#### Input
	__u32 event_id;		/* to KFD */

Returns:
- EINVAL - if event with provided id was not found

### SET_EVENT
		AMDKFD_IOW(0x0A, struct kfd_ioctl_set_event_args)

Increses event age by one wrapping around u64 to 2.

Wakes up all waiters.
You can read the new age in data retured from [WAIT_EVENTS] for SIGNAL events.

#### Input
	__u32 event_id;		/* to KFD */

The event must have type SIGNAL.

Returns:
- EINVAL - if event not found

### RESET_EVENT
		AMDKFD_IOW(0x0B, struct kfd_ioctl_reset_event_args)

Reset the event to not signalled state.

#### Input
	__u32 event_id;		/* to KFD */

- EINVAL - if event not found or the event type is not SIGNAL

### WAIT_EVENTS
		AMDKFD_IOWR(0x0C, struct kfd_ioctl_wait_events_args)

```C
struct kfd_memory_exception_failure {
	__u32 NotPresent;	/* Page not present or supervisor privilege */
	__u32 ReadOnly;	/* Write access to a read-only page */
	__u32 NoExecute;	/* Execute access to a page marked NX */
	__u32 imprecise;	/* Can't determine the	exact fault address */
};

/* memory exception data */
struct kfd_hsa_memory_exception_data {
	struct kfd_memory_exception_failure failure;
	__u64 va;
	__u32 gpu_id;
	__u32 ErrorType; /* 0 = no RAS error,
			  * 1 = ECC_SRAM,
			  * 2 = Link_SYNFLOOD (poison),
			  * 3 = GPU hang (not attributable to a specific cause),
			  * other values reserved
			  */
};

/* hw exception data */
struct kfd_hsa_hw_exception_data {
	__u32 reset_type;
	__u32 reset_cause;
	__u32 memory_lost;
	__u32 gpu_id;
};

/* hsa signal event data */
struct kfd_hsa_signal_event_data {
	__u64 last_event_age;	/* to and from KFD */
};

struct kfd_event_data {
	union {
		/* From KFD */
		struct kfd_hsa_memory_exception_data memory_exception_data;
		struct kfd_hsa_hw_exception_data hw_exception_data;
		/* To and From KFD */
		struct kfd_hsa_signal_event_data signal_event_data;
	};
	__u64 kfd_event_data_ext;	/* pointer to an extension structure
					   for future exception types */
	__u32 event_id;		/* to KFD */
	__u32 pad;
};
```
You must keep track of event_type from event creation to know which variant of the union to use.

For SIGNAL/DEBUG events you specify the last_event_age parameter.

- If set to value greater than 0; for example 1 if you don't know current age,
when the event's age is different it will be marked as signalled and new age retured.

- If set to 0; event will be marked as signalled only after it's age changes after the waiter is registered, so there is a greater chance you will miss an event

#### Inputs
	__u64 events_ptr;		/* pointed to struct
					   kfd_event_data array, to KFD */
	__u32 num_events;		/* to KFD */
	__u32 wait_for_all;		/* to KFD */
	__u32 timeout;		/* to KFD */

timeout 0 - immediate
timeout 1..u32::MAX -1 - time in milisecs
timeout u32::MAX - indefinite

#### Outputs
```C
    #define KFD_IOC_WAIT_RESULT_COMPLETE	0
    #define KFD_IOC_WAIT_RESULT_TIMEOUT		1
    #define KFD_IOC_WAIT_RESULT_FAIL		2
	__u32 wait_result;		/* from KFD */
```
You can get result FAIL if you wait an a destroyed event or destroy an event while waiting on events.

- ENOMEM if couldn't allocate waiters
- EFAULT if couldn't copy event data into kernel space
- EINVAL if event is destoyed during waiting
- EIO if everything was successful but wait result is FAIL
- EINTR if received SIGKILL signal
- ERESTARTSYS if received other signals
