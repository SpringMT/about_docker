# Mount Namespace(マウント名前空間)
マウントポイントはシステムで共通で、全部のプロセスが同じファイルパスで同じファイルを見られるはずです。

マウント名前空間を変えると、プロセスごと異なるデバイスをマウントし、別のファイルシステムを見せることができます。

マウント名前空間を使うと，名前空間内で行ったマウント操作を，他の名前空間には反映させないといったことができます。

もちろん，他の名前空間に反映させるという設定を行うこともできます。

これは /tmpなどを隔離するために使われることが多いです。(systemdのPrivateTmp機能など)

https://enakai00.hatenablog.com/entry/20130923/1379927579

unshareコマンドの -m オプションで実行します。

```
vagrant@ubuntu-bionic:~$ sudo unshare -m /bin/bash
root@ubuntu-bionic:~# touch /root/hosts

# https://pgmemo.tokyo/data/archives/1209.html mount -o bindの説明
# bindオプションを使うと、指定したファイルパスを、別のディレクトリにマウントすることができる。
# シンボリックリンクの代わりとなる
root@ubuntu-bionic:~# mount -o bind /etc/hosts /root/hosts
root@ubuntu-bionic:~# ls -l /root/hosts
-rw-r--r-- 1 root root 260 Nov 14 20:02 /root/hosts
root@ubuntu-bionic:~# cat /root/hosts
127.0.0.1   localhost
 
# The following lines are desirable for IPv6 capable hosts
::1 ip6-localhost   ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
ff02::3 ip6-allhosts
127.0.1.1   ubuntu-bionic   ubuntu-bionic
 
root@ubuntu-bionic:~# ls -l /proc/$$/ns/
total 0
lrwxrwxrwx 1 root root 0 Nov 14 20:45 cgroup -> 'cgroup:[4026531835]'
lrwxrwxrwx 1 root root 0 Nov 14 20:45 ipc -> 'ipc:[4026531839]'
lrwxrwxrwx 1 root root 0 Nov 14 20:45 mnt -> 'mnt:[4026532175]' <= PID 1と違う
lrwxrwxrwx 1 root root 0 Nov 14 20:45 net -> 'net:[4026531993]'
lrwxrwxrwx 1 root root 0 Nov 14 20:45 pid -> 'pid:[4026531836]'
lrwxrwxrwx 1 root root 0 Nov 14 20:45 pid_for_children -> 'pid:[4026531836]'
lrwxrwxrwx 1 root root 0 Nov 14 20:45 user -> 'user:[4026531837]'
lrwxrwxrwx 1 root root 0 Nov 14 20:45 uts -> 'uts:[4026531838]'
root@ubuntu-bionic:~# echo $$
9722
```

別のshellから見てみると

```
root@ubuntu-bionic:~# echo $$
9822
root@ubuntu-bionic:~# ls -l /root/hosts
-rw-r--r-- 1 root root 0 Nov 14 20:42 /root/hosts
root@ubuntu-bionic:~# cat /root/hosts
root@ubuntu-bionic:~#
```

別のMount Namespaceから見るとサイズが0で，ファイルの中身は空になります。

マウントはunshareコマンドで作成したマウント名前空間内でだけ有効であることがわかります。

> Mount Propagation
> マウント名前空間をまたいでマウント命令(mountやunmount)を伝播させる機能
> https://tech.retrieva.jp/entry/2019/04/16/155828#%E4%BD%99%E8%AB%871-Mount-Propagation%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6
> https://tech.retrieva.jp/entry/2019/04/16/155828#%E4%BD%99%E8%AB%871-Mount-Propagation%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6
> https://www.ibm.com/developerworks/jp/linux/library/l-mount-namespaces.html

作った名前空間は、その名前空間に属するプロセスが無くなると消滅します。

なので、umountし忘れてexitしたらどうしようもなくなった。。。。(結局作り直したが、、)

これはなにか解決する方法あるんかな、、

## 参考文献
* https://gihyo.jp/admin/serial/01/linux_containers/0002?page=2
* https://www.ibm.com/developerworks/jp/linux/library/l-mount-namespaces.html
* https://note.com/ryoma_0923/n/nd9e9fac44c73
* https://tech.retrieva.jp/entry/2019/04/16/155828#mnt-namespace
* https://kernhack.hatenablog.com/entry/2015/05/30/115705
