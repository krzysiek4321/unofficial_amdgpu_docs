# Tiling/Swizzling Mode
The main idea is to store image pixels in such a way which prevents cache misses for certain operations on groups of pixels.

##### How can I use it?
Don't know

## IOCTLs
### get_tile_config
    AMDKFD_IOWR(0x12, struct kfd_ioctl_get_tile_config_args)

```C
struct kfd_ioctl_get_tile_config_args {
	/* to KFD: pointer to tile array */
	__u64 tile_config_ptr;
	/* to KFD: pointer to macro tile array */
	__u64 macro_tile_config_ptr;
	/* to KFD: array size allocated by user mode
	 * from KFD: array size filled by kernel
	 */
	__u32 num_tile_configs;
	/* to KFD: array size allocated by user mode
	 * from KFD: array size filled by kernel
	 */
	__u32 num_macro_tile_configs;

	__u32 gpu_id;		/* to KFD */
	__u32 gb_addr_config;	/* from KFD */
	__u32 num_banks;		/* from KFD */
	__u32 num_ranks;		/* from KFD */
	/* struct size can be extended later if needed
	 * without breaking ABI compatibility
	 */
};
```
