# Dockerコンテナ(ランタイム)
docker runコマンドでコンテナの生成ができます。

コンテナの生成はcontainerdが行っています。

## docker run

まずはdocker runをします

```
root@ubuntu-bionic:~# docker run --name test1 -it debian
root@1d7cad3c59e8:/#
```


この例ではdebian:latest イメージを使い、 test1というコンテナを実行します。

-it は疑似 TTY（pseudo-TTY）をコンテナの標準入力に接続するよう、 Docker に対して命令します。

https://docs.docker.jp/engine/reference/commandline/run.html

psを見てみます。

```
root       944  0.0  4.6 985752 47252 ?        Ssl  12:46   0:00 /usr/bin/containerd
root      2923  0.0  0.5 108660  5320 ?        Sl   12:56   0:00  \_ containerd-shim -namespace moby -workdir /var/lib/containerd/io.containerd.runtime.v1.linux/moby/1d7cad3c59e8c948f5d03bc414aa2a952eb3704069c0a4eed1fa72f842f04209 -address /run/containerd/containerd.sock -containerd-binary /usr/bin/containerd -runtime-root /var/run/docker/runtime-runc
root      2955  0.0  0.3   3868  3028 pts/0    Ss+  12:56   0:00      \_ bash

root      2572  0.0  0.7 107992  7240 ?        Ss   12:53   0:00  \_ sshd: vagrant [priv]
vagrant   2653  0.0  0.5 108352  5112 ?        S    12:53   0:00      \_ sshd: vagrant@pts/2
vagrant   2654  0.0  0.5  23220  5044 pts/2    Ss   12:53   0:00          \_ -bash
root      2682  0.0  0.4  63976  4140 pts/2    S    12:54   0:00              \_ sudo -s
root      2683  0.0  0.5  23148  5216 pts/2    S    12:54   0:00                  \_ /bin/bash
root      2906  0.0  5.8 855180 59132 pts/2    Sl+  12:56   0:00                      \_ docker run --name test1 -it debian
```

containerdの子プロセスとして、コンテナが立ち上がっています。

このプロセスがどのように生成されたかをstraceでみてみました。

ちょっと長かったので必要そうな部分を抜き出してみます。

今までの総決算みたいな感じになります。

## システムコールの流れ

```
root@ubuntu-bionic:~# strace -Ttt -ff -s 2048 -p 944
strace: Process 944 attached with 12 threads
[pid  2853] 12:56:27.088012 restart_syscall(<... resuming interrupted futex ...> <unfinished ...>
[pid  2770] 12:56:27.088398 futex(0xc000498f48, FUTEX_WAIT_PRIVATE, 0, NULL <unfinished ...>
[pid  1115] 12:56:27.088668 futex(0xc0004984c8, FUTEX_WAIT_PRIVATE, 0, NULL <unfinished ...>
[pid  1158] 12:56:27.089073 futex(0xc000499648, FUTEX_WAIT_PRIVATE, 0, NULL <unfinished ...>
[pid  1124] 12:56:27.089449 epoll_pwait(4,  <unfinished ...>
[pid  1085] 12:56:27.089705 restart_syscall(<... resuming interrupted futex ...> <unfinished ...>
[pid  1114] 12:56:27.089895 futex(0x55bf8c83e150, FUTEX_WAKE_PRIVATE, 1 <unfinished ...>


[pid  1124] 12:56:29.698267 pwrite64(3, "\4\0\0\0\0\0\0\0\2\0\2\0\0\0\0\0\0\0\0\0 \0\0\0\t\0\0\0\17\0\0\0\1\0\0\0(\0\0\0\6\0\0\0[\0\0\0createdat\1\0\0\0\16\327K\5})\235\302\260\377\377labels\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\2\0\1\0\0\0\0\0\0\0\0\0\20\0\0\0\27\0\0\0\24\0\0\0containerd.io/gc.expire2020-11-22T12:56:29Z\0\0\0\0\0\0\0\0\0\0\0\
```

```
[pid  2923] 12:56:29.764409 chdir("/run/containerd/io.containerd.runtime.v1.linux/moby/1d7cad3c59e8c948f5d03bc414aa2a952eb3704069c0a4eed1fa72f842f04209") = 0 <0.000113>
[pid  2923] 12:56:29.766225 execve("/usr/bin/containerd-shim", ["containerd-shim", "-namespace", "moby", "-workdir", "/var/lib/containerd/io.containerd.runtime.v1.linux/moby/1d7cad3c59e8c948f5d03bc414aa2a952eb3704069c0a4eed1fa72f842f04209", "-address", "/run/containerd/containerd.sock", "-containerd-binary", "/usr/bin/containerd", "-runtime-root", "/var/run/docker/runtime-runc"], 0xc0006705c0 /* 7 vars */ <unfinished ...>
[pid  2853] 12:56:29.768134 renameat(AT_FDCWD, "/run/containerd/io.containerd.runtime.v1.linux/moby/1d7cad3c59e8c948f5d03bc414aa2a952eb3704069c0a4eed1fa72f842f04209/.shim.pid", AT_FDCWD, "/run/containerd/io.containerd.runtime.v1.linux/moby/1d7cad3c59e8c948f5d03bc414aa2a952eb3704069c0a4eed1fa72f842f04209/shim.pid") = 0 <0.000018>
[pid  2931] 12:56:29.901077 execve("/usr/bin/runc", ["runc", "--root", "/var/run/docker/runtime-runc/moby", "--log", "/run/containerd/io.containerd.runtime.v1.linux/moby/1d7cad3c59e8c948f5d03bc414aa2a952eb3704069c0a4eed1fa72f842f04209/log.json", "--log-format", "json", "create", "--bundle", "/run/containerd/io.containerd.runtime.v1.linux/moby/1d7cad3c59e8c948f5d03bc414aa2a952eb3704069c0a4eed1fa72f842f04209", "--pid-file", "/run/containerd/io.containerd.runtime.v1.linux/moby/1d7cad3c59e8c948f5d03bc414aa2a952eb3704069c0a4eed1fa72f842f04209/init.pid", "--console-socket", "/tmp/pty185923347/pty.sock", "1d7cad3c59e8c948f5d03bc414aa2a952eb3704069c0a4eed1fa72f842f04209"], 0xc00006c600 /* 6 vars */ <unfinished ...>
[pid  2931] 12:56:29.916500 clone(child_stack=0x7fb57b36afb0, flags=CLONE_VM|CLONE_FS|CLONE_FILES|CLONE_SIGHAND|CLONE_THREAD|CLONE_SYSVSEM|CLONE_SETTLS|CLONE_PARENT_SETTID|CLONE_CHILD_CLEARTID, parent_tidptr=0x7fb57b36b9d0, tls=0x7fb57b36b700, child_tidptr=0x7fb57b36b9d0) = 2933 <0.000040>
[pid  2931] 12:56:29.918454 clone(child_stack=0x7fb579b67fb0, flags=CLONE_VM|CLONE_FS|CLONE_FILES|CLONE_SIGHAND|CLONE_THREAD|CLONE_SYSVSEM|CLONE_SETTLS|CLONE_PARENT_SETTID|CLONE_CHILD_CLEARTID, parent_tidptr=0x7fb579b689d0, tls=0x7fb579b68700, child_tidptr=0x7fb579b689d0) = 2936 <0.000030>
```


```
[pid  2938] 12:56:29.977074 mknodat(AT_FDCWD, "/var/run/docker/runtime-runc/moby/1d7cad3c59e8c948f5d03bc414aa2a952eb3704069c0a4eed1fa72f842f04209/exec.fifo", S_IFIFO|0622) = 0 <0.000021>
[pid  2938] 12:56:29.977137 umask(022)  = 000 <0.000005>
[pid  2938] 12:56:29.977169 fchownat(AT_FDCWD, "/var/run/docker/runtime-runc/moby/1d7cad3c59e8c948f5d03bc414aa2a952eb3704069c0a4eed1fa72f842f04209/exec.fifo", 0, 0, 0) = 0 <0.000010>
[pid  2939] 12:56:30.000273 execve("/proc/self/exe", ["runc", "init"], 0xc000098af0 /* 8 vars */) = 0 <0.000299>
[pid  2939] 12:56:30.005776 mount("/proc/self/exe", "/var/run/docker/runtime-runc/moby/1d7cad3c59e8c948f5d03bc414aa2a952eb3704069c0a4eed1fa72f842f04209/runc.DNQYgB", 0x559db0661329, MS_BIND, 0x559db0661329) = 0 <0.002024>
[pid  2939] 12:56:30.007857 mount("", "/var/run/docker/runtime-runc/moby/1d7cad3c59e8c948f5d03bc414aa2a952eb3704069c0a4eed1fa72f842f04209/runc.DNQYgB", 0x559db0661329, MS_RDONLY|MS_REMOUNT|MS_BIND, "") = 0 <0.000960>
[pid  2939] 12:56:30.008954 umount2("/var/run/docker/runtime-runc/moby/1d7cad3c59e8c948f5d03bc414aa2a952eb3704069c0a4eed1fa72f842f04209/runc.DNQYgB", MNT_DETACH <unfinished ...>
[pid  2938] 12:56:30.016222 mkdirat(AT_FDCWD, "/sys/fs/cgroup/cpuset/docker/1d7cad3c59e8c948f5d03bc414aa2a952eb3704069c0a4eed1fa72f842f04209", 0755) = 0 <0.000081>
[pid  2938] 12:56:30.022097 mkdirat(AT_FDCWD, "/sys/fs/cgroup/devices/docker/1d7cad3c59e8c948f5d03bc414aa2a952eb3704069c0a4eed1fa72f842f04209", 0755) = 0 <0.000038>
[pid  2939] 12:56:30.026437 unlink("/var/run/docker/runtime-runc/moby/1d7cad3c59e8c948f5d03bc414aa2a952eb3704069c0a4eed1fa72f842f04209/runc.DNQYgB" <unfinished ...>
[pid  2938] 12:56:30.026754 mkdirat(AT_FDCWD, "/sys/fs/cgroup/memory/docker/1d7cad3c59e8c948f5d03bc414aa2a952eb3704069c0a4eed1fa72f842f04209", 0755) = 0 <0.000056>
[pid  2938] 12:56:30.035251 mkdirat(AT_FDCWD, "/sys/fs/cgroup/cpu,cpuacct/docker/1d7cad3c59e8c948f5d03bc414aa2a952eb3704069c0a4eed1fa72f842f04209", 0755) = 0 <0.000046>
[pid  2938] 12:56:30.045167 mkdirat(AT_FDCWD, "/sys/fs/cgroup/pids/docker/1d7cad3c59e8c948f5d03bc414aa2a952eb3704069c0a4eed1fa72f842f04209", 0755 <unfinished ...>
[pid  2938] 12:56:30.047955 mkdirat(AT_FDCWD, "/sys/fs/cgroup/blkio/docker/1d7cad3c59e8c948f5d03bc414aa2a952eb3704069c0a4eed1fa72f842f04209", 0755) = 0 <0.000057>
[pid  2938] 12:56:30.051800 mkdirat(AT_FDCWD, "/sys/fs/cgroup/hugetlb/docker/1d7cad3c59e8c948f5d03bc414aa2a952eb3704069c0a4eed1fa72f842f04209", 0755) = 0 <0.000039>
[pid  2938] 12:56:30.056247 mkdirat(AT_FDCWD, "/sys/fs/cgroup/net_cls,net_prio/docker/1d7cad3c59e8c948f5d03bc414aa2a952eb3704069c0a4eed1fa72f842f04209", 0755) = 0 <0.000054>
[pid  2938] 12:56:30.062617 mkdirat(AT_FDCWD, "/sys/fs/cgroup/perf_event/docker/1d7cad3c59e8c948f5d03bc414aa2a952eb3704069c0a4eed1fa72f842f04209", 0755) = 0 <0.000043>
[pid  2938] 12:56:30.065627 mkdirat(AT_FDCWD, "/sys/fs/cgroup/freezer/docker/1d7cad3c59e8c948f5d03bc414aa2a952eb3704069c0a4eed1fa72f842f04209", 0755) = 0 <0.000041>
```


### unshare(2)とclone(2)を使ったプロセスの隔離

```
[pid  2939] 12:56:30.075595 prctl(PR_SET_NAME, "runc:[0:PARENT]"...) = 0 <0.000071>
[pid  2939] 12:56:30.075841 clone(strace: Process 2949 attached
 <unfinished ...>
[pid  2949] 12:56:30.078600 prctl(PR_SET_NAME, "runc:[1:CHILD]") = 0 <0.000008>
[pid  2949] 12:56:30.078649 unshare(CLONE_NEWNS|CLONE_NEWUTS|CLONE_NEWIPC|CLONE_NEWNET|CLONE_NEWPID) = 0 <0.000582>
[pid  2949] 12:56:30.079282 clone(child_stack=0x7ffc2401e670, flags=CLONE_PARENT|SIGCHLD) = 2955 <0.000160>
strace: Process 2955 attached
```



```
[pid  2955] 12:56:30.193452 keyctl(KEYCTL_JOIN_SESSION_KEYRING, "_ses.1d7cad3c59e8c948f5d03bc414aa2a952eb3704069c0a4eed1fa72f842f04209") = 597974975 <0.000166>
[pid  2955] 12:56:30.193819 keyctl(KEYCTL_DESCRIBE, 597974975, "", 0) = 91 <0.000121>
[pid  2955] 12:56:30.194130 keyctl(KEYCTL_DESCRIBE, 597974975, "keyring;0;0;3f130000;_ses.1d7cad3c59e8c948f5d03bc414aa2a952eb3704069c0a4eed1fa72f842f04209", 91) = 91 <0.000119>
[pid  2955] 12:56:30.194389 keyctl(KEYCTL_SETPERM, 597974975, KEY_POS_VIEW|KEY_POS_READ|KEY_POS_WRITE|KEY_POS_SEARCH|KEY_POS_LINK|KEY_POS_SETATTR|KEY_USR_VIEW|KEY_USR_READ|KEY_USR_SEARCH|KEY_USR_LINK) = 0 <0.000058>
[pid  2955] 12:56:30.194594 socket(AF_NETLINK, SOCK_RAW, NETLINK_ROUTE) = 7 <0.000192>
[pid  2955] 12:56:30.195332 bind(7, {sa_family=AF_NETLINK, nl_pid=0, nl_groups=00000000}, 12) = 0 <0.000122>
```

```
[pid  2955] 12:56:30.219655 mount("/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged", "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged", 0xc000145c8a, MS_BIND|MS_REC, NULL <unfinished ...>
[pid  2955] 12:56:30.225925 mount("proc", "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/proc", "proc", MS_NOSUID|MS_NODEV|MS_NOEXEC, NULL <unfinished ...>
[pid  2955] 12:56:30.229409 mount("tmpfs", "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/dev", "tmpfs", MS_NOSUID|MS_STRICTATIME, "mode=755,size=65536k" <unfinished ...>
[pid  2955] 12:56:30.237685 mkdirat(AT_FDCWD, "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/dev/pts", 0755 <unfinished ...>
[pid  2955] 12:56:30.239339 mount("devpts", "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/dev/pts", "devpts", MS_NOSUID|MS_NOEXEC, "newinstance,ptmxmode=0666,mode=0620,gid=5" <unfinished ...>
[pid  2955] 12:56:30.244421 mount("sysfs", "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/sys", "sysfs", MS_RDONLY|MS_NOSUID|MS_NODEV|MS_NOEXEC, NULL <unfinished ...>
[pid  2955] 12:56:30.253217 mount("tmpfs", "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/sys/fs/cgroup", "tmpfs", MS_NOSUID|MS_NODEV|MS_NOEXEC, "mode=755" <unfinished ...>
[pid  2955] 12:56:30.254509 fchmodat(AT_FDCWD, "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/sys/fs/cgroup", 0555 <unfinished ...>
[pid  2955] 12:56:30.264746 mkdirat(AT_FDCWD, "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/sys/fs/cgroup/systemd", 0755 <unfinished ...>
[pid  2955] 12:56:30.266197 mount("/sys/fs/cgroup/systemd/docker/1d7cad3c59e8c948f5d03bc414aa2a952eb3704069c0a4eed1fa72f842f04209", "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/sys/fs/cgroup/systemd", 0xc000145e26, MS_RDONLY|MS_NOSUID|MS_NODEV|MS_NOEXEC|MS_BIND|MS_REC, NULL <unfinished ...>
[pid  2955] 12:56:30.266871 mount("/sys/fs/cgroup/systemd/docker/1d7cad3c59e8c948f5d03bc414aa2a952eb3704069c0a4eed1fa72f842f04209", "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/sys/fs/cgroup/systemd", 0xc000145e2b, MS_RDONLY|MS_NOSUID|MS_NODEV|MS_NOEXEC|MS_REMOUNT|MS_BIND|MS_REC, NULL <unfinished ...>
[pid  2955] 12:56:30.277906 mkdirat(AT_FDCWD, "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/sys/fs/cgroup/cpu,cpuacct", 0755 <unfinished ...>
[pid  2955] 12:56:30.279359 mount("/sys/fs/cgroup/cpu,cpuacct/docker/1d7cad3c59e8c948f5d03bc414aa2a952eb3704069c0a4eed1fa72f842f04209", "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/sys/fs/cgroup/cpu,cpuacct", 0xc000145e30, MS_RDONLY|MS_NOSUID|MS_NODEV|MS_NOEXEC|MS_BIND|MS_REC, NULL <unfinished ...>
[pid  2955] 12:56:30.279945 mount("/sys/fs/cgroup/cpu,cpuacct/docker/1d7cad3c59e8c948f5d03bc414aa2a952eb3704069c0a4eed1fa72f842f04209", "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/sys/fs/cgroup/cpu,cpuacct", 0xc000145e35, MS_RDONLY|MS_NOSUID|MS_NODEV|MS_NOEXEC|MS_REMOUNT|MS_BIND|MS_REC, NULL <unfinished ...>
[pid  2955] 12:56:30.286836 mkdirat(AT_FDCWD, "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/sys/fs/cgroup/freezer", 0755 <unfinished ...>
[pid  2955] 12:56:30.287874 mount("/sys/fs/cgroup/freezer/docker/1d7cad3c59e8c948f5d03bc414aa2a952eb3704069c0a4eed1fa72f842f04209", "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/sys/fs/cgroup/freezer", 0xc000145e3a, MS_RDONLY|MS_NOSUID|MS_NODEV|MS_NOEXEC|MS_BIND|MS_REC, NULL <unfinished ...>
[pid  2955] 12:56:30.289087 mount("/sys/fs/cgroup/freezer/docker/1d7cad3c59e8c948f5d03bc414aa2a952eb3704069c0a4eed1fa72f842f04209", "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/sys/fs/cgroup/freezer", 0xc000145e40, MS_RDONLY|MS_NOSUID|MS_NODEV|MS_NOEXEC|MS_REMOUNT|MS_BIND|MS_REC, NULL <unfinished ...>
[pid  2955] 12:56:30.296580 mkdirat(AT_FDCWD, "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/sys/fs/cgroup/blkio", 0755 <unfinished ...>
[pid  2955] 12:56:30.297333 mount("/sys/fs/cgroup/blkio/docker/1d7cad3c59e8c948f5d03bc414aa2a952eb3704069c0a4eed1fa72f842f04209", "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/sys/fs/cgroup/blkio", 0xc000145e45, MS_RDONLY|MS_NOSUID|MS_NODEV|MS_NOEXEC|MS_BIND|MS_REC, NULL <unfinished ...>
[pid  2955] 12:56:30.298960 mount("/sys/fs/cgroup/blkio/docker/1d7cad3c59e8c948f5d03bc414aa2a952eb3704069c0a4eed1fa72f842f04209", "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/sys/fs/cgroup/blkio", 0xc000145e4a, MS_RDONLY|MS_NOSUID|MS_NODEV|MS_NOEXEC|MS_REMOUNT|MS_BIND|MS_REC, NULL <unfinished ...>
[pid  2955] 12:56:30.309641 mkdirat(AT_FDCWD, "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/sys/fs/cgroup/pids", 0755 <unfinished ...>
[pid  2955] 12:56:30.311520 mount("/sys/fs/cgroup/pids/docker/1d7cad3c59e8c948f5d03bc414aa2a952eb3704069c0a4eed1fa72f842f04209", "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/sys/fs/cgroup/pids", 0xc000145e50, MS_RDONLY|MS_NOSUID|MS_NODEV|MS_NOEXEC|MS_BIND|MS_REC, NULL <unfinished ...>
[pid  2955] 12:56:30.312049 mount("/sys/fs/cgroup/pids/docker/1d7cad3c59e8c948f5d03bc414aa2a952eb3704069c0a4eed1fa72f842f04209", "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/sys/fs/cgroup/pids", 0xc000145e55, MS_RDONLY|MS_NOSUID|MS_NODEV|MS_NOEXEC|MS_REMOUNT|MS_BIND|MS_REC, NULL <unfinished ...>
[pid  2955] 12:56:30.321064 mkdirat(AT_FDCWD, "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/sys/fs/cgroup/net_cls,net_prio", 0755 <unfinished ...>
[pid  2955] 12:56:30.321591 mount("/sys/fs/cgroup/net_cls,net_prio/docker/1d7cad3c59e8c948f5d03bc414aa2a952eb3704069c0a4eed1fa72f842f04209", "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/sys/fs/cgroup/net_cls,net_prio", 0xc000145e5a, MS_RDONLY|MS_NOSUID|MS_NODEV|MS_NOEXEC|MS_BIND|MS_REC, NULL <unfinished ...>
[pid  2955] 12:56:30.322895 mount("/sys/fs/cgroup/net_cls,net_prio/docker/1d7cad3c59e8c948f5d03bc414aa2a952eb3704069c0a4eed1fa72f842f04209", "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/sys/fs/cgroup/net_cls,net_prio", 0xc000145e60, MS_RDONLY|MS_NOSUID|MS_NODEV|MS_NOEXEC|MS_REMOUNT|MS_BIND|MS_REC, NULL <unfinished ...>
[pid  2955] 12:56:30.330176 mkdirat(AT_FDCWD, "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/sys/fs/cgroup/hugetlb", 0755 <unfinished ...>

[pid  2955] 12:56:30.332333 mount("/sys/fs/cgroup/hugetlb/docker/1d7cad3c59e8c948f5d03bc414aa2a952eb3704069c0a4eed1fa72f842f04209", "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/sys/fs/cgroup/hugetlb", 0xc000145e65, MS_RDONLY|MS_NOSUID|MS_NODEV|MS_NOEXEC|MS_BIND|MS_REC, NULL <unfinished ...>
[pid  2955] 12:56:30.332793 mount("/sys/fs/cgroup/hugetlb/docker/1d7cad3c59e8c948f5d03bc414aa2a952eb3704069c0a4eed1fa72f842f04209", "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/sys/fs/cgroup/hugetlb", 0xc000145e6a, MS_RDONLY|MS_NOSUID|MS_NODEV|MS_NOEXEC|MS_REMOUNT|MS_BIND|MS_REC, NULL <unfinished ...>
[pid  2955] 12:56:30.346488 mkdirat(AT_FDCWD, "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/sys/fs/cgroup/cpuset", 0755 <unfinished ...>
[pid  2955] 12:56:30.347186 mount("/sys/fs/cgroup/cpuset/docker/1d7cad3c59e8c948f5d03bc414aa2a952eb3704069c0a4eed1fa72f842f04209", "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/sys/fs/cgroup/cpuset", 0xc000145e70, MS_RDONLY|MS_NOSUID|MS_NODEV|MS_NOEXEC|MS_BIND|MS_REC, NULL <unfinished ...>
[pid  2955] 12:56:30.349367 mount("/sys/fs/cgroup/cpuset/docker/1d7cad3c59e8c948f5d03bc414aa2a952eb3704069c0a4eed1fa72f842f04209", "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/sys/fs/cgroup/cpuset", 0xc000145e75, MS_RDONLY|MS_NOSUID|MS_NODEV|MS_NOEXEC|MS_REMOUNT|MS_BIND|MS_REC, NULL <unfinished ...>
[pid  2955] 12:56:30.362514 mkdirat(AT_FDCWD, "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/sys/fs/cgroup/memory", 0755 <unfinished ...>
[pid  2955] 12:56:30.362967 mount("/sys/fs/cgroup/memory/docker/1d7cad3c59e8c948f5d03bc414aa2a952eb3704069c0a4eed1fa72f842f04209", "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/sys/fs/cgroup/memory", 0xc000145e7a, MS_RDONLY|MS_NOSUID|MS_NODEV|MS_NOEXEC|MS_BIND|MS_REC, NULL <unfinished ...>
[pid  2955] 12:56:30.364469 mount("/sys/fs/cgroup/memory/docker/1d7cad3c59e8c948f5d03bc414aa2a952eb3704069c0a4eed1fa72f842f04209", "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/sys/fs/cgroup/memory", 0xc000145e80, MS_RDONLY|MS_NOSUID|MS_NODEV|MS_NOEXEC|MS_REMOUNT|MS_BIND|MS_REC, NULL <unfinished ...>
[pid  2955] 12:56:30.374722 mkdirat(AT_FDCWD, "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/sys/fs/cgroup/perf_event", 0755 <unfinished ...>
[pid  2955] 12:56:30.376315 mount("/sys/fs/cgroup/perf_event/docker/1d7cad3c59e8c948f5d03bc414aa2a952eb3704069c0a4eed1fa72f842f04209", "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/sys/fs/cgroup/perf_event", 0xc000145e85, MS_RDONLY|MS_NOSUID|MS_NODEV|MS_NOEXEC|MS_BIND|MS_REC, NULL <unfinished ...>
[pid  2955] 12:56:30.377242 mount("/sys/fs/cgroup/perf_event/docker/1d7cad3c59e8c948f5d03bc414aa2a952eb3704069c0a4eed1fa72f842f04209", "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/sys/fs/cgroup/perf_event", 0xc000145e8a, MS_RDONLY|MS_NOSUID|MS_NODEV|MS_NOEXEC|MS_REMOUNT|MS_BIND|MS_REC, NULL <unfinished ...>
[pid  2955] 12:56:30.388488 mkdirat(AT_FDCWD, "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/sys/fs/cgroup/devices", 0755 <unfinished ...>
[pid  2955] 12:56:30.391337 mount("/sys/fs/cgroup/devices/docker/1d7cad3c59e8c948f5d03bc414aa2a952eb3704069c0a4eed1fa72f842f04209", "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/sys/fs/cgroup/devices", 0xc000145e95, MS_RDONLY|MS_NOSUID|MS_NODEV|MS_NOEXEC|MS_REMOUNT|MS_BIND|MS_REC, NULL <unfinished ...>
[pid  2955] 12:56:30.406021 mkdirat(AT_FDCWD, "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/sys/fs/cgroup/rdma", 0755 <unfinished ...>
[pid  2955] 12:56:30.407807 mount("/sys/fs/cgroup/rdma", "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/sys/fs/cgroup/rdma", 0xc000145e9a, MS_RDONLY|MS_NOSUID|MS_NODEV|MS_NOEXEC|MS_BIND|MS_REC, NULL <unfinished ...>
[pid  2955] 12:56:30.408988 mount("/sys/fs/cgroup/rdma", "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/sys/fs/cgroup/rdma", 0xc000145ea0, MS_RDONLY|MS_NOSUID|MS_NODEV|MS_NOEXEC|MS_REMOUNT|MS_BIND|MS_REC, NULL <unfinished ...>
[pid  2955] 12:56:30.409970 symlinkat("cpu,cpuacct", AT_FDCWD, "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/sys/fs/cgroup/cpu" <unfinished ...>
[pid  2955] 12:56:30.410554 symlinkat("cpu,cpuacct", AT_FDCWD, "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/sys/fs/cgroup/cpuacct" <unfinished ...>
[pid  2955] 12:56:30.411225 symlinkat("net_cls,net_prio", AT_FDCWD, "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/sys/fs/cgroup/net_cls" <unfinished ...>
[pid  2955] 12:56:30.411903 symlinkat("net_cls,net_prio", AT_FDCWD, "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/sys/fs/cgroup/net_prio" <unfinished ...>
[pid  2955] 12:56:30.412577 mount("/sys/fs/cgroup", "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/sys/fs/cgroup", 0xc000145ea5, MS_RDONLY|MS_NOSUID|MS_NODEV|MS_NOEXEC|MS_REMOUNT|MS_BIND, NULL <unfinished ...>
[pid  2955] 12:56:30.415563 mkdirat(AT_FDCWD, "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/dev/mqueue", 0755 <unfinished ...>
[pid  2955] 12:56:30.415931 mount("mqueue", "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/dev/mqueue", "mqueue", MS_NOSUID|MS_NODEV|MS_NOEXEC, NULL <unfinished ...>
[pid  2955] 12:56:30.419712 mkdirat(AT_FDCWD, "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/dev/shm", 0755 <unfinished ...>
[pid  2955] 12:56:30.420920 mount("shm", "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/dev/shm", "tmpfs", MS_NOSUID|MS_NODEV|MS_NOEXEC, "mode=1777,size=67108864" <unfinished ...>
[pid  2955] 12:56:30.427351 mount("/var/lib/docker/containers/1d7cad3c59e8c948f5d03bc414aa2a952eb3704069c0a4eed1fa72f842f04209/resolv.conf", "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/etc/resolv.conf", 0xc000145efa, MS_BIND|MS_REC, NULL <unfinished ...>
[pid  2955] 12:56:30.427863 mount("", "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/etc/resolv.conf", 0xc000145f00, MS_REC|MS_PRIVATE, NULL <unfinished ...>
[pid  2955] 12:56:30.432670 mount("/var/lib/docker/containers/1d7cad3c59e8c948f5d03bc414aa2a952eb3704069c0a4eed1fa72f842f04209/hostname", "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/etc/hostname", 0xc000145f20, MS_BIND|MS_REC, NULL <unfinished ...>
[pid  2955] 12:56:30.433296 mount("", "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/etc/hostname", 0xc000145f26, MS_REC|MS_PRIVATE, NULL <unfinished ...>
[pid  2955] 12:56:30.436280 mount("/var/lib/docker/containers/1d7cad3c59e8c948f5d03bc414aa2a952eb3704069c0a4eed1fa72f842f04209/hosts", "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/etc/hosts", 0xc000145f27, MS_BIND|MS_REC, NULL <unfinished ...>
[pid  2955] 12:56:30.436932 mount("", "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/etc/hosts", 0xc000145f2d, MS_REC|MS_PRIVATE, NULL <unfinished ...>
[pid  2955] 12:56:30.443513 mknodat(AT_FDCWD, "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/dev/null", S_IFCHR|0666, makedev(1, 3) <unfinished ...>
[pid  2955] 12:56:30.444173 fchownat(AT_FDCWD, "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/dev/null", 0, 0, 0 <unfinished ...>
[pid  2955] 12:56:30.444753 mknodat(AT_FDCWD, "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/dev/random", S_IFCHR|0666, makedev(1, 8) <unfinished ...>
[pid  2955] 12:56:30.445351 fchownat(AT_FDCWD, "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/dev/random", 0, 0, 0 <unfinished ...>
[pid  2955] 12:56:30.455885 mknodat(AT_FDCWD, "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/dev/urandom", S_IFCHR|0666, makedev(1, 9) <unfinished ...>
[pid  2955] 12:56:30.458318 fchownat(AT_FDCWD, "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/dev/urandom", 0, 0, 0 <unfinished ...>
[pid  2955] 12:56:30.462010 unlinkat(AT_FDCWD, "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/dev/ptmx", 0 <unfinished ...>
[pid  2955] 12:56:30.462768 unlinkat(AT_FDCWD, "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/dev/ptmx", AT_REMOVEDIR <unfinished ...>
[pid  2955] 12:56:30.463583 symlinkat("pts/ptmx", AT_FDCWD, "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/dev/ptmx" <unfinished ...>
[pid  2955] 12:56:30.467257 symlinkat("/proc/self/fd", AT_FDCWD, "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/dev/fd" <unfinished ...>
[pid  2955] 12:56:30.468121 symlinkat("/proc/self/fd/0", AT_FDCWD, "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/dev/stdin" <unfinished ...>
[pid  2955] 12:56:30.468779 symlinkat("/proc/self/fd/1", AT_FDCWD, "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/dev/stdout" <unfinished ...>
[pid  2955] 12:56:30.469788 symlinkat("/proc/self/fd/2", AT_FDCWD, "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/dev/stderr" <unfinished ...>
[pid  2955] 12:56:30.470489 symlinkat("/proc/kcore", AT_FDCWD, "/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged/dev/core" <unfinished ...>
```

```
[pid  2967] 12:56:30.490649 execve("/proc/1159/exe", ["libnetwork-setkey", "-exec-root=/var/run/docker", "1d7cad3c59e8c948f5d03bc414aa2a952eb3704069c0a4eed1fa72f842f04209", "0f34ad9f49b4"], 0xc000196dc0 /* 6 vars */ <unfinished ...>
```
```
[pid  2937] 12:56:30.492264 write(10, "{\"ociVersion\":\"1.0.1-dev\",\"id\":\"1d7cad3c59e8c948f5d03bc414aa2a952eb3704069c0a4eed1fa72f842f04209\",\"status\":\"creating\",\"pid\":2955,\"bundle\":\"/run/containerd/io.containerd.runtime.v1.linux/moby/1d7cad3c59e8c948f5d03bc414aa2a952eb3704069c0a4eed1fa72f842f04209\"}", 257 <unfinished ...>
```


### pivot_root(2)を使ったルートファイルシステムの入れ替え

```
[pid  2955] 12:56:31.791254 chdir("/var/lib/docker/overlay2/4d0746bf25d0395a308ede0615a5f3093e7cead33963ccadc452e3477844a38d/merged" <unfinished ...>
[pid  2955] 12:56:31.793748 pivot_root(".", ".") = 0 <0.000053>
[pid  2955] 12:56:31.793833 fchdir(7)   = 0 <0.000008>
[pid  2955] 12:56:31.793876 mount("", ".", 0xc000182122, MS_REC|MS_SLAVE, NULL <unfinished ...>
[pid  2955] 12:56:31.819142 mount("/dev/pts/0", "/dev/console", 0xc000182178, MS_BIND, NULL <unfinished ...>
[pid  2955] 12:56:31.842281 mount("tmpfs", "/proc/acpi", "tmpfs", MS_RDONLY, NULL <unfinished ...>
[pid  2955] 12:56:31.842986 mount("/dev/null", "/proc/kcore", 0xc00018233c, MS_BIND, NULL <unfinished ...>
[pid  2955] 12:56:31.844048 mount("/dev/null", "/proc/keys", 0xc00018234a, MS_BIND, NULL <unfinished ...>
[pid  2955] 12:56:31.844237 mount("/dev/null", "/proc/latency_stats", 0xc00018236a, MS_BIND, NULL <unfinished ...>
[pid  2955] 12:56:31.844696 mount("/dev/null", "/proc/timer_list", 0xc00018237a, MS_BIND, NULL <unfinished ...>
[pid  2955] 12:56:31.845037 mount("/dev/null", "/proc/timer_stats", 0xc00018238a, MS_BIND, NULL) = -1 ENOENT (No such file or directory) <0.000012>
[pid  2955] 12:56:31.845089 mount("/dev/null", "/proc/sched_debug", 0xc00018239a, MS_BIND, NULL) = 0 <0.000015>
[pid  2955] 12:56:31.845146 mount("/dev/null", "/proc/scsi", 0xc0001823aa, MS_BIND, NULL <unfinished ...>
[pid  2955] 12:56:31.858985 mount("/dev/null", "/sys/firmware", 0xc0001823fe, MS_BIND, NULL <unfinished ...>
[pid  2955] 12:56:31.874235 mount("tmpfs", "/sys/firmware", "tmpfs", MS_RDONLY, NULL <unfinished ...>
[pid  2955] 12:56:31.876789 gettid()    = 1 <0.000006>
[pid  2955] 12:56:31.920695 seccomp(SECCOMP_SET_MODE_FILTER, SECCOMP_FILTER_FLAG_TSYNC, {len=1021, filter=0x55e0905448b0}) = 0 <0.000775>
[pid  2955] 12:56:31.932182 chdir("/" <unfinished ...>
```

### capabilityの設定

```
[pid  2955] 12:56:31.953394 capget({version=_LINUX_CAPABILITY_VERSION_3, pid=0}, {effective=1<<CAP_CHOWN|1<<CAP_DAC_OVERRIDE|1<<CAP_DAC_READ_SEARCH|1<<CAP_FOWNER|1<<CAP_FSETID|1<<CAP_KILL|1<<CAP_SETGID|1<<CAP_SETUID|1<<CAP_SETPCAP|1<<CAP_LINUX_IMMUTABLE|1<<CAP_NET_BIND_SERVICE|1<<CAP_NET_BROADCAST|1<<CAP_NET_ADMIN|1<<CAP_NET_RAW|1<<CAP_IPC_LOCK|1<<CAP_IPC_OWNER|1<<CAP_SYS_MODULE|1<<CAP_SYS_RAWIO|1<<CAP_SYS_CHROOT|1<<CAP_SYS_PTRACE|1<<CAP_SYS_PACCT|1<<CAP_SYS_ADMIN|1<<CAP_SYS_BOOT|1<<CAP_SYS_NICE|1<<CAP_SYS_RESOURCE|1<<CAP_SYS_TIME|1<<CAP_SYS_TTY_CONFIG|1<<CAP_MKNOD|1<<CAP_LEASE|1<<CAP_AUDIT_WRITE|1<<CAP_AUDIT_CONTROL|1<<CAP_SETFCAP|1<<CAP_MAC_OVERRIDE|1<<CAP_MAC_ADMIN|1<<CAP_SYSLOG|1<<CAP_WAKE_ALARM|1<<CAP_BLOCK_SUSPEND|1<<CAP_AUDIT_READ, permitted=1<<CAP_CHOWN|1<<CAP_DAC_OVERRIDE|1<<CAP_DAC_READ_SEARCH|1<<CAP_FOWNER|1<<CAP_FSETID|1<<CAP_KILL|1<<CAP_SETGID|1<<CAP_SETUID|1<<CAP_SETPCAP|1<<CAP_LINUX_IMMUTABLE|1<<CAP_NET_BIND_SERVICE|1<<CAP_NET_BROADCAST|1<<CAP_NET_ADMIN|1<<CAP_NET_RAW|1<<CAP_IPC_LOCK|1<<CAP_IPC_OWNER|1<<CAP_SYS_MODULE|1<<CAP_SYS_RAWIO|1<<CAP_SYS_CHROOT|1<<CAP_SYS_PTRACE|1<<CAP_SYS_PACCT|1<<CAP_SYS_ADMIN|1<<CAP_SYS_BOOT|1<<CAP_SYS_NICE|1<<CAP_SYS_RESOURCE|1<<CAP_SYS_TIME|1<<CAP_SYS_TTY_CONFIG|1<<CAP_MKNOD|1<<CAP_LEASE|1<<CAP_AUDIT_WRITE|1<<CAP_AUDIT_CONTROL|1<<CAP_SETFCAP|1<<CAP_MAC_OVERRIDE|1<<CAP_MAC_ADMIN|1<<CAP_SYSLOG|1<<CAP_WAKE_ALARM|1<<CAP_BLOCK_SUSPEND|1<<CAP_AUDIT_READ, inheritable=0}) = 0 <0.000008>
[pid  2955] 12:56:31.955838 capset({version=_LINUX_CAPABILITY_VERSION_3, pid=0}, {effective=1<<CAP_CHOWN|1<<CAP_DAC_OVERRIDE|1<<CAP_FOWNER|1<<CAP_FSETID|1<<CAP_KILL|1<<CAP_SETGID|1<<CAP_SETUID|1<<CAP_SETPCAP|1<<CAP_NET_BIND_SERVICE|1<<CAP_NET_RAW|1<<CAP_SYS_CHROOT|1<<CAP_MKNOD|1<<CAP_AUDIT_WRITE|1<<CAP_SETFCAP, permitted=1<<CAP_CHOWN|1<<CAP_DAC_OVERRIDE|1<<CAP_FOWNER|1<<CAP_FSETID|1<<CAP_KILL|1<<CAP_SETGID|1<<CAP_SETUID|1<<CAP_SETPCAP|1<<CAP_NET_BIND_SERVICE|1<<CAP_NET_RAW|1<<CAP_SYS_CHROOT|1<<CAP_MKNOD|1<<CAP_AUDIT_WRITE|1<<CAP_SETFCAP, inheritable=1<<CAP_CHOWN|1<<CAP_DAC_OVERRIDE|1<<CAP_FOWNER|1<<CAP_FSETID|1<<CAP_KILL|1<<CAP_SETGID|1<<CAP_SETUID|1<<CAP_SETPCAP|1<<CAP_NET_BIND_SERVICE|1<<CAP_NET_RAW|1<<CAP_SYS_CHROOT|1<<CAP_MKNOD|1<<CAP_AUDIT_WRITE|1<<CAP_SETFCAP} <unfinished ...>
```

```
[pid  2993] 12:56:32.041556 execve("/usr/bin/runc", ["runc", "--root", "/var/run/docker/runtime-runc/moby", "--log", "/run/containerd/io.containerd.runtime.v1.linux/moby/1d7cad3c59e8c948f5d03bc414aa2a952eb3704069c0a4eed1fa72f842f04209/log.json", "--log-format", "json", "start", "1d7cad3c59e8c948f5d03bc414aa2a952eb3704069c0a4eed1fa72f842f04209"], 0xc0000500c0 /* 6 vars */) = 0 <0.000540>
```


### 実行

```
[pid  2955] 12:56:32.162811 execve("/bin/bash", ["bash"], 0xc0001617d0 /* 4 vars */ <unfinished ...>
```

ここでstraceが固まった。。。

フルバージョン

https://gist.github.com/SpringMT/7719f14dfa0aacbc9b05c253f656dd7c

参考資料


