# Compute Wave Store Resume (CWSR)
If enabled in module parameters,
allows the gpu to stop a wave during execution, save state and resume after some time.

## Terminology
### Trap Base Address (TBA)
Address accessible to the GPU/APU to memory for the CWSR trap handler code in native gpu ISA.

### Trap Memory Address (TMA)
Address accessible to the GPU/APU to memory reserved for the CWSR trap handler to use.

## Default trap handler
Sometimes reffered as first level handler.

Each gpu generation has it's own trap handler version.

### Size and offsets
It is always `2 * PAGE_SIZE` in size.
TBA starts at `0` offset.
TMA starts at `1.5 * PAGE_SIZE` offset.

### Reserved Virtual Address
See `AMDGPU_VA_RESERVED_TRAP_START`

### Read more
You can find the assigned trap handlers in `kernel/drivers/gpu/drm/amd/amdkfd/kfd_device.c`.

For example for gfx103\* the trap handler bytecode is generated from
`kernel/drivers/gpu/drm/amd/amdkfd/cwsr_trap_handler_gfx10.asm`.

You can verify it's correct by decompiling the bytecode used in `kfd_device.c`.

## Supplying a custom trap handler
Use the [`set_trap_handler`](ioctl/dbg.md#set_trap_handler) ioctl.

It will register the new handler as seccond level handler.

Take note the supplied tba and tma values must be addresses in gpu's address space for dGPU and
memory set as EXECUTABLE.

### Calling convention
todo

## Suspending and resuming waves
todo

## Notes on internals
### There is actually a distincion between two scenarios

#### For APUs
Here it uses `mmap` internally to allocate memory for CWSR in RAM and set the address.

tba_address = &cpu allocated memory
tma_address = tba_address + tma_offset


#### For dGPUs
The memory address is statically reserved in the gpu address space. See `cwsr_base`.

The memory is formally allocated during `acquire_vm` ioctl at the cwsr_base gpu addresses,
with flags GTT | EXECUTABLE | NO_SUBSTITUTE.
It gets pinned to the GTT.

tba_address = cwsr_base
tma_address = tba_address + tma_offset.

### Special tma values for default handler
```C
u64 *TMA;
TMA[0] = second_level_trap_base_address;
TMA[1] = second_level_trap_memory_address;
TMA[2] = enable_flag;
```

### Is it possible to set a custom handler before the first level handler is installed?
Yes but it doesn't matter:

- for apu, during process creation the first_level handler is installed,
- for dgpu, you can call `set_trap_handler` before `acquire_vm`, but during init_cwsr_dgpu it's
going to overwrite the `tba_addr` and `tma_addr` to default handler and you have to set your custom
handler again; so just do it once after `acquire_vm`.
