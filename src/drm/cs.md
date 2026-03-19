# Command Submission

### Job number requirements and limits
A sumbission must have at least one jobs (IB).

For devices with Single Root I/O Virtualization Virtual Function (**SRIOV_VF**)
there must be **exactly one job**.

There is a limit to at most 4 jobs (IBs) in a submission.
And there is a limit of at most 4 different entities (rings) used by these jobs.

##### How do I check if GPU is sriov_vf?
todo

### Job validation
For some rings the IB content might be validated (parse_cs) or changed (patch_cs_in_place)
by ring driver.

## Enforce isolation
In most cases, by default jobs are executed one after another without cleaning used registers and memory.

For GFX rings isolation is always on (=1).

You can choose to enable enforcing isolation by writing
isolation policy value into
`/sys/class/drm/*/device/enforce_isolation`

Policy values:
- 0 - no isolation
- 1 - isolation
- 2 - legacy isolation
- 3 - isolation, no cleaner shader

## User fence
A sumission can have a user fence which is a single uint64 value
in a special non userptr BO sized PAGE_SIZE.

Current's submission fence handle (seqno) is sent to the ring
I imagine it will write that value into the user fence when job is done.

##### What can I do with it? How is it useful?
todo

## IB flags
### Constant Engine (CE) / Drawing Engine (DE)
Since GCN1 there are two parallel engines fed from primary ring buffer.

Constant Engine allows to pre load data into caches that will be used
by the Drawing Engine, while the Drawing Engine is still busy with previous submission.

To do this you need to submit two IBs, one with AMDGPU_IB_FLAG_CE and one without.
If there is a CE IB (called a CONST_IB), it will be put on the ring prior to the DE IB.

## Context lost / GPU resets
During a submission if a ctx becomes invalid you'll get ECANCELED.
If you already submitted jobs to the gpu and a ctx becomes invalid
the jobs will have -ECANCELLED in their fences written and not be rerun.

## Sync objects

## Synchronizing with other submissions
