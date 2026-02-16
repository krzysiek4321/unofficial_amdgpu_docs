# Events

## CREATE_EVENT
		AMDKFD_IOWR(0x08, struct kfd_ioctl_create_event_args)

### Inputs
```C
    #define KFD_IOC_EVENT_SIGNAL		0
    #define KFD_IOC_EVENT_NODECHANGE		1
    #define KFD_IOC_EVENT_DEVICESTATECHANGE	2
    #define KFD_IOC_EVENT_HW_EXCEPTION		3
    #define KFD_IOC_EVENT_SYSTEM_EVENT		4
    #define KFD_IOC_EVENT_DEBUG_EVENT		5
    #define KFD_IOC_EVENT_PROFILE_EVENT		6
    #define KFD_IOC_EVENT_QUEUE_EVENT		7
    #define KFD_IOC_EVENT_MEMORY		8
	__u32 event_type;		/* to KFD */
	__u32 auto_reset;		/* to KFD */
	__u32 node_id;		/* to KFD - only valid for certain
							event types */
```

### Outputs
	__u64 event_page_offset;	/* from KFD */
	__u32 event_trigger_data;	/* from KFD - signal events only */
	__u32 event_id;		/* from KFD */
	__u32 event_slot_index;	/* from KFD */


## DESTROY_EVENT
		AMDKFD_IOW(0x09, struct kfd_ioctl_destroy_event_args)

### Input
	__u32 event_id;		/* to KFD */

## SET_EVENT
		AMDKFD_IOW(0x0A, struct kfd_ioctl_set_event_args)

### Input
	__u32 event_id;		/* to KFD */

## RESET_EVENT
		AMDKFD_IOW(0x0B, struct kfd_ioctl_reset_event_args)

### Input
	__u32 event_id;		/* to KFD */

## WAIT_EVENTS
		AMDKFD_IOWR(0x0C, struct kfd_ioctl_wait_events_args)

### Inputs
	__u64 events_ptr;		/* pointed to struct
					   kfd_event_data array, to KFD */
	__u32 num_events;		/* to KFD */
	__u32 wait_for_all;		/* to KFD */
	__u32 timeout;		/* to KFD */

### Outputs
```C
    #define KFD_IOC_WAIT_RESULT_COMPLETE	0
    #define KFD_IOC_WAIT_RESULT_TIMEOUT		1
    #define KFD_IOC_WAIT_RESULT_FAIL		2
	__u32 wait_result;		/* from KFD */
```
