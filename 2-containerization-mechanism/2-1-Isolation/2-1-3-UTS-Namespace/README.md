# UTS Namespace(UTS名前空間)

UTSはUnix Time-sharing Systemの略だそうですが、今はその意味が失われています。

UTS名前空間はホスト名・ドメインを隔離します。

この機能によってコンテナごとにホスト名を指定できます。

```
vagrant@ubuntu-bionic:~$ hostname
ubuntu-bionic
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

vagrant@ubuntu-bionic:~$ sudo unshare --uts /bin/bash

root@ubuntu-bionic:~# ls -l /proc/$$/ns/
total 0
lrwxrwxrwx 1 root root 0 Nov 16 07:26 cgroup -> 'cgroup:[4026531835]'
lrwxrwxrwx 1 root root 0 Nov 16 07:26 ipc -> 'ipc:[4026531839]'
lrwxrwxrwx 1 root root 0 Nov 16 07:26 mnt -> 'mnt:[4026531840]'
lrwxrwxrwx 1 root root 0 Nov 16 07:26 net -> 'net:[4026531993]'
lrwxrwxrwx 1 root root 0 Nov 16 07:26 pid -> 'pid:[4026531836]'
lrwxrwxrwx 1 root root 0 Nov 16 07:26 pid_for_children -> 'pid:[4026531836]'
lrwxrwxrwx 1 root root 0 Nov 16 07:26 user -> 'user:[4026531837]'
lrwxrwxrwx 1 root root 0 Nov 16 07:26 uts -> 'uts:[4026532175]'
root@ubuntu-bionic:~# hostname
ubuntu-bionic
root@ubuntu-bionic:~# hostname "changed-hostname"
root@ubuntu-bionic:~# hostname
changed-hostname # <= 変わっている
root@ubuntu-bionic:~# cat /etc/hostname
ubuntu-bionic  # <= /etc配下は共通なので、ここは元のhostnameがでてくる
root@ubuntu-bionic:~# exit
exit
vagrant@ubuntu-bionic:~$ hostname
ubuntu-bionic
```

## 参考資料
* https://tenforward.hatenablog.com/entry/20140812/1407834440
* https://gihyo.jp/admin/serial/01/linux_containers/0002?page=2
