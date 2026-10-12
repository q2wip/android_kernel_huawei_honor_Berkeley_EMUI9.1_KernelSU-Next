# ReSukiSU / BakaSU manual-hook patches (Kirin 970 / 4.9)

These patches add the **BakaSU (ReSukiSU) manual hooks** required for the
non-GKI manual-hook mode on Huawei Kirin 970 (Linux 4.9) kernels.

They are applied on top of the base kernel tree by
`.github/workflows/Build Honor_Berkeley ReSukiSU Kernel.yml` before the
kernel is built, using `patch -p1`.

## Hook contract

Per `https://bakasu.org/guide/manual-integrate.html`, when
`CONFIG_KSU_MANUAL_HOOK=y` and the `AUTO_*` helpers are enabled, BakaSU
requires these hooks to exist in the kernel source:

| Patch | File | Symbols |
|-------|------|---------|
| 0001  | `fs/exec.c` | `ksu_handle_execveat` (hooked in `do_execve` + `compat_do_execve`) |
| 0002  | `fs/stat.c` | `ksu_handle_stat`, `ksu_handle_newfstat_ret`, `ksu_handle_fstat64_ret` |
| 0003  | `fs/open.c` | `ksu_handle_faccessat` |
| 0004  | `kernel/reboot.c` | `ksu_handle_sys_reboot` |
| 0005  | `security/selinux/hooks.c` | su domain transition in `selinux_bprm_set_creds` |

`CONFIG_KSU_MANUAL_HOOK_AUTO_SETUID_HOOK=y`,
`CONFIG_KSU_MANUAL_HOOK_AUTO_INITRC_HOOK=y` and
`CONFIG_KSU_MANUAL_HOOK_AUTO_INPUT_HOOK=y` cover the setuid / sys_read /
input hooks, so no manual patch is needed for
`kernel/sys.c`, `fs/read_write.c` or `drivers/input/input.c`.

`CONFIG_KALLSYMS_ALL=y` is required so that the static-symbol-export
patches (selinuxfs / ss/services) can be skipped.

## Notes

- The hook set was derived from xixiaobei-bei/KernelSU_on_Huawei branch
  `SEA-AL10-TL10-TL10HKA_EMUI9.1_ReSukiSU` (a known-working 4.9
  ReSukiSU tree); only the `CONFIG_KSU`/`ksu_handle` hunks were kept, so
  the unrelated vendor changes (HWAA, BFMR, MS_SUBMOUNT, ...) are **not**
  pulled in.
- Do **not** add `ksu_vfs_read_hook` / `is_ksu_transition`; BakaSU treats
  those as incompatible.
