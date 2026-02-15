# Useful tips
These are some linux kernel features which might be helpful to stydy amdgpu kernel module.

## Tracefs
We can use tracefs to veryfy at runtime the callstack
of driver functions, we expect to be executed.

1. Have a program triggering some event, for example a kernel function call
2. Have sufficient permissions to modify files in /sys/kernel/tracing or /sys/kernel/debug/tracing.
3. Navigate to the tracefs dir.
4. Disable tracing
`echo 0 > tracing_on`
5. Enable a tracing event.

Pick one from `events/` dir end set enable to 1 or create a custom event.

5. Set tracer to trace function nesting.
`echo function_graph > current_tracer`
6. Set graph tracer to filter the callchain to a specific function.
`echo kfd_ioctl_create_queue > set_graph_function`
7. If you need more controll on what triggers the tracer or how deep it should look checkout the `README` file in tracefs.
8. Enable tracing
`echo 1 > tracing_on`
9. Run the program
10. Optionally, disable tracing
11. Read the `trace` file

## Dyndebug
In order to not clutter the kernel dmesg buffer most messages are surpressed.
They can be enabled at runtime by writing to `/proc/dynamic_debug/control` file.

Requires root access.

Read more at <https://docs.kernel.org/admin-guide/dynamic-debug-howto.html#dynamic-debug>.
