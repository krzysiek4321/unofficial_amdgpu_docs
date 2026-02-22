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

### 1 -> Reserved Mem
Although it is a public api, it's not designed to be used by the user.

It's used when initializing CWSR for APUs in `kfd_open()` (opening the kfd file).

Allocated memory in kernel space (2 * PAGE_SIZE in size) for this process and maps it into process address space.
ENOMEM if out of memory.
EINVAL if process kfd data was not found

But `mmap()` by itself doesn't set this memory for CWSR.

### 0 -> MMIO
