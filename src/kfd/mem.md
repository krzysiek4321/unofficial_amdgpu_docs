# Memory

## GTT
Can be mmaped via drm file descriptor

## VRAM with large bar
Can be mmaped via drm file descriptor

## MMIO_REMAP
You can only write to this memory.

Reads are ignored.

## Scatter Gather memory (Doorbell, MMIO_REMAP)
Grep for `ttm_bo_type_sg`.

Can be written to cannot be read?.

Such memory will produce a SIGBUS on access if it was not specifically marked to be mappable.


## VMID
There is a total of 16 VMIDs per VMHUB.
VMID 0 is reserved for the system.

Since gfx10
VMID 1..7 used for graphics
VMID 8..15 used for kfd

## Evictions, moves and pinning
### When can eviction happen?

You can manually trigger an eviction in via /sys/kernel/debug/dri/0/amdgpu_evict_*.

### What does an eviction change or impact?

### When can a move happen?

### What is pinning?

Pinning means to lock pages in memory along with keeping them at a fixed
offset. It is required when a buffer can not be moved, for example, when
a display buffer is being scanned out.

### Which memory is automatically pinned?

In KFD only MMIO_REMAP and DOORBELL memory get's pinned (to GTT) during [alloc_memory_of_gpu](./alloc.md#alloc_memory_of_gpu)
and unpinned during [free_memory_of_gpu](./alloc.md#free_memory_of_gpu).

Special MES proc_ctx_bo and gang_ctx_bo also get pinned to GTT during first queue creation.

Kfd creates a kernel reserved IB bo for every process, which also gets pinned to the GTT.
DGPU's CWSR buffer also gets pinned to GTT.

Also when providing a signal page in [create_event](./events.md#create_event) provided BO gets pinned to GTT.

It's invalid to try to pin/unpin USERPTR memory.
