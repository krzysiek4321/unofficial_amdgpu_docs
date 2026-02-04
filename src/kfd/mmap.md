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

### 2 -> Events

### 1 -> Reserved Mem

### 0 -> MMIO
