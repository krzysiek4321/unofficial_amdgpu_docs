## RDNA 2
### instruction cache
- 4 way set-associative
- 32kB(4 banks of 128 cachelines)
- cache line is 64bytes long
- shared for all SIMD in a WGP
- `s_icache_inv` to flush

### constant cache
Don't know, perhaps it's the same as scalar cache.

### sqc data cache
Don't know, instructions mentioning this cache are only present in the Reference Guide.

### texture caches
It's actually vector caches but the data first goes to texture mapping unit, for each address in a vector, the TMU will sample the four nearest neighbors, decompress the data, and perform interpolation.

### scalar (data) cache
- 4-way set-associative
- write-back
- 16kB(2 banks of 128 cachelines)
- line is 64bytes
- shared by all SIMD in a WGP
- `s_dcache_inv` to flush

### LDS
- 128kB for each WGP
- 64 banks, each has an atomic unit and 512 4-byte entries

### GDS
- 64kB globaly shared
- 32 banks, each has an atomic unit and 512 4-byte entries
- has some special features to talk to buffers in gpu memory

### vector cache (shader cache, gl0 cache)
- shared in a CU (2 SIMD32)
- 32-way
- 16kB
- write-through with LRU replacement
- 128byte cache line
- `buffer_gl0_inv` to flush

### RB cache
I don't know.
RDNA whitepaper mentiones an RB cache, which looking at silicone diagrams looks like
ROP for Navi 22, but I need more info.

### L1
- accessed by scalar cache, vector cache, instruction cache
- read only
- 16-way
- supposedly 128kB, but it doesn't show in amd-smi
- shared within a shader array (10 CUs for gfx1031)
- `buffer_gl1_inv` to flush with acknowledge or `s_gl1_inv` without

### L2
- accessed by L1 cache
- multiple channels
- 16-way
- size is gpu dependant (12 * 256kB (3kB) for Navi 22 (gfx1031+))
- has atomic units that support relaxed consistency mode through ack after (maybe not all) atomic operations
- shared by all CUs
- perhaps `v_pipeflush` to flush, but usually you should set GLC,SLC,DLC bits to controll caches

### L3
- accessed by L2 cache
- size dependant on gpu (96MB for gfx1031)
- ryzen inspired "infinity cache", introduced in RDNA2, but instructions are not aware of this cache


### Additional notes
I'm not including latency info, because it's probably different for gfx1031 than for gfx1030, which Chester Lam used for measurements.

`v_pipeflush` - "flush the VALU destination cache", whatever that means

A CU shares a request and return bus between SIMD32, but it's possible for an individual SIMD32 to receive 2 cache lines per clock (one from LDS and one from L0)

Cache banks describe physical silicone blocks and n-way describe logical grouping of cachelines

**Cache n-way** associativity means that when a memory address is accessed
the memory unit first selects which cache set (of size n * cache_line) the address falls in using modulo arithmetic.
Next it checks if any of the available sets (slots) already has the memory desired.
If not it's a cache miss and the cache loads the memory from higher level.
This allows an optimization for when memory is not tightly packed, so for realistic memory access patterns.

### Sources
- [AMD's RDNA2 the Reference Guide](https://gpuopen.com/amd-gpu-architecture-programming-documentation/)
- [AMD's RDNA white paper](https://www.techpowerup.com/gpu-specs/docs/amd-rdna-whitepaper.pdf)
- [AMD's machine readable ISA spec for RDNA2](https://gpuopen.com/machine-readable-isa/)
- AMD's RDNA2 marketing materials
- output from [amd-smi](https://github.com/ROCm/amdsmi) for Radeon RX 6700 XT (gfx1031)
- techpowerup article on [Navi 21](https://www.techpowerup.com/gpu-specs/amd-navi-21.g923) and [Navi 22](https://www.techpowerup.com/gpu-specs/amd-navi-22.g951) which contain annotated images of silicone die layout
- ["AMD’s RDNA 2: Shooting For the Top" by Chester Lam](https://chipsandcheese.com/p/amds-rdna-2-shooting-for-the-top)
- [Mesa3D's Unofficial GCN/RDNA ISA reference errata](https://gitlab.freedesktop.org/mesa/mesa/-/blob/main/src/amd/compiler/README-ISA.md)
