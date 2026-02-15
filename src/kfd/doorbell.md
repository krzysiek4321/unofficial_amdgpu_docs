# Doorbells
There is a maximum of 1024 queues per process.
Each is assigned a doorbell.
A doorbell can be 32 or 64 bits wide.

So mapping `mmap()` would need to be round_up(1024 * single_doorbell_size_in_bytes, PAGE_SIZE) in size.

### How can I tell if doorbells are 32 or 64 bits wide for a given device?
Don't know yet.

I believe they are always 32bit because you are supposed to write a new wptr value and wptr is u32.

## Whais is it for?
It's purpose is to notify the gpu when we wrote new commands into a queue.
We write the new "wptr" value into a doorbell for a given queue.

## bitmap
It's 1024 bits, split into 2 512 bit parts, the seccond called **mirror**, set the same way first part is.
