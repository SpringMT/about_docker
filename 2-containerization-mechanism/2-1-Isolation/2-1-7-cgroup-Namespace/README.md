# cgroup Namespace(cgroup名前空間)
cgroup名前空間では、プロセスのcgroupを隔離します。

cgroupfsが名前空間ごとに見えるようになります。

3-2-1. cgroup

cgroup名前空間によって分離すると、コンテナ内では自身のcgroupがルートになります。

試してみます。

```
root@ubuntu-bionic:~# ls -l /proc/$$/ns/
total 0
lrwxrwxrwx 1 root root 0 Dec  7 06:16 cgroup -> 'cgroup:[4026531835]'
lrwxrwxrwx 1 root root 0 Dec  7 06:16 ipc -> 'ipc:[4026531839]'
lrwxrwxrwx 1 root root 0 Dec  7 06:16 mnt -> 'mnt:[4026531840]'
lrwxrwxrwx 1 root root 0 Dec  7 06:16 net -> 'net:[4026531993]'
lrwxrwxrwx 1 root root 0 Dec  7 06:16 pid -> 'pid:[4026531836]'
lrwxrwxrwx 1 root root 0 Dec  7 06:16 pid_for_children -> 'pid:[4026531836]'
lrwxrwxrwx 1 root root 0 Dec  7 06:16 user -> 'user:[4026531837]'
lrwxrwxrwx 1 root root 0 Dec  7 06:16 uts -> 'uts:[4026531838]'

# cgroupへの所属
root@ubuntu-bionic:~# mkdir /sys/fs/cgroup/memory/test01 
root@ubuntu-bionic:~# echo $$
13606
root@ubuntu-bionic:~# echo 13606 > /sys/fs/cgroup/memory/test01/tasks 
root@ubuntu-bionic:~# cat /proc/$$/cgroup | grep memory
9:memory:/test01 # <- 所属するcgroupは"/test01"となっている

root@ubuntu-bionic:~# unshare --cgroup --mount /bin/bash # cgroupとマウント名前空間を作成
root@ubuntu-bionic:~# ls -l /proc/$$/ns/
total 0
lrwxrwxrwx 1 root root 0 Dec  7 06:27 cgroup -> 'cgroup:[4026532180]' # <-別の名前空間
lrwxrwxrwx 1 root root 0 Dec  7 06:27 ipc -> 'ipc:[4026531839]'
lrwxrwxrwx 1 root root 0 Dec  7 06:27 mnt -> 'mnt:[4026532175]'
lrwxrwxrwx 1 root root 0 Dec  7 06:27 net -> 'net:[4026531993]'
lrwxrwxrwx 1 root root 0 Dec  7 06:27 pid -> 'pid:[4026531836]'
lrwxrwxrwx 1 root root 0 Dec  7 06:27 pid_for_children -> 'pid:[4026531836]'
lrwxrwxrwx 1 root root 0 Dec  7 06:27 user -> 'user:[4026531837]'
lrwxrwxrwx 1 root root 0 Dec  7 06:27 uts -> 'uts:[4026531838]'

root@ubuntu-bionic:~# echo $$
13760
root@ubuntu-bionic:~# cat /sys/fs/cgroup/memory/test01/tasks
13606
13760
13771
```

作成した名前空間を確認します。

```
# 作成した名前空間内で確認
root@ubuntu-bionic:~# cat /proc/13606/cgroup | grep memory
9:memory:/  # 名前空間内で見るとルートに属している
root@ubuntu-bionic:~# cat /proc/13760/cgroup | grep memory
9:memory:/
# cgroupファイルの中身が，名前空間が作成された時点に所属していたcgroupをルートとしたツリーで見えるようになる

# 親環境と同じツリーがみえちゃっている
root@ubuntu-bionic:~# ls -d /sys/fs/cgroup/memory/test01
/sys/fs/cgroup/memory/test01
root@ubuntu-bionic:~# cat /proc/13606/mountinfo | grep memory
41 30 0:36 /.. /sys/fs/cgroup/memory rw,nosuid,nodev,noexec,relatime shared:21 - cgroup cgroup rw,memory
root@ubuntu-bionic:~# cat /proc/13760/mountinfo | grep memory
288 262 0:36 /.. /sys/fs/cgroup/memory rw,nosuid,nodev,noexec,relatime - cgroup cgroup rw,memory
```

親環境から隔離するために、cgroupfsをマウントし直します。

```
# cgroupfsのマウントし直して、自分の名前スペースだけのcgroupにする
root@ubuntu-bionic:~# umount /sys/fs/cgroup/memory
root@ubuntu-bionic:~# cat /proc/self/mountinfo | grep '/sys/fs/cgroup/memory'
root@ubuntu-bionic:~# mount -t cgroup -o memory memory /sys/fs/cgroup/memory
root@ubuntu-bionic:~# cat /proc/self/mountinfo | grep '/sys/fs/cgroup/memory'
288 262 0:36 / /sys/fs/cgroup/memory rw,relatime - cgroup memory rw,memory
root@ubuntu-bionic:~# ls -d /sys/fs/cgroup/memory/test01
ls: cannot access '/sys/fs/cgroup/memory/test01': No such file or directory
root@ubuntu-bionic:~# cat /sys/fs/cgroup/memory/tasks 
13606
13760
13823

# 名前空間内でtest02グループを作成
root@ubuntu-bionic:~# mkdir /sys/fs/cgroup/memory/test02

# 親の名前空間からみてみると
vagrant@ubuntu-bionic:~$ ls -F /sys/fs/cgroup/memory/test01 | grep test
test02/
# test01グループ直下にtest02グループがある。
# 名前空間内では親と同じcgroupfsツリーを自身のcgroupをルートとしてそれ以下だけを見せている。
```


名前空間から抜ければtasksのプロセスはなくなるけどtest02は残ります。

```
vagrant@ubuntu-bionic:~$ cat /sys/fs/cgroup/memory/test01/test02/tasks
vagrant@ubuntu-bionic:~$ 
```


## 参考資料
* https://tenforward.hatenablog.com/entry/20160412/1460460722
* https://man7.org/linux/man-pages/man7/cgroup_namespaces.7.html
* https://gihyo.jp/admin/serial/01/linux_containers/0034?page=2
