# Useful tips
These are some linux kernel features which might be helpful to stydy amdgpu kernel module.

## Tracefs

### Function graphs
We can use tracefs to veryfy at runtime the callstack
of driver functions, we expect to be executed.

For example, run as root:
```sh
trace-cmd record -p function_graph -g kfd_ioctl_acquire_vm -n _printk --max-graph-depth=6
trace-cmd report
```

## Dyndebug
In order to not clutter the kernel dmesg buffer most messages are surpressed.
They can be enabled at runtime by writing to `/proc/dynamic_debug/control` file.

Requires root access.

For example, to enable all amdgpu messages use:
```sh
echo 'file *amdgpu* +p'
```

But you probably want to limit which events get printed.

Read more at <https://docs.kernel.org/admin-guide/dynamic-debug-howto.html#dynamic-debug>.

Unfortunetelly there is no mechanism to filter by process id
