# PID Namespace(PID 名前空間)
PID名前空間は文字通りプロセスIDを名前空間ごとに持てます。

通常はPIDは一意に決まる数字が振られますが，異なる名前空間にいるプロセスは同じPIDを持つことができます。

unshareコマンドでPID名前空間を分離できます。

unshareコマンドには、指定したプロセスが新しいPID名前空間で pid=1 なプロセスになるよう、unshare(2) で新しいpid名前空間を作ったあと、一回 fork(2) してから execvp(2) する、 --fork オプションが用意されています。

```
vagrant@ubuntu-bionic:~$ echo $$
9575
vagrant@ubuntu-bionic:~$ ls -l /proc/$$/ns/
total 0
lrwxrwxrwx 1 vagrant vagrant 0 Nov 16 03:03 cgroup -> 'cgroup:[4026531835]'
lrwxrwxrwx 1 vagrant vagrant 0 Nov 16 03:03 ipc -> 'ipc:[4026531839]'
lrwxrwxrwx 1 vagrant vagrant 0 Nov 16 03:03 mnt -> 'mnt:[4026531840]'
lrwxrwxrwx 1 vagrant vagrant 0 Nov 16 03:03 net -> 'net:[4026531993]'
lrwxrwxrwx 1 vagrant vagrant 0 Nov 16 03:03 pid -> 'pid:[4026531836]'
lrwxrwxrwx 1 vagrant vagrant 0 Nov 16 03:03 pid_for_children -> 'pid:[4026531836]'
lrwxrwxrwx 1 vagrant vagrant 0 Nov 16 03:03 user -> 'user:[4026531837]'
lrwxrwxrwx 1 vagrant vagrant 0 Nov 16 03:03 uts -> 'uts:[4026531838]'

# unshareコマンド実行
vagrant@ubuntu-bionic:~$ sudo unshare --pid --fork /bin/bash
root@ubuntu-bionic:~# echo $$
1
root@ubuntu-bionic:~# ls -l /proc/$$/ns/
total 0
lrwxrwxrwx 1 root root 0 Nov 16 03:03 cgroup -> 'cgroup:[4026531835]'
lrwxrwxrwx 1 root root 0 Nov 16 03:03 ipc -> 'ipc:[4026531839]'
lrwxrwxrwx 1 root root 0 Nov 16 03:03 mnt -> 'mnt:[4026531840]'
lrwxrwxrwx 1 root root 0 Nov 16 03:03 net -> 'net:[4026531993]'
lrwxrwxrwx 1 root root 0 Nov 16 03:03 pid -> 'pid:[4026531836]' <= 変わってない？？
lrwxrwxrwx 1 root root 0 Nov 16 03:03 pid_for_children -> 'pid:[4026531836]'
lrwxrwxrwx 1 root root 0 Nov 16 03:03 user -> 'user:[4026531837]'
lrwxrwxrwx 1 root root 0 Nov 16 03:03 uts -> 'uts:[4026531838]'

# 全部見えている
root@ubuntu-bionic:~# ps -Al
F S   UID   PID  PPID  C PRI  NI ADDR SZ WCHAN  TTY          TIME CMD
4 S     0     1     0  0  80   0 - 19510 ep_pol ?        00:00:02 systemd
1 S     0     2     0  0  80   0 -     0 kthrea ?        00:00:00 kthreadd
1 I     0     4     2  0  60 -20 -     0 worker ?        00:00:00 kworker/0:0H
1 R     0     5     2  0  80   0 -     0 -      ?        00:00:00 kworker/u4:0
1 I     0     6     2  0  60 -20 -     0 rescue ?        00:00:00 mm_percpu_wq
1 S     0     7     2  0  80   0 -     0 smpboo ?        00:00:00 ksoftirqd/0
1 I     0     8     2  0  80   0 -     0 rcu_gp ?        00:00:00 rcu_sched
1 I     0     9     2  0  80   0 -     0 rcu_gp ?        00:00:00 rcu_bh
....
5 S  1000  9494  9493  0  80   0 - 28005 sigtim ?        00:00:00 (sd-pam)
5 R  1000  9574  9491  0  80   0 - 27088 -      ?        00:00:00 sshd
0 S  1000  9575  9574  0  80   0 -  5805 wait   pts/0    00:00:00 bash
1 R     0  9589     2  0  80   0 -     0 -      ?        00:00:00 kworker/u4:1
1 I     0  9635     2  0  80   0 -     0 worker ?        00:00:00 kworker/0:1
4 S     0  9651  9575  0  80   0 - 15994 poll_s pts/0    00:00:00 sudo
4 S     0  9652  9651  0  80   0 -  1980 wait   pts/0    00:00:00 unshare
0 S     0  9653  9652  0  80   0 -  5754 wait   pts/0    00:00:00 bash
0 R     0  9665  9653  0  80   0 -  7337 -      pts/0    00:00:00 ps
```

PID名前空間を分けたはずなのにps経由で元のPID名前空間の情報が見えてしまっています。

/proc ファイルシステムが元の名前空間と同じことに起因しています。

psコマンドは /proc を見ています。

なので、/procをマウント名前空間を使って分離します。

unshareコマンドには、 新しいマウント名前空間を作成しprocファイルシステムをmountpoint (デフォルトは /proc) に再マウントしてくれる `--mount-proc` オプションがあります。

```
       --mount-proc[=mountpoint]

              Just before running the program, mount the proc filesystem at mountpoint (default is /proc).  This is useful when creating a  new  PID  names‐

              pace.   It also implies creating a new mount namespace since the /proc mount would otherwise mess up existing programs on the system.  The new

              proc filesystem is explicitly mounted as private (with MS_PRIVATE|MS_REC).
```
https://ysatoautonomous.github.io/jm-draft/man2html/util-linux/util-linux-2.34/draft/man1/unshare.1.html

まさにこれなので、--mount-procを使って再度試してみます。

```
vagrant@ubuntu-bionic:~$ echo $$
9575
vagrant@ubuntu-bionic:~$ ls -l /proc/$$/ns/
total 0
lrwxrwxrwx 1 vagrant vagrant 0 Nov 16 03:03 cgroup -> 'cgroup:[4026531835]'
lrwxrwxrwx 1 vagrant vagrant 0 Nov 16 03:03 ipc -> 'ipc:[4026531839]'
lrwxrwxrwx 1 vagrant vagrant 0 Nov 16 03:03 mnt -> 'mnt:[4026531840]'
lrwxrwxrwx 1 vagrant vagrant 0 Nov 16 03:03 net -> 'net:[4026531993]'
lrwxrwxrwx 1 vagrant vagrant 0 Nov 16 03:03 pid -> 'pid:[4026531836]'
lrwxrwxrwx 1 vagrant vagrant 0 Nov 16 03:03 pid_for_children -> 'pid:[4026531836]'
lrwxrwxrwx 1 vagrant vagrant 0 Nov 16 03:03 user -> 'user:[4026531837]'
lrwxrwxrwx 1 vagrant vagrant 0 Nov 16 03:03 uts -> 'uts:[4026531838]'
vagrant@ubuntu-bionic:~$ sudo unshare --pid --fork --mount-proc /bin/bash
root@ubuntu-bionic:~# echo $$
1
root@ubuntu-bionic:~# ls -l /proc/$$/ns/
total 0
lrwxrwxrwx 1 root root 0 Nov 16 04:03 cgroup -> 'cgroup:[4026531835]'
lrwxrwxrwx 1 root root 0 Nov 16 04:03 ipc -> 'ipc:[4026531839]'
lrwxrwxrwx 1 root root 0 Nov 16 04:03 mnt -> 'mnt:[4026532175]'
lrwxrwxrwx 1 root root 0 Nov 16 04:03 net -> 'net:[4026531993]'
lrwxrwxrwx 1 root root 0 Nov 16 04:03 pid -> 'pid:[4026532176]'
lrwxrwxrwx 1 root root 0 Nov 16 04:03 pid_for_children -> 'pid:[4026532176]'
lrwxrwxrwx 1 root root 0 Nov 16 04:03 user -> 'user:[4026531837]'
lrwxrwxrwx 1 root root 0 Nov 16 04:03 uts -> 'uts:[4026531838]'
root@ubuntu-bionic:~# ps -Al
F S   UID   PID  PPID  C PRI  NI ADDR SZ WCHAN  TTY          TIME CMD
4 S     0     1     0  0  80   0 -  5754 wait   pts/0    00:00:00 bash
0 R     0    12     1  0  80   0 -  7337 -      pts/0    00:00:00 ps
```

新しいPID名前空間ではpsコマンド経由でもそのPID名前空間のプロセスしか見えないようになっています。

また、PID名前空間が異なると同じPIDが存在できることもわかるかと思います。(PID 1が親と子にそれぞれ存在する)

では親の(unshareコマンドを叩いた)PID名前空間ではどのようにみえているでしょうか。

```
# 子でsleepコマンドを打つ
sleep 60

# 親
root      9491  0.0  0.7 107992  7252 ?        Ss   02:59   0:00  \_ sshd: vagrant [priv]
vagrant   9574  0.0  0.4 108352  5028 ?        S    02:59   0:00  |   \_ sshd: vagrant@pts/0
vagrant   9575  0.0  0.5  23220  5108 pts/0    Ss   02:59   0:00  |       \_ -bash
root      9700  0.0  0.4  63976  4212 pts/0    S    04:03   0:00  |           \_ sudo unshare --pid --fork --mount-proc /bin/bash
root      9701  0.0  0.0   7920   772 pts/0    S    04:03   0:00  |               \_ unshare --pid --fork --mount-proc /bin/bash
root      9702  0.0  0.4  23016  4928 pts/0    S    04:03   0:00  |                   \_ /bin/bash
root      9827  0.0  0.0   7932   788 pts/0    S+   04:10   0:00  |                       \_ sleep 60 <- sleepコマンドが見えている
```

PID名前空間では、親のPID名前空間からはこのPID名前空間のプロセスが見えます。

これはPID名前空間の仕様となっています。

## 参考資料
* https://tech.retrieva.jp/entry/2019/04/16/155828#pid-namespace
* https://gihyo.jp/admin/serial/01/linux_containers/0002?page=3
