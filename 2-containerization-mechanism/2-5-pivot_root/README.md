# pivot_root
pivot_rootはプロセスのrootファイルシステムを入れ替えます。

chrootはプロセスのルートディレクトリが指すパスを変更します。chrootは呼び出し前のrootファイルシステムの参照を持っています。

2020/11/24時点ではcontainer breakoutを防ぐためにDockerは基本的にpivot_rootを使ってルートファイルシステムを隔離しています。

https://github.com/opencontainers/runc/blob/e8498d3ad580b31d8755253e60570466be5a66ba/libcontainer/rootfs_linux.go#L772-L827

## 現在のプロセスのrootディレクトリの確認

```
vagrant@ubuntu-bionic:~$ ls -l /proc/$$/root
lrwxrwxrwx 1 vagrant vagrant 0 Nov 23 17:04 /proc/2654/root -> /
```

## pivot_rootコマンドを使ったrootファイルシステムの入れ替え

https://linuxjm.osdn.jp/html/util-linux/man8/pivot_root.8.html

pivot_rootコマンド(pivot_root(8))はカレントプロセスの root ファイルシステムを put_old ディレクトリに移動し、 new_root を新しい root ファイルシステムにします。

つまり、現在のプロセスのルートファイルシステムを別の場所にマウントしなおします。

pivot_root(8) は pivot_root(2) を呼び出しているだけです。

pivot_root new_root put_old  
pivot_root(2)はいくつか制限があります。

```
new_root および put_old には以下の制限がある:
- ディレクトリでなければならない。
- new_root と put_old は現在の root と同じファイルシステムにあってはならない。
- put_old は new_root 以下になければならない。すなわち put_old を差す文字列に 1 個以上の ../ を付けることによって new_root と同じディレクトリが得られなければならない。
- 他のファイルシステムが put_old にマウントされていてはならない。
```

https://linuxjm.osdn.jp/html/LDP_man-pages/man2/pivot_root.2.html

これらの条件を満たすことと、他のプロセスに影響を与えないようにするために下記を行います。

* マウント名前空間を使って現在のマウントをPrivateマウントにする
* mountのbindでディレクトリをマウントすることで、別ファイルシステムとする

## 試してみる

まずは、新しいファイルシステムを作るための準備です

```
vagrant@ubuntu-bionic:~$ mkdir pivot_root_test
vagrant@ubuntu-bionic:~$ cd pivot_root_test/
vagrant@ubuntu-bionic:~/pivot_root_test$ ls -l /proc/$$/root
lrwxrwxrwx 1 vagrant vagrant 0 Nov 23 17:04 /proc/2654/root -> /

vagrant@ubuntu-bionic:~/pivot_root_test$ cat /proc/$$/mountinfo
21 26 0:20 / /sys rw,nosuid,nodev,noexec,relatime shared:7 - sysfs sysfs rw
22 26 0:4 / /proc rw,nosuid,nodev,noexec,relatime shared:13 - proc proc rw
23 26 0:6 / /dev rw,nosuid,relatime shared:2 - devtmpfs udev rw,size=491532k,nr_inodes=122883,mode=755
24 23 0:21 / /dev/pts rw,nosuid,noexec,relatime shared:3 - devpts devpts rw,gid=5,mode=620,ptmxmode=000
25 26 0:22 / /run rw,nosuid,noexec,relatime shared:5 - tmpfs tmpfs rw,size=100860k,mode=755
26 0 8:1 / / rw,relatime shared:1 - ext4 /dev/sda1 rw,data=ordered
27 21 0:7 / /sys/kernel/security rw,nosuid,nodev,noexec,relatime shared:8 - securityfs securityfs rw
28 23 0:23 / /dev/shm rw,nosuid,nodev shared:4 - tmpfs tmpfs rw
29 25 0:24 / /run/lock rw,nosuid,nodev,noexec,relatime shared:6 - tmpfs tmpfs rw,size=5120k
30 21 0:25 / /sys/fs/cgroup ro,nosuid,nodev,noexec shared:9 - tmpfs tmpfs ro,mode=755
31 30 0:26 / /sys/fs/cgroup/unified rw,nosuid,nodev,noexec,relatime shared:10 - cgroup2 cgroup rw
32 30 0:27 / /sys/fs/cgroup/systemd rw,nosuid,nodev,noexec,relatime shared:11 - cgroup cgroup rw,xattr,name=systemd
33 21 0:28 / /sys/fs/pstore rw,nosuid,nodev,noexec,relatime shared:12 - pstore pstore rw
34 30 0:29 / /sys/fs/cgroup/cpu,cpuacct rw,nosuid,nodev,noexec,relatime shared:14 - cgroup cgroup rw,cpu,cpuacct
35 30 0:30 / /sys/fs/cgroup/freezer rw,nosuid,nodev,noexec,relatime shared:15 - cgroup cgroup rw,freezer
36 30 0:31 / /sys/fs/cgroup/blkio rw,nosuid,nodev,noexec,relatime shared:16 - cgroup cgroup rw,blkio
37 30 0:32 / /sys/fs/cgroup/pids rw,nosuid,nodev,noexec,relatime shared:17 - cgroup cgroup rw,pids
38 30 0:33 / /sys/fs/cgroup/net_cls,net_prio rw,nosuid,nodev,noexec,relatime shared:18 - cgroup cgroup rw,net_cls,net_prio
39 30 0:34 / /sys/fs/cgroup/hugetlb rw,nosuid,nodev,noexec,relatime shared:19 - cgroup cgroup rw,hugetlb
40 30 0:35 / /sys/fs/cgroup/cpuset rw,nosuid,nodev,noexec,relatime shared:20 - cgroup cgroup rw,cpuset
41 30 0:36 / /sys/fs/cgroup/memory rw,nosuid,nodev,noexec,relatime shared:21 - cgroup cgroup rw,memory
42 30 0:37 / /sys/fs/cgroup/perf_event rw,nosuid,nodev,noexec,relatime shared:22 - cgroup cgroup rw,perf_event
43 30 0:38 / /sys/fs/cgroup/devices rw,nosuid,nodev,noexec,relatime shared:23 - cgroup cgroup rw,devices
44 30 0:39 / /sys/fs/cgroup/rdma rw,nosuid,nodev,noexec,relatime shared:24 - cgroup cgroup rw,rdma
45 23 0:40 / /dev/hugepages rw,relatime shared:25 - hugetlbfs hugetlbfs rw,pagesize=2M
46 23 0:18 / /dev/mqueue rw,relatime shared:26 - mqueue mqueue rw
47 21 0:8 / /sys/kernel/debug rw,relatime shared:27 - debugfs debugfs rw
48 22 0:41 / /proc/sys/fs/binfmt_misc rw,relatime shared:28 - autofs systemd-1 rw,fd=43,pgrp=1,timeout=0,minproto=5,maxproto=5,direct,pipe_ino=11765
49 21 0:42 / /sys/fs/fuse/connections rw,relatime shared:29 - fusectl fusectl rw
50 21 0:19 / /sys/kernel/config rw,relatime shared:30 - configfs configfs rw
82 26 0:43 / /vagrant rw,nodev,relatime shared:31 - vboxsf /vagrant rw
232 26 0:46 / /var/lib/lxcfs rw,nosuid,nodev,relatime shared:115 - fuse.lxcfs lxcfs rw,user_id=0,group_id=0,allow_other
244 82 0:48 / /vagrant rw,nodev,relatime shared:121 - vboxsf vagrant rw
238 25 0:47 / /run/user/1000 rw,nosuid,nodev,relatime shared:118 - tmpfs tmpfs rw,size=100860k,mode=700,uid=1000,gid=1000

vagrant@ubuntu-bionic:~/pivot_root_test$ NEW_ROOT=$(pwd)
vagrant@ubuntu-bionic:~/pivot_root_test$ echo $NEW_ROOT
/home/vagrant/pivot_root_test
vagrant@ubuntu-bionic:~/pivot_root_test$ cp -a /bin /lib /lib64 ./
vagrant@ubuntu-bionic:~/pivot_root_test$ cp -a /usr ./ # ps を打つのに必要ですが容量食うので不要の場合はしなくてOK
vagrant@ubuntu-bionic:~/pivot_root_test$ mkdir $NEW_ROOT/{.put_old,proc}
```

次に、pivot_rootコマンドを実行します。

コマンドの説明は下記のとおりです。
```
# sudo unshare -mpf /bin/sh -c " \      # -m mount -p pid -f fork r ルートファイルシステムをPrivateマウントする
  mount --bind $NEW_ROOT $NEW_ROOT && \ # mount --bind してNEW_ROOTを別のファイルシステムにします
  mount -t proc proc $NEW_ROOT/proc && \ # put_oldをアンマウントしてd、元のrootファイルシステムを切り離すので、procfsをマウントしてマウントポイントを参照できるようにします
  pivot_root $NEW_ROOT $NEW_ROOT/.put_old && \ # pivot_rootでrootファイルシステムを入れ替えます もとのrootファイルシステムは.put_oldに入ります。
  umount -l /.put_old && \ # 元のrootファイルシステムをアンマウントします
  cd / && \ # プロセスのカレントディレクトリを変更します
  exec /bin/sh # プロセスイメージを入れ替えます
"
```

```
vagrant@ubuntu-bionic:~/pivot_root_test$ sudo unshare -mpf /bin/sh -c " \
  mount --bind $NEW_ROOT $NEW_ROOT && \
  mount -t proc proc $NEW_ROOT/proc && \
  pivot_root $NEW_ROOT $NEW_ROOT/.put_old && \
  umount -l /.put_old && \
  cd / && \
  exec /bin/sh
"
# ls -l /proc/$$/root
lrwxrwxrwx 1 0 0 0 Nov 23 19:18 /proc/1/root -> /
# cat /proc/$$/mountinfo
302 250 8:1 /home/vagrant/pivot_root_test / rw,relatime - ext4 /dev/sda1 rw,data=ordered
303 302 0:49 / /proc rw,relatime - proc proc rw

# ps
  PID TTY          TIME CMD
    1 ?        00:00:00 sh
    6 ?        00:00:00 ps
# ps auxwwf
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
0            1  0.0  0.0   4636   272 ?        S    19:17   0:00 /bin/sh
0            7  0.0  0.3  37804  3268 ?        R+   19:17   0:00 ps auxwwf
# ls
bin  lib  lib64  proc  usr

# ls -al
total 28
drwxrwxr-x   8 1000 1000 4096 Nov 23 19:17 .
drwxrwxr-x   8 1000 1000 4096 Nov 23 19:17 ..
drwxrwxr-x   2 1000 1000 4096 Nov 23 18:41 .put_old
drwxr-xr-x   2 1000 1000 4096 Nov 12 15:54 bin
drwxr-xr-x  21 1000 1000 4096 Nov 16 02:04 lib
drwxr-xr-x   3 1000 1000 4096 Nov 16 02:05 lib64
dr-xr-xr-x 119    0    0    0 Nov 23 19:17 proc
drwxr-xr-x  11 1000 1000 4096 Nov 16 02:04 usr

# cat /etc/hosts
cat: /etc/hosts: No such file or directory
# ls /etc
ls: cannot access '/etc': No such file or directory
# touch test.txt
# ls
bin  lib  lib64  proc  test.txt  usr
# 
```

## chrootとの比較
chroot はプロセスが CAP_SYS_CHROOT Capability を持っている場合に、脱獄(chroot 環境から元の環境に移動できる)が可能です。

https://eh-career.com/engineerhub/entry/2019/02/05/103000#chroot%E3%81%A8pivot_root

https://container-security.dev/namespace/chroot-and-pivot_root.html

pivot_rootではそれができないので、基本的にpivot_rootを使うのがよいでしょう。

## 参考資料
* https://www.ibm.com/support/knowledgecenter/ja/ssw_aix_71/devicemanagement/fs_root.html
* chrootを使った問題の一つ
  * https://github.com/opencontainers/runc/pull/1962
* pivot_root できる条件
  * https://tenforward.hatenablog.com/entry/2017/06/28/021019
* https://container-security.dev/namespace/chroot-and-pivot_root.html
* https://blog.framinal.life/entry/2020/04/09/183208
