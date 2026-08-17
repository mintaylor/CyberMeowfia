# Porting Notes

Notes for porting this exploit to another device or firmware version.

## Slide parameters

On Samsung devices the bootloader randomizes the kernel image's physical
load per boot (the `nokaslr` cmdline only disables virtual KASLR), so the
runtime slide must be resolved. Two per-kernel-binary parameters are
needed for the slide leak:

| Parameter                         | Meaning                                                                            |
| --------------------------------- | ---------------------------------------------------------------------------------- |
| `SLIDE_TRACEFS_EVENT_ID`          | runtime id of the `sched_blocked_reason` trace event                               |
| `SLIDE_TRACEFS_WORKER_CALLER_OFF` | link offset of the instruction after the blocking `bl schedule` in `worker_thread` |

Both are fingerprints of the kernel binary, not of the device: they can
change across firmware updates and must be re-derived per kernel.

### EVENT_ID

Prefer reading it from the device (authoritative, no tooling):

```sh
adb shell cat /sys/kernel/tracing/events/sched/sched_blocked_reason/id
```

Offline derivation (no device):

1. Extract kallsyms from the boot image kernel.
2. `event_index = (__event_sched_blocked_reason - __start_ftrace_events) / 8`
3. `event_id = __TRACE_LAST_TYPE + event_index`, where `__TRACE_LAST_TYPE`
   is the last value of `enum trace_type` in `kernel/trace/trace.h`
   (20 on both Android 6.1 and 6.12).

### CALLER_OFF

1. From kallsyms: link offsets of `worker_thread` and `schedule`.
2. Disassemble `worker_thread`, find the `bl schedule` on the blocking path
   (the branch target must match the `schedule` offset).
3. `CALLER_OFF` = address of the instruction AFTER the `bl` (the recorded
   `caller` is the return PC, not the `bl` itself).

Worked example (current target):

```text
kallsyms: worker_thread = 0x1037cc, schedule = 0x121d738
disasm:   0x103874: bl 0x121d738     <- blocking-path schedule call
          0x103878: mov x0, x19      <- return PC (instruction after bl)
=> CALLER_OFF = 0x103878
```

Sanity check on device: `observed caller - (KIMAGE_TEXT_BASE + CALLER_OFF)`
must be 64KB-aligned and <= `0x1f0000`.

Reference (current target):

| Device              | Kernel              | EVENT_ID | CALLER_OFF |
| ------------------- | ------------------- | -------- | ---------- |
| `m3q-S9480ZCS4AZG1` | 6.12.30-android16-5 | 110      | 0x103878   |

## BTF

Struct offsets come from the BTF embedded in the firmware kernel image.
BTF reference values (m3q, 6.12.30-android16-5):

| Field                                           | BTF offset |
| ----------------------------------------------- | ---------- |
| `prio` (`FAKE_TASK_PRIO_OFF`)                   | 0x94       |
| `pi_lock` (`FAKE_TASK_PI_LOCK_OFF`)             | 0x9ec      |
| `pi_waiters` (`FAKE_TASK_PI_WAITERS_OFF`)       | 0xa00      |
| `pi_top_task` (`FAKE_TASK_PI_TOP_TASK_OFF`)     | 0xa10      |
| `pi_blocked_on` (`FAKE_TASK_PI_BLOCKED_ON_OFF`) | 0xa18      |
| `pipe_inode_info.tmp_page`                      | 0x90       |
| `pipe_buffer.flags`                             | 0x18       |

These are reference values only and are NOT written into the m3q
target.h. Only `pipe_buffer.flags` (0x18, device-verified) is used by the
current route; re-check the rest when changing the route or porting to a
new kernel family.
