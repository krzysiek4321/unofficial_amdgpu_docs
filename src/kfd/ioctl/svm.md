# Shared Virtual Memory (SVM)

Requires CONFIG_HSA_AMD_SVM to be enabled when building amdgpu module.

Allows sharing virtual address space between GPUs and CPU.?

#### How is that different from cpu mapping?
todo

#### How do I obtain a cpu address for kfd memory handle?
todo

## SVM
    AMDKFD_IOWR(0x20, struct kfd_ioctl_svm_args)

You can get or set attributes for gpu memory mapped to the given
VA range.

#### Input requirements
Both start_addr and size must be non zero and PAGE_SIZE aligned.

The meaning of the attribute value depends on the attribute type.

A variable number of attributes can be given.
`nattr` specifies the number of attributes or how many the kernel can populate.

New attributes can be
added in the future without breaking the ABI. If unknown attributes
are given, the function returns -EINVAL.

#### What if the VA range has multiple BOs
For **get** it returns flag intersection.

For **set** it tries to set provided flags to all of these objects.

#### What if the VA range only partially includes a BO?
For example you create a BO of 16 memory pages, but the provided VA range only includes 4 pages.

It then splits the VA mapping to set provided flags only for these pages.

#### What if different pages have different preferred or prefetch locations?
0xffffffff will be returned 

#### How do I get gpu specific attributes?
You provide gpu_id as attribute value.
See the C definitions below.

#### C definitions
```C
struct kfd_ioctl_svm_args {
	__u64 start_addr;
	__u64 size;
	__u32 op;
	__u32 nattr;
	/* Variable length array of attributes */
	struct kfd_ioctl_svm_attribute attrs[];
};

struct kfd_ioctl_svm_attribute {
	__u32 type;
	__u32 value;
};

/* Guarantee host access to memory */
#define KFD_IOCTL_SVM_FLAG_HOST_ACCESS 0x00000001
/* Fine grained coherency between all devices with access */
#define KFD_IOCTL_SVM_FLAG_COHERENT    0x00000002
/* Use any GPU in same hive as preferred device */
#define KFD_IOCTL_SVM_FLAG_HIVE_LOCAL  0x00000004
/* GPUs only read, allows replication */
#define KFD_IOCTL_SVM_FLAG_GPU_RO      0x00000008
/* Allow execution on GPU */
#define KFD_IOCTL_SVM_FLAG_GPU_EXEC    0x00000010
/* GPUs mostly read, may allow similar optimizations as RO, but writes fault */
#define KFD_IOCTL_SVM_FLAG_GPU_READ_MOSTLY     0x00000020
/* Keep GPU memory mapping always valid as if XNACK is disable */
#define KFD_IOCTL_SVM_FLAG_GPU_ALWAYS_MAPPED   0x00000040
/* Fine grained coherency between all devices using device-scope atomics */
#define KFD_IOCTL_SVM_FLAG_EXT_COHERENT        0x00000080

enum kfd_ioctl_svm_op {
	KFD_IOCTL_SVM_OP_SET_ATTR,
	KFD_IOCTL_SVM_OP_GET_ATTR
};

/** kfd_ioctl_svm_location - Enum for preferred and prefetch locations
 *
 * GPU IDs are used to specify GPUs as preferred and prefetch locations.
 * Below definitions are used for system memory or for leaving the preferred
 * location unspecified.
 */
enum kfd_ioctl_svm_location {
	KFD_IOCTL_SVM_LOCATION_SYSMEM = 0,
	KFD_IOCTL_SVM_LOCATION_UNDEFINED = 0xffffffff
};

/**
 * kfd_ioctl_svm_attr_type - SVM attribute types
 *
 * @KFD_IOCTL_SVM_ATTR_PREFERRED_LOC: gpuid of the preferred location, 0 for
 *                                    system memory
 * @KFD_IOCTL_SVM_ATTR_PREFETCH_LOC: gpuid of the prefetch location, 0 for
 *                                   system memory. Setting this triggers an
 *                                   immediate prefetch (migration).
 * @KFD_IOCTL_SVM_ATTR_ACCESS:
 * @KFD_IOCTL_SVM_ATTR_ACCESS_IN_PLACE:
 * @KFD_IOCTL_SVM_ATTR_NO_ACCESS: specify memory access for the gpuid given
 *                                by the attribute value
 * @KFD_IOCTL_SVM_ATTR_SET_FLAGS: bitmask of flags to set (see
 *                                KFD_IOCTL_SVM_FLAG_...)
 * @KFD_IOCTL_SVM_ATTR_CLR_FLAGS: bitmask of flags to clear
 * @KFD_IOCTL_SVM_ATTR_GRANULARITY: migration granularity
 *                                  (log2 num pages)
 */
enum kfd_ioctl_svm_attr_type {
	KFD_IOCTL_SVM_ATTR_PREFERRED_LOC,
	KFD_IOCTL_SVM_ATTR_PREFETCH_LOC,
	KFD_IOCTL_SVM_ATTR_ACCESS,
	KFD_IOCTL_SVM_ATTR_ACCESS_IN_PLACE,
	KFD_IOCTL_SVM_ATTR_NO_ACCESS,
	KFD_IOCTL_SVM_ATTR_SET_FLAGS,
	KFD_IOCTL_SVM_ATTR_CLR_FLAGS,
	KFD_IOCTL_SVM_ATTR_GRANULARITY
};
```

## SET_XNACK_MODE
    AMDKFD_IOWR(0x21, struct kfd_ioctl_set_xnack_mode_args)

Requires CONFIG_HSA_AMD_SVM=y when building amdgpu module and
it's good to set amdgpu.noretry=0 in module parameters,
because the default usually means OFF.

Allows you to query if xnack is enabled if you provide a negative value.
Also you can try to set xnack mode (true/false).

XNACK is about changing how gpu behaves when a page fault happens.
The goal is to gracefully recover from page faults.

To learn more grep amdgpu source code for `noretry`.

```C
struct kfd_ioctl_set_xnack_mode_args {
	__s32 xnack_enabled;
};
```
#### When can I change XNACK mode?
Only when your process has no queues running.

#### Which gpus does it apply to?
No older than gfx901, but you need to check if your gpu supports it.
See [llvm amdgpu target features](https://llvm.org/docs/AMDGPUUsage.html#processors).
You might notice it says some gfx8 gpus have xnack, but linux source code takes priority.

It seems to me this feature has been abandoned for gpus older than gfx103.

#### Can I run my compiled shaders with XNACK on?
You can run a regular shader, but unless it was compiled with xnack support it may not use it and run slower than with XNACK off.
See [xnack target feature](https://llvm.org/docs/AMDGPUUsage.html#target-features).
