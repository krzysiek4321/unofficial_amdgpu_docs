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

### 0 -> MMIO
