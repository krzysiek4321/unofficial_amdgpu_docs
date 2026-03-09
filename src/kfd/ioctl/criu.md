# CRIU
Checkpoint restore.

You need to have CAP_CHECKPOINT_RESTORE or CAP_SYS_ADMIN capability.

## CRIU_OP
	AMDKFD_IOCTL_DEF(AMDKFD_IOC_CRIU_OP, kfd_ioctl_criu, KFD_IOC_FLAG_CHECKPOINT_RESTORE),
	AMDKFD_IOWR(0x22, struct kfd_ioctl_criu_args)

```C
/*
 * CRIU IOCTLs (Checkpoint Restore In Userspace)
 *
 * When checkpointing a process, the userspace application will perform:
 * 1. PROCESS_INFO op to determine current process information. This pauses execution and evicts
 *    all the queues.
 * 2. CHECKPOINT op to checkpoint process contents (BOs, queues, events, svm-ranges)
 * 3. UNPAUSE op to un-evict all the queues
 *
 * When restoring a process, the CRIU userspace application will perform:
 *
 * 1. RESTORE op to restore process contents
 * 2. RESUME op to start the process
 *
 * Note: Queues are forced into an evicted state after a successful PROCESS_INFO. User
 * application needs to perform an UNPAUSE operation after calling PROCESS_INFO.
 */

enum kfd_criu_op {
	KFD_CRIU_OP_PROCESS_INFO,
	KFD_CRIU_OP_CHECKPOINT,
	KFD_CRIU_OP_UNPAUSE,
	KFD_CRIU_OP_RESTORE,
	KFD_CRIU_OP_RESUME,
};

/**
 * kfd_ioctl_criu_args - Arguments perform CRIU operation
 * @devices:		[in/out] User pointer to memory location for devices information.
 * 			This is an array of type kfd_criu_device_bucket.
 * @bos:		[in/out] User pointer to memory location for BOs information
 * 			This is an array of type kfd_criu_bo_bucket.
 * @priv_data:		[in/out] User pointer to memory location for private data
 * @priv_data_size:	[in/out] Size of priv_data in bytes
 * @num_devices:	[in/out] Number of GPUs used by process. Size of @devices array.
 * @num_bos		[in/out] Number of BOs used by process. Size of @bos array.
 * @num_objects:	[in/out] Number of objects used by process. Objects are opaque to
 *				 user application.
 * @pid:		[in/out] PID of the process being checkpointed
 * @op			[in] Type of operation (kfd_criu_op)
 *
 * Return: 0 on success, -errno on failure
 */
struct kfd_ioctl_criu_args {
	__u64 devices;		/* Used during ops: CHECKPOINT, RESTORE */
	__u64 bos;		/* Used during ops: CHECKPOINT, RESTORE */
	__u64 priv_data;	/* Used during ops: CHECKPOINT, RESTORE */
	__u64 priv_data_size;	/* Used during ops: PROCESS_INFO, RESTORE */
	__u32 num_devices;	/* Used during ops: PROCESS_INFO, RESTORE */
	__u32 num_bos;		/* Used during ops: PROCESS_INFO, RESTORE */
	__u32 num_objects;	/* Used during ops: PROCESS_INFO, RESTORE */
	__u32 pid;		/* Used during ops: PROCESS_INFO, RESUME */
	__u32 op;
};

struct kfd_criu_device_bucket {
	__u32 user_gpu_id;
	__u32 actual_gpu_id;
	__u32 drm_fd;
	__u32 pad;
};

struct kfd_criu_bo_bucket {
	__u64 addr;
	__u64 size;
	__u64 offset;
	__u64 restored_offset;    /* During restore, updated offset for BO */
	__u32 gpu_id;             /* This is the user_gpu_id */
	__u32 alloc_flags;
	__u32 dmabuf_fd;
	__u32 pad;
};
```
