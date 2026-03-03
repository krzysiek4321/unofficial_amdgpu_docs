# CRIU
Checkpoint restore.

You need to have CAP_CHECKPOINT_RESTORE or CAP_SYS_ADMIN capability.

## CRIU_OP
	AMDKFD_IOCTL_DEF(AMDKFD_IOC_CRIU_OP, kfd_ioctl_criu, KFD_IOC_FLAG_CHECKPOINT_RESTORE),
	AMDKFD_IOWR(0x22, struct kfd_ioctl_criu_args)
