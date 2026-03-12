# Profiling gpus

## IOCTLs
### GET_CLOCK_COUNTERS
		AMDKFD_IOWR(0x05, struct kfd_ioctl_get_clock_counters_args)

#### Inputs
	__u32 gpu_id;		/* to KFD */

#### Outputs
	__u64 gpu_clock_counter;	/* from KFD */
	__u64 cpu_clock_counter;	/* from KFD */
	__u64 system_clock_counter;	/* from KFD */
	__u64 system_clock_freq;	/* from KFD */
