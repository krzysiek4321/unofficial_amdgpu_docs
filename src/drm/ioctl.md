# ioctl

First is the name of the kernel function corresponding to the ioctl.
Second are the drm permissions necessary to access the ioctl.

Check `kernel/drivers/gpu/drm/drm_ioctl.c` for more info.

## AMDGPU specific
Add `AMDGPU_` to get C definitions.
<details>
<summary>click to expand</summary>

### GEM_CREATE
amdgpu_gem_create_ioctl, DRM_AUTH|DRM_RENDER_ALLOW

###	CTX
amdgpu_ctx_ioctl, DRM_AUTH|DRM_RENDER_ALLOW

###	VM
amdgpu_vm_ioctl, DRM_AUTH|DRM_RENDER_ALLOW),

### SCHED
amdgpu_sched_ioctl, DRM_MASTER),

### BO_LIST
amdgpu_bo_list_ioctl, DRM_AUTH|DRM_RENDER_ALLOW),

### FENCE_TO_HANDLE
amdgpu_cs_fence_to_handle_ioctl, DRM_AUTH|DRM_RENDER_ALLOW),

### GEM_MMAP
amdgpu_gem_mmap_ioctl, DRM_AUTH|DRM_RENDER_ALLOW),

### GEM_WAIT_IDLE
amdgpu_gem_wait_idle_ioctl, DRM_AUTH|DRM_RENDER_ALLOW),

### CS
amdgpu_cs_ioctl, DRM_AUTH|DRM_RENDER_ALLOW),

### INFO
amdgpu_info_ioctl, DRM_AUTH|DRM_RENDER_ALLOW),

### WAIT_CS
amdgpu_cs_wait_ioctl, DRM_AUTH|DRM_RENDER_ALLOW),

### WAIT_FENCES
amdgpu_cs_wait_fences_ioctl, DRM_AUTH|DRM_RENDER_ALLOW),

### GEM_METADATA
amdgpu_gem_metadata_ioctl, DRM_AUTH|DRM_RENDER_ALLOW),

### GEM_VA
amdgpu_gem_va_ioctl, DRM_AUTH|DRM_RENDER_ALLOW),

### GEM_OP
amdgpu_gem_op_ioctl, DRM_AUTH|DRM_RENDER_ALLOW),

### GEM_USERPTR
amdgpu_gem_userptr_ioctl, DRM_AUTH|DRM_RENDER_ALLOW),

### USERQ
amdgpu_userq_ioctl, DRM_AUTH|DRM_RENDER_ALLOW),

### USERQ_SIGNAL
amdgpu_userq_signal_ioctl, DRM_AUTH|DRM_RENDER_ALLOW),

### USERQ_WAIT
amdgpu_userq_wait_ioctl, DRM_AUTH|DRM_RENDER_ALLOW),

</details>

## Drm common
Add `DRM_IOCTL_` to get C definitions.

<details>
<summary>click to expand</summary>

### VERSION
drm_version, DRM_RENDER_ALLOW),

### GET_UNIQUE
drm_getunique, 0),

### GET_MAGIC
drm_getmagic, 0

Called by the node which needs to be authenticated.
Procudes a magic value to be passed to the process holding a master.

### GET_CLIENT
drm_getclient, 0),

### GET_STATS
drm_getstats, 0),

### GET_CAP
drm_getcap, DRM_RENDER_ALLOW),

### SET_CLIENT_CAP
drm_setclientcap, 0),

### SET_VERSION
drm_setversion, DRM_MASTER),


### SET_UNIQUE
drm_invalid_op, DRM_AUTH|DRM_MASTER|DRM_ROOT_ONLY),

### BLOCK
drm_noop, DRM_AUTH|DRM_MASTER|DRM_ROOT_ONLY),

### UNBLOCK
drm_noop, DRM_AUTH|DRM_MASTER|DRM_ROOT_ONLY),

### AUTH_MAGIC
drm_authmagic, DRM_MASTER

Takes the magic token, searches for corresponding opened drm_node file (client) and set's it as authenticated.

### SET_MASTER
drm_setmaster_ioctl, 0),

Returns:
- 0 if successful or was already master
- EACCESS if not capable(CAP_SYS_ADMIN) **and** (this client was never a master or it was a master but current process's thread group doesn't match the clients tgid)
- EBUSY if we have access but there is a master set for the device
- EINVAL if we have access, there is no master set for device and this client doesn't have a master linked
- ENOMEM if couldn't allocate memory for master struct

### DROP_MASTER
drm_dropmaster_ioctl, 0

Returns:
- EACCESS if not capable(CAP_SYS_ADMIN) **and** (this client was never a master or it was a master but current process's thread group doesn't match the clients tgid)
- EINVAL if we are not a master
    or if we are a master and our lease owner isn't current dev master
    or if there is no current dev master


### ADD_DRAW
drm_noop, DRM_AUTH|DRM_MASTER|DRM_ROOT_ONLY),

### RM_DRAW
drm_noop, DRM_AUTH|DRM_MASTER|DRM_ROOT_ONLY),


### FINISH
drm_noop, DRM_AUTH),


### WAIT_VBLANK
drm_wait_vblank_ioctl, 0),


### UPDATE_DRAW
drm_noop, DRM_AUTH|DRM_MASTER|DRM_ROOT_ONLY),


### GEM_CLOSE
drm_gem_close_ioctl, DRM_RENDER_ALLOW),

### GEM_FLINK
drm_gem_flink_ioctl, DRM_AUTH),

### GEM_OPEN
drm_gem_open_ioctl, DRM_AUTH),

### GEM_CHANGE_HANDLE
drm_gem_change_handle_ioctl, DRM_RENDER_ALLOW),


### MODE_GETRESOURCES
drm_mode_getresources, 0),


### PRIME_HANDLE_TO_FD
drm_prime_handle_to_fd_ioctl, DRM_RENDER_ALLOW),

- EPERM - if you try to export a USERPTR memory
    or underlying BO has AMDGPU_GEM_CREATE_VM_ALWAYS_VALID flag set

### PRIME_FD_TO_HANDLE
drm_prime_fd_to_handle_ioctl, DRM_RENDER_ALLOW),


### SET_CLIENT_NAME
drm_set_client_name, DRM_RENDER_ALLOW),


### MODE_GETPLANERESOURCES
drm_mode_getplane_res, 0),

### MODE_GETCRTC
drm_mode_getcrtc, 0),

### MODE_SETCRTC
drm_mode_setcrtc, DRM_MASTER),

### MODE_GETPLANE
drm_mode_getplane, 0),

### MODE_SETPLANE
drm_mode_setplane, DRM_MASTER),

### MODE_CURSOR
drm_mode_cursor_ioctl, DRM_MASTER),

### MODE_GETGAMMA
drm_mode_gamma_get_ioctl, 0),

### MODE_SETGAMMA
drm_mode_gamma_set_ioctl, DRM_MASTER),

### MODE_GETENCODER
drm_mode_getencoder, 0),

### MODE_GETCONNECTOR
drm_mode_getconnector, 0),

### MODE_ATTACHMODE
drm_noop, DRM_MASTER),

### MODE_DETACHMODE
drm_noop, DRM_MASTER),

### MODE_GETPROPERTY
drm_mode_getproperty_ioctl, 0),

### MODE_SETPROPERTY
drm_connector_property_set_ioctl, DRM_MASTER),

### MODE_GETPROPBLOB
drm_mode_getblob_ioctl, 0),

### MODE_GETFB
drm_mode_getfb, 0),

### MODE_GETFB2
drm_mode_getfb2_ioctl, 0),

### MODE_ADDFB
drm_mode_addfb_ioctl, 0),

### MODE_ADDFB2
drm_mode_addfb2_ioctl, 0),

### MODE_RMFB
drm_mode_rmfb_ioctl, 0),

### MODE_CLOSEFB
drm_mode_closefb_ioctl, 0),

### MODE_PAGE_FLIP
drm_mode_page_flip_ioctl, DRM_MASTER),

### MODE_DIRTYFB
drm_mode_dirtyfb_ioctl, DRM_MASTER),

### MODE_CREATE_DUMB
drm_mode_create_dumb_ioctl, 0),

### MODE_MAP_DUMB
drm_mode_mmap_dumb_ioctl, 0),

### MODE_DESTROY_DUMB
drm_mode_destroy_dumb_ioctl, 0),

### MODE_OBJ_GETPROPERTIES
drm_mode_obj_get_properties_ioctl, 0),

### MODE_OBJ_SETPROPERTY
drm_mode_obj_set_property_ioctl, DRM_MASTER),

### MODE_CURSOR2
drm_mode_cursor2_ioctl, DRM_MASTER),

### MODE_ATOMIC
drm_mode_atomic_ioctl, DRM_MASTER),

### MODE_CREATEPROPBLOB
drm_mode_createblob_ioctl, 0),

### MODE_DESTROYPROPBLOB
drm_mode_destroyblob_ioctl, 0),

### SYNCOBJ_CREATE
drm_syncobj_create_ioctl, DRM_RENDER_ALLOW),

### SYNCOBJ_DESTROY
drm_syncobj_destroy_ioctl, DRM_RENDER_ALLOW),

### SYNCOBJ_HANDLE_TO_FD
drm_syncobj_handle_to_fd_ioctl, DRM_RENDER_ALLOW),

### SYNCOBJ_FD_TO_HANDLE
drm_syncobj_fd_to_handle_ioctl, DRM_RENDER_ALLOW),

### SYNCOBJ_TRANSFER
drm_syncobj_transfer_ioctl, DRM_RENDER_ALLOW),

### SYNCOBJ_WAIT
drm_syncobj_wait_ioctl, DRM_RENDER_ALLOW),

### SYNCOBJ_TIMELINE_WAIT
drm_syncobj_timeline_wait_ioctl, DRM_RENDER_ALLOW),

### SYNCOBJ_EVENTFD
drm_syncobj_eventfd_ioctl, DRM_RENDER_ALLOW),

### SYNCOBJ_RESET
drm_syncobj_reset_ioctl, DRM_RENDER_ALLOW),

### SYNCOBJ_SIGNAL
drm_syncobj_signal_ioctl, DRM_RENDER_ALLOW),

### SYNCOBJ_TIMELINE_SIGNAL
drm_syncobj_timeline_signal_ioctl, DRM_RENDER_ALLOW),

### SYNCOBJ_QUERY
drm_syncobj_query_ioctl, DRM_RENDER_ALLOW),

### CRTC_GET_SEQUENCE
drm_crtc_get_sequence_ioctl, 0),

### CRTC_QUEUE_SEQUENCE
drm_crtc_queue_sequence_ioctl, 0),

### MODE_CREATE_LEASE
drm_mode_create_lease_ioctl, DRM_MASTER),

### MODE_LIST_LESSEES
drm_mode_list_lessees_ioctl, DRM_MASTER),

### MODE_GET_LEASE
drm_mode_get_lease_ioctl, DRM_MASTER),

### MODE_REVOKE_LEASE
drm_mode_revoke_lease_ioctl, DRM_MASTER),

</details>
