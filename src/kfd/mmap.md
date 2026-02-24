# MMAP api

Mmap's offset is split into bitfields:
| MSB | LSB | field |
|-----|-----|---------|
| 64  | 62  | mmap_type |
| 62  | 46  | gpu_id |
| 46  |  0  | ...    |

## GPU_ID
Specific to the current device topology.
Can be viewed at `/sys/class/kfd/kfd/topology`.

## MMAP_TYPE

### 3 -> Doorbell
As of now you must map all doorbells allocated for current process.

Use here the doorbell_offset you received from [AMDKFD_IOC_CREATE_QUEUE](ioctl.md#create_queue)
It contains all the fields already populated.

### 2 -> Events
You can use this to map the event signal page.

Use the maximum size of 4096 * 8 bytes.

I don't know yet why you'd want to map less.

You can index this page with event_id, but only for SIGNAL and DEBUG event types:
```C
u64 event_value = event_page[event_id];
```

Returns:
- EINVAL if signal page has not been created yet or you used too large size

### 1 -> Reserved Mem
Although it is a public api, it's not designed to be used by the user.

It's used when initializing CWSR for APUs in `kfd_open()` (opening the kfd file).

Allocated memory in kernel space (2 * PAGE_SIZE in size) for this process and maps it into process address space.
ENOMEM if out of memory.
EINVAL if process kfd data was not found

But `mmap()` by itself doesn't set this memory for CWSR.

### 0 -> MMIO
Must be exactly PAGE_SIZE in size. Assumes PAGE_SIZE is 4096 bytes.
It split into 1024 32bit values.

It mapps to a special singleton BO created by the amdgpu module during device initialization.
It mapps to a special MMIO region called REG_HOLE.

Although it allows direct access to gpu like kernel does with `WREG32` there are no raw regs there
to access by the user and the firmware needs to be instructed to look into that region for specific
values.

There are 2 set up values.
```C
u32 *mapped_page = mmap(fd, 0, );
mapped_page[KFD_MMIO_REMAP_HDP_MEM_FLUSH_CNTL];
mapped_page[KFD_MMIO_REMAP_HDP_REG_FLUSH_CNTL];
```

#### What do these values do?
Don't know exactly, they flush something in HDP, but I need more info still.

In kernel they write a 0 into there to perform a device wide flush of HDP.
Or send a PACKET_3 into a specific ring with a write 0 command to that register.

Host Data Path (HDP) is an old thing dating back to at least r600 gpus.
HDP is an IP block in a gpu.
It has clock gating settings.
Perhaps the reg flush is for flushing the hdp settings as they are controlled via registers.
