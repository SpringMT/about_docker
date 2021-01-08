# 隔離
LinuxにはNamespace(名前空間)という仕組みがあります。

`名前空間内のプロセスに対して、 自分たちが専用の分離されたグローバルリソースを持っているかのように見せる仕組み` となります。

https://linuxjm.osdn.jp/html/LDP_man-pages/man7/namespaces.7.html

プロセスをグループ化して，コンテナの隔離された空間を作り出す機能に重要な役割を果たします。

名前空間は『名前空間』という機能がひとつ存在するわけでなく，独立させたいリソースによっていくつかの機能があります。

Namespaceは現在8種類あります。

https://man7.org/linux/man-pages/man7/namespaces.7.html

name | 説明 | kernel version
-|-|-
Mount Namespace	| マウントポイントを隔離してプロセス独自のファイルシステムを扱えるようにします。	| 2.4.19
PID Namespace	| PID番号空間を隔離してユニークなPIDを持ちます。新しいNamespaceで最初に作成されたプロセスはPID1となり、通常のPID1プロセスと同様の特性を持ちます。 https://linuxjm.osdn.jp/html/LDP_man-pages/man7/pid_namespaces.7.html | 2.6.24
Network Namespace |	ネットワークスタックを隔離します。 |	2.6.26
IPC Namespace	SysV | IPCオブジェクト、 POSIXキューを隔離します。(IPC : Inter-Process Communication) | 2.6.19
UTS Namespace | ホスト名やNISドメイン名など、 unameシステムコールで返される情報を隔離します。(UTS : UNIX Time Sharing) | 2.6.19
User Namespace | User ID, Group IDを隔離します。Namespace内ではUser IDが0で特権ユーザーである一方、他のNamespaceからは非特権ユーザーとして扱われる、という状態を持つことができます。https://linuxjm.osdn.jp/html/LDP_man-pages/man7/user_namespaces.7.html | 3.8
Cgroup Namespace | cgroupルートディレクトリを隔離します。新しくCgroup Namespaceを作成すると現在のcgroupディレクトリがcgroupルートディレクトリになります。https://gihyo.jp/admin/serial/01/linux_containers/0034 | 4.4
Time Namespace | 時間を隔離します。 Linux 5.6でmergeされたらしい。 https://tenforward.hatenablog.com/entry/2020/04/01/120142 https://udzura.hatenablog.jp/entry/2020/04/08/002542 | 5.6

### 参照

* https://eh-career.com/engineerhub/entry/2019/02/05/103000
* https://man7.org/linux/man-pages/man7/namespaces.7.html
* https://linuxjm.osdn.jp/html/LDP_man-pages/man7/namespaces.7.html
* https://gihyo.jp/admin/serial/01/linux_containers/0002?page=2

## unshare(2)
呼び出したプロセスを新しい名前空間に移動します。

unshareコマンド(unshare(1))は 新しいNamespaceを作成し、指定したコマンドを実行します。

https://man7.org/linux/man-pages/man1/unshare.1.html

## Namespace(名前空間)の確認
プロセスのNamespaceの確認は /proc/<PID>/ns でできます。

このサブディレクトリには、名前空間毎に 1 エントリーが置かれます。

https://linuxjm.osdn.jp/html/LDP_man-pages/man5/proc.5.html

例
```
root@ubuntu-bionic:~# ls -l /proc/$$/ns/ 
total 0
lrwxrwxrwx 1 root root 0 Nov 14 20:45 cgroup -> 'cgroup:[4026531835]'
lrwxrwxrwx 1 root root 0 Nov 14 20:45 ipc -> 'ipc:[4026531839]'
lrwxrwxrwx 1 root root 0 Nov 14 20:45 mnt -> 'mnt:[4026531840]'
lrwxrwxrwx 1 root root 0 Nov 14 20:45 net -> 'net:[4026531993]'
lrwxrwxrwx 1 root root 0 Nov 14 20:45 pid -> 'pid:[4026531836]'
lrwxrwxrwx 1 root root 0 Nov 14 20:45 pid_for_children -> 'pid:[4026531836]'
lrwxrwxrwx 1 root root 0 Nov 14 20:45 user -> 'user:[4026531837]'
lrwxrwxrwx 1 root root 0 Nov 14 20:45 uts -> 'uts:[4026531838]'
```

ここにある各ファイル（シンボリックリンク）はNamespaceの種別とinode番号で構成される文字列です。この文字列が同じプロセス同士はNamespaceを共有しています。

```
root@ubuntu-bionic:~# ls -l /proc/$$/ns/ 
total 0
lrwxrwxrwx 1 root root 0 Nov 14 20:45 cgroup -> 'cgroup:[4026531835]'
lrwxrwxrwx 1 root root 0 Nov 14 20:45 ipc -> 'ipc:[4026531839]'
lrwxrwxrwx 1 root root 0 Nov 14 20:45 mnt -> 'mnt:[4026531840]'
lrwxrwxrwx 1 root root 0 Nov 14 20:45 net -> 'net:[4026531993]'
lrwxrwxrwx 1 root root 0 Nov 14 20:45 pid -> 'pid:[4026531836]'
lrwxrwxrwx 1 root root 0 Nov 14 20:45 pid_for_children -> 'pid:[4026531836]'
lrwxrwxrwx 1 root root 0 Nov 14 20:45 user -> 'user:[4026531837]'
lrwxrwxrwx 1 root root 0 Nov 14 20:45 uts -> 'uts:[4026531838]'
root@ubuntu-bionic:~# ls -l /proc/1/ns/ 
total 0
lrwxrwxrwx 1 root root 0 Nov 14 20:53 cgroup -> 'cgroup:[4026531835]'
lrwxrwxrwx 1 root root 0 Nov 14 20:53 ipc -> 'ipc:[4026531839]'
lrwxrwxrwx 1 root root 0 Nov 14 20:53 mnt -> 'mnt:[4026531840]'
lrwxrwxrwx 1 root root 0 Nov 14 20:53 net -> 'net:[4026531993]'
lrwxrwxrwx 1 root root 0 Nov 14 20:53 pid -> 'pid:[4026531836]'
lrwxrwxrwx 1 root root 0 Nov 14 20:53 pid_for_children -> 'pid:[4026531836]'
lrwxrwxrwx 1 root root 0 Nov 14 20:53 user -> 'user:[4026531837]'
lrwxrwxrwx 1 root root 0 Nov 14 20:53 uts -> 'uts:[4026531838]'
```

PID 1のプロセスとNamespaceを共有していることがわかります。
