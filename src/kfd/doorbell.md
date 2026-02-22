# Doorbells
There is a maximum of 1024 queues per process.
Each is assigned a doorbell.

They are automatically created with queues.

## Size
Doorbell size is device dependent.
For < gfx901 it's 4 bytes.
For gfx901+ it's 8 bytes.

So mapping `mmap()` would need to be `2 * PAGE_SIZE` in size for gfx901+ and `PAGE_SIZE` for older engines.

### Why are doorbells 8 bytes for all newer gpu if a queue has size in u32 and `*wptr` is an index?

## Index
### How can I tell which address from the mmap doorbells page or pages to write the new wptr to?
Is it as simple as just `idx = offset & SIZE`?

## Whais is it for?
It's purpose is to notify the gpu when we wrote new commands into a queue.
We write the new "wptr" value into a doorbell for a given queue.

## bitmap
It's 1024 bits, split into 2 512 bit parts, the seccond called **mirror**, set the same way first part is.
