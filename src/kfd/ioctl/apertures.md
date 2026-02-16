# Apertures
It allows a user to query what devices are available.
But it's impossible to tell which is which without looking into
topology info.

Topology can be found in KFD sysfs.

## GET_PROCESS_APERTURES
		AMDKFD_IOR(0x06, struct kfd_ioctl_get_process_apertures_args)

### Outputs
```C
struct {
    __u64 lds_base;		/* from KFD */
	__u64 lds_limit;		/* from KFD */
	__u64 scratch_base;		/* from KFD */
	__u64 scratch_limit;		/* from KFD */
	__u64 gpuvm_base;		/* from KFD */
	__u64 gpuvm_limit;		/* from KFD */
	__u32 gpu_id;		/* from KFD */
} nodes[7];
__u32 num_of_nodes;
```

## GET_PROCESS_APERTURES_NEW
		AMDKFD_IOWR(0x14, struct kfd_ioctl_get_process_apertures_new_args)

Just like [GET_PROCESS_APERTURES](#get_process_apertures) except there is no limit to the number of nodes.
