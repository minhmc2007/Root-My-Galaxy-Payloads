# a55x-A556E-6.1.157-20260703

Offline port profile for Samsung Galaxy A55 5G (SM-A556E, a55x platform,
Exynos s5e5545). Derived from the user-supplied `boot.img.lz4` following
[`docs/PORTING.md`](../../../docs/PORTING.md). Hardware validation has NOT
been performed yet.

## Firmware identity (from the boot image)

```text
boot.img size      : 67108864
boot.img SHA-256   : 586E402F109D8514797A37837552AA297D70FADDE4B8E3B3FE8553F9850EC189
kernel size        : 38697472
kernel SHA-256     : 43C03309E7F5F5BAC314239BA0D3B1086D1B1EB0D34AE12E4B36AD9D4CF6C787
kernel release     : 6.1.157-android14-11
kernel build       : #1 SMP PREEMPT Fri Jul  3 09:06:14 UTC 2026
toolchain          : Android clang 17.0.2 (r487747c), LLD 17.0.2, LTO
ARM64 Image        : text_offset 0x0, image_size 0x27a0000, flags 0xa
ELF base           : 0xffffffc008000000 (vmlinux-to-elf)
raw BTF blob       : [0x1876a88, 0x1e2a8c4), 5979708 bytes
```

Device readout: `samsung/a55xnsxx/a55x:16/BP4A.251205.006/A556EXXSGDZG2:user/release-keys`
(`getprop ro.build.fingerprint`), cross-confirmed by the sboot.bin signer
block (`SM-A556E` / `A556EXXSGDZG2`, built 20260703183749). The kernel
build date indicates a July 2026 security-maintenance release.

## Derived values and how

| Constant | Value | Method |
| --- | --- | --- |
| All `*_OFF` symbols | see `target.h` | `vmlinux.nm` from `vmlinux-to-elf` + `llvm-nm` |
| `ASHMEM_MISC_FOPS_OFF` | `0x02464430` | `.data` scan for `miscdevice.fops == &ashmem_fops`; name `"ashmem"`, minor `MISC_DYNAMIC_MINOR`; `kallsyms` does not track `ashmem_misc` on this build |
| `SLIDE_NFULNL_LOGGER_NAME_OFF` | `0x016ca0d5` | first qword of `nfulnl_logger` -> string, verified |
| `SLIDE_RANDOM_TABLE_BOOT_ID_DATA_PTR_OFF` | `0x0241e878` | `random_table[]` walk, stride `0x40`, entry `boot_id` idx 4; `.data` value == `&sysctl_bootid` verified |
| `SLIDE_TRACEFS_EVENT_ID` | `106` | `__TRACE_LAST_TYPE` (20) + (`__event_sched_blocked_reason` - `__start_ftrace_events`) / 8 = 20 + 86 |
| `SLIDE_TRACEFS_WORKER_CALLER_OFF` | `0x000dbc94` | capstone: return address of `bl schedule` in `worker_thread` |
| `SELINUX_ENFORCING_OFF` | `0x025c92f8` | `selinux_state + 0x0`; cross-checked: `sel_read_enforce` ldrb and `sel_write_enforce` strb both use displacement 0 |
| `COMPACT_RT_MUTEX_WAITER` | `1` | target BTF: `sizeof(struct rt_mutex_waiter) == 0x58`, `prio` at `0x44`, `deadline` at `0x48` |
| fops/task_struct/page layouts | see `target.h` | target BTF (this kernel emits bit-scaled member offsets with `kflag=0` on some structs; normalized by /8 with non-aligned rejection) |
| `p0_fingerprint.h` | 32 slides | raw Image qwords at slide + {0x000..0xe00, step 0x200} |

## Values pending hardware confirmation

- `P0_KERNEL_PHYS_LOAD` `0x80000000`: **confirmed** from the a55x
  `sboot.bin` (`A556EXXSGDZG2`): the pre-jump branch computes
  `entry = 0x80000000 + g_slot` with `x0 = 0xA0000000` (DTB), and
  `g_slot = (SMC 0xC200101C ret & 0x1F) << 16` selects one of 32
  64 KiB slots over `[0x80000000, 0x801F0000]`; the 32-slide
  fingerprint table covers exactly that range.
- `SLIDE_PSELECT_WORD_SHIFT` `3`: matches every device-tested 6.1 compact
  target; verify with the documented kworker-caller 64 KiB alignment check.
- `SKB_DATA_DELTA` `-0x1000`, `MM_STRUCT_SZ` `0x400`: 6.1 compact family
  values, device-tested on S926B/S921B.
- KernelSU late-load module: must be built with exact vermagic
  `6.1.157-android14-11 SMP preempt mod_unload modversions aarch64` and
  audited against the recovered `vmlinux.elf` per
  [`kernelsu/README.md`](../../../kernelsu/README.md).

## Build

```sh
make TARGET=a55x-A556E-6.1.157-20260703 ANDROID_NDK_HOME=/path/to/android-ndk release
```
