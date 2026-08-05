# SM-A556E A556EXXSGDZG2 target record

This record contains the exact offline inputs and derived values for profile
`a55x-A556EXXSGDZG2`. Values were recovered from the stock images supplied for
this port; none were selected by model name alone. Hardware validation is
pending on a physical SM-A556E; no on-device result is claimed in this record.

## Firmware identity

```text
model: SM-A556E
device: a55x
display build: A556EXXSGDZG2
fingerprint: samsung/a55xnsxx/a55x:16/BP4A.251205.006/A556EXXSGDZG2:user/release-keys
SoC: Exynos 1480 (s5e8845)
kernel release: 6.1.157-android14-11
kernel build: #1 SMP PREEMPT Fri Jul  3 09:06:14 UTC 2026
ABI: arm64-v8a
page size: 4096
```

## Firmware source

Stock files:

```text
boot.img size: 67108864 (64 MiB)
boot.img SHA-256: 586E402F109D8514797A37837552AA297D70FADDE4B8E3B3FE8553F9850EC189
vendor_boot.img size: 67108864 (64 MiB)
vendor_boot.img SHA-256: ECD132312203F78735EA575F7897E44D4C59A16F514B52ACFEEF879559DA23C4
boot header version: 4
kernel size: 38697472
kernel SHA-256: 43C03309E7F5F5BAC314239BA0D3B1086D1B1EB0D34AE12E4B18AD9D4CF6C787
```

The kernel begins at boot-image offset `0x1000` and is byte-for-byte identical
to `analysis/ida/boot-kernel.bin`. The DTB root node reports `compatible`
`samsung,armv8` / `samsung,s5e8845`. The boot signer block confirms the exact
image identity: `SM-A556E_MEA`, build `A556EXXSGDZG2`, date `20260703183802`.

## Recovered artifacts

`analysis/ida/` contains the recovered and validated analysis set:

```text
vmlinux.elf        recovered ELF
vmlinux.nm         symbol table with exact virtual addresses
vmlinux.btf        BTF section
vmlinux-btf.raw    raw BTF
vmlinux-btf.h      generated BTF header
```

All offsets follow the `KIMAGE_TEXT_BASE 0xffffffc008000000` plus `*_OFF`
convention: an offset `O` in this record corresponds to virtual address
`0xffffffc008000000 + O` in the running kernel.

## Target derivation

The KASLR slide constants:

| Macro | Exact derivation | Value |
| --- | --- | ---: |
| `SLIDE_TRACEFS_EVENT_ID` | `__event_sched_blocked_reason` at event index 86; `__TRACE_LAST_TYPE == 20` | `106` |
| `SLIDE_TRACEFS_WORKER_CALLER_OFF` | `worker_thread` `bl schedule`; saved return instruction at `0xffffffc0080dbc94` | `0x000dbc94` |
| `SLIDE_PSELECT_WORD_SHIFT` | waiter at `E-0x1e8`, fd-set bits at `E-0x200`; futex chain `0x70+0x60+0x1b0-0x98`, pselect `0x90+0x1c0-0x50` | `3` |
| `COMPACT_RT_MUTEX_WAITER` | compact `rt_mutex_waiter` size 88 (`0x58`) | `1` |
| `SKB_DATA_DELTA` | `skb_shared_info` 344 bytes, order-3 page `0x1000` | `-0x1000LL` |
| `KERNELSNITCH_FUTEX_HASH_SIZE` | `futex_init` in `kernel/futex/core.c`; the s5e8845 DT exposes 8 possible CPUs, so `roundup_pow_of_two(256 * num_possible_cpus())` = `roundup_pow_of_two(2048)` = `2048` (`0x800`) | `0x800` |

For comparison, the Exynos 2400 (`e1s`/`e2s`) targets expose 10 possible CPUs, giving `roundup_pow_of_two(2560)` = `4096` (`0x1000`); the A55E hash size must *not* be copied from them.

Symbol offsets, taken from the actual `vmlinux.nm` virtual addresses:

| Macro/use | Exact derivation | Offset |
| --- | --- | ---: |
| `CALL_USERMODEHELPER_EXEC_WORK_OFF` | `call_usermodehelper_exec_work` | `0x000d4360` |
| `NOOP_LLSEEK_OFF` | `noop_llseek` | `0x0039ec9c` |
| `COPY_SPLICE_READ_OFF` | `generic_file_splice_read` fallback | `0x003ec8a0` |
| `CONFIGFS_READ_ITER_OFF` | `configfs_read_iter` | `0x0046cce8` |
| `CONFIGFS_BIN_WRITE_ITER_OFF` | `configfs_bin_write_iter` | `0x0046d218` |
| `SLIDE_RANDOM_TABLE_BOOT_ID_DATA_PTR_OFF` | `random_table` + `0x108` | `0x0041e878` |
| `ASHMEM_IOCTL_OFF` | `ashmem_ioctl` | `0x00d2d4f8` |
| `ASHMEM_COMPAT_IOCTL_OFF` | `compat_ashmem_ioctl` | `0x00d2de30` |
| `ASHMEM_MMAP_OFF` | `ashmem_mmap` | `0x00d2de88` |
| `ASHMEM_OPEN_OFF` | `ashmem_open` | `0x00d2e0a8` |
| `ASHMEM_RELEASE_OFF` | `ashmem_release` | `0x00d2e130` |
| `ASHMEM_SHOW_FDINFO_OFF` | `ashmem_show_fdinfo` | `0x00d2e250` |
| `ANON_PIPE_BUF_OPS_OFF` | `anon_pipe_buf_ops` | `0x0120c610` |
| `SLIDE_NFULNL_LOGGER_NAME_OFF` | `"nfnetlink_log"` referenced by `nfulnl_logger.name` | `0x016ca0d5` |
| `KMALLOC_CACHES_OFF` | `kmalloc_caches` | `0x01792ed8` |
| `ASHMEM_MISC_FOPS_OFF` / `ASHMEM_FOPS_OFF` | `ashmem_fops` | `0x013c9b50` |
| `SYSTEM_UNBOUND_WQ_OFF` | `system_unbound_wq` | `0x022cae60` |
| `SLIDE_NFULNL_LOGGER_OBJECT_OFF` | `nfulnl_logger` | `0x022d29e0` |
| `INIT_TASK_OFF` | `init_task` | `0x022df700` |
| `ROOT_TASK_GROUP_OFF` | `root_task_group` | `0x024f4d40` |
| `SLIDE_SYSCTL_BOOTID_OFF` | `sysctl_bootid` | `0x026b0518` |
| `SELINUX_ENFORCING_OFF` | `selinux_state.enforcing` | `0x025c92f8` |

The `nf_conntrack` helpers are present in the recovered ELF
(`nf_conntrack_alloc` at `0xffffffc008f06094`, `nf_conntrack_hash` at
`0xffffffc00a2d2a08`, both confirmed in `vmlinux.nm`); no
nf_conntrack-derived constant is consumed by this target's feature set.

Target BTF confirms the same exploit-relevant 6.1 layouts verified on the e2s
target: `file_operations` is `0x110`, `page`/`slab` is `0x40`,
`miscdevice.fops` is `0x10`, and all task, configfs, workqueue and pipe offsets
in `target.h` match this Image. In particular, `rt_mutex_waiter` is the compact
`0x58`-byte layout with `pi_tree_entry=0x18`, `task=0x30`, `lock=0x38`,
`wake_state=0x40`, `prio=0x44`, `deadline=0x48` and `ww_ctx=0x50`.

The P0 fingerprint table is `src/targets/a55x-A556EXXSGDZG2/p0_fingerprint.h`,
generated from `analysis/ida/boot-kernel.bin` at source offsets `0x000`
through `0xe00` (eight words per slide, 32 slide candidates). All 256 emitted
qwords were compared back to their source offsets; the table is verified 256/256
qwords. SHA-256 of the generated header:

```text
6DD649FCBE1239DA4891DF58A2C8BE4F6986B04F1120D0C3C6CABCE95523CA62
```

## Payload & build status

The `src/targets/a55x-A556EXXSGDZG2/` profile is complete:

```text
target.h SHA-256: 8E9D05C29D904E544F088AC284DB207E9AB32975DEB7E5DDA45EE04187A760A6
p0_fingerprint.h SHA-256: 6DD649FCBE1239DA4891DF58A2C8BE4F6986B04F1120D0C3C6CABCE95523CA62
```

`target.h` carries the full APP_PAYLOAD feature set, including the
physical-P0-oracle gates (`APP_PHYS_P0_ORACLE`, `APP_REQUIRE_FRESH_P0_SESSION`)
and the complete fingerprint, alias, CFI, production-slot and mirror
configuration.
