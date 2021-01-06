# cgroup
## 概要
cgroupではプロセスが利用できるリソースを制限することができます。

cgroupの機能は具体的に表現すると以下のようになります。

* CPU時間の制限、割り当てCPUの指定
* メモリ使用量の制限、OOM killerの有効化/無効化
* プロセス数の確認と制限
* デバイスのアクセス制御
* ネットワーク優先度設定
* タスクの一時停止/再開
* CPU使用量、メモリ使用量のレポート
* ネットワークパケットをタグ付け(tc で利用)
* 名前空間と組み合わせることでプロセスを仮想マシンのように扱うことができます。

https://eh-career.com/engineerhub/entry/2019/02/05/103000#cgroup より

cgroupはv1, v2がありますがここではv1を主に使います。

cgroup機能を使ってプロセスをグループ化したものをcgroupと呼びます。

## cgroupの操作方法
### cgourpfs
cgroupはcgroupファイルシステム（以降cgroupfs）という仮想的なファイルシステムを使って操作します。(/procとかと似たような感じ)

### cgroup-tools(libcgroup)パッケージ
cgroup-toolsパッケージで提供されるコマンドで操作可能です。

* cgcreateコマンド: 新しくサブグループを作成します。
* cgdeleteコマンド: 指定したサブグループを削除します。
* cgexecコマンド: 指定したサブグループでコマンドを実行します。

## サブシステム
cgroupも扱うリソースによってサブシステムと呼ばれる独立した機能でリソースを扱います。

サブシステム | 機能
-|-
blkio | ブロックデバイスの入出力アクセスを制御 (Read <= N bytes/sec)
cpu | スケジューラーを使用して cgroup タスクのCPUのアクセスを制御
cpuset | マルチコアの場合に利用する CPU, メモリノードを制御
devices | デバイスへのアクセス制御
freezer | cgroup 内のタスクの一時停止, 再開
memory | メモリリソースの制御
net_cls | Linux トラフィックコントローラ (tc or netfilter) がパケットを識別できるよう, クラス識別子 (classid) によりネットワークパケットにタグをつける
net_prio | NIC 別にトラフィックのプライオリティを設定する
### 参照
* https://access.redhat.com/documentation/ja-jp/red_hat_enterprise_linux/6/html/resource_management_guide/ch01
* https://gihyo.jp/admin/serial/01/linux_containers/0004

## 階層構造
通常のファイルシステムに階層構造でディレクトリと作るのと同じようにcgroupを作れます。

cgroupfsのトップディレクトリをルートにした階層構造になります。

![](https://gihyo.jp/assets/images/admin/serial/01/linux_containers/0003/thumb/TH800_001.jpg)

参照 : https://gihyo.jp/admin/serial/01/linux_containers/0003?page=2

この図では，「⁠デスクトップアプリ」と「デーモン」グループにそれぞれ30％ずつCPUを割り当てていますので，そのさらに下のcgroupは親の割り当てである30％からリソースが割り当てられます。

「⁠ブラウザ⁠」、「⁠エディタ」グループに親である「デスクトップアプリ」グループに割り当てられたリソースの半分ずつを割り当てるとそれぞれ15％が割り当てられます。

cgroupの親子関係の間では、子のcgroupは親のcgroupの制限を受けます。

絶対値で制限を設定するような場合、子のcgroupで親のcgroupの制限を超えるような設定を行っても先に親のcgroupの制限に引っかかります。

## 確認
まずはmountの状態

```
vagrant@ubuntu-bionic:~$ uname -a
Linux ubuntu-bionic 4.15.0-123-generic #126-Ubuntu SMP Wed Oct 21 09:40:11 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
vagrant@ubuntu-bionic:~$ mount -t cgroup
cgroup on /sys/fs/cgroup/systemd type cgroup (rw,nosuid,nodev,noexec,relatime,xattr,name=systemd)
cgroup on /sys/fs/cgroup/net_cls,net_prio type cgroup (rw,nosuid,nodev,noexec,relatime,net_cls,net_prio)
cgroup on /sys/fs/cgroup/rdma type cgroup (rw,nosuid,nodev,noexec,relatime,rdma)
cgroup on /sys/fs/cgroup/freezer type cgroup (rw,nosuid,nodev,noexec,relatime,freezer)
cgroup on /sys/fs/cgroup/pids type cgroup (rw,nosuid,nodev,noexec,relatime,pids)
cgroup on /sys/fs/cgroup/cpu,cpuacct type cgroup (rw,nosuid,nodev,noexec,relatime,cpu,cpuacct)
cgroup on /sys/fs/cgroup/memory type cgroup (rw,nosuid,nodev,noexec,relatime,memory)
cgroup on /sys/fs/cgroup/devices type cgroup (rw,nosuid,nodev,noexec,relatime,devices)
cgroup on /sys/fs/cgroup/cpuset type cgroup (rw,nosuid,nodev,noexec,relatime,cpuset)
cgroup on /sys/fs/cgroup/blkio type cgroup (rw,nosuid,nodev,noexec,relatime,blkio)
cgroup on /sys/fs/cgroup/hugetlb type cgroup (rw,nosuid,nodev,noexec,relatime,hugetlb)
cgroup on /sys/fs/cgroup/perf_event type cgroup (rw,nosuid,nodev,noexec,relatime,perf_event)
vagrant@ubuntu-bionic:~$ mount -t cgroup2
cgroup on /sys/fs/cgroup/unified type cgroup2 (rw,nosuid,nodev,noexec,relatime)
```


有効になっているcgroupサブシステムの確認

```
vagrant@ubuntu-bionic:~$ cat /proc/cgroups 
#subsys_name	hierarchy	num_cgroups	enabled
cpuset	9	1	1
cpu	6	63	1
cpuacct	6	63	1
blkio	10	63	1
memory	7	85	1
devices	8	63	1
freezer	4	1	1
net_cls	2	1	1
perf_event	12	1	1
net_prio	2	1	1
hugetlb	11	1	1
pids	5	69	1
rdma	3	1	1
```

### cgroupの作成

作成は通常のファイルシステムと同じようにcgroupfs上でのmkdirコマンドでできます。

```
vagrant@ubuntu-bionic:~$ cd /sys/fs/cgroup/cpu
vagrant@ubuntu-bionic:/sys/fs/cgroup/cpu$ sudo mkdir test01
vagrant@ubuntu-bionic:/sys/fs/cgroup/cpu$ ls
cgroup.clone_children  cpu.cfs_period_us  cpu.stat       cpuacct.usage_all         cpuacct.usage_percpu_user  notify_on_release  tasks
cgroup.procs           cpu.cfs_quota_us   cpuacct.stat   cpuacct.usage_percpu      cpuacct.usage_sys          release_agent      test01 <= test01ができている
cgroup.sane_behavior   cpu.shares         cpuacct.usage  cpuacct.usage_percpu_sys  cpuacct.usage_user         system.slice       user.slice
vagrant@ubuntu-bionic:/sys/fs/cgroup/cpu$ ls test01/
cgroup.clone_children  cpu.cfs_period_us  cpu.shares  cpuacct.stat   cpuacct.usage_all     cpuacct.usage_percpu_sys   cpuacct.usage_sys   notify_on_release
cgroup.procs           cpu.cfs_quota_us   cpu.stat    cpuacct.usage  cpuacct.usage_percpu  cpuacct.usage_percpu_user  cpuacct.usage_user  tasks

vagrant@ubuntu-bionic:/sys/fs/cgroup/cpu$ cat /proc/self/cgroup
12:perf_event:/
11:hugetlb:/
10:blkio:/user.slice
9:cpuset:/
8:devices:/user.slice
7:memory:/user.slice
6:cpu,cpuacct:/test01 <= cpuのサブシステムにtest01が指定されている
5:pids:/user.slice/user-1000.slice/session-6.scope
4:freezer:/
3:rdma:/
2:net_cls,net_prio:/
1:name=systemd:/user.slice/user-1000.slice/session-6.scope
0::/user.slice/user-1000.slice/session-6.scope
```

test01配下には自動的にファイルが作られます。

### プロセスの登録
tasksファイルにPIDを書き込めば登録できます。

```
vagrant@ubuntu-bionic:/sys/fs/cgroup/cpu$ cat test01/tasks
vagrant@ubuntu-bionic:/sys/fs/cgroup/cpu$ 
vagrant@ubuntu-bionic:/sys/fs/cgroup/cpu$ echo $$ | sudo tee -a /sys/fs/cgroup/cpu/test01/tasks
9575
vagrant@ubuntu-bionic:/sys/fs/cgroup/cpu$ cat /sys/fs/cgroup/cpu/test01/tasks
9575
10819 <= catに使ったPID cgroupに登録したプロセスの子プロセスやその子孫も自動的に同じcgroupに登録される
```

### cgroupの削除
tasksの中から該当のプロセスを追い出せばrmdirでも削除できます

```
vagrant@ubuntu-bionic:/sys/fs/cgroup/cpu$ sudo cgdelete cpu:test01
vagrant@ubuntu-bionic:/sys/fs/cgroup/cpu$
vagrant@ubuntu-bionic:/sys/fs/cgroup/cpu$ cat /sys/fs/cgroup/cpu/test01/tasks 
cat: /sys/fs/cgroup/cpu/test01/tasks: No such file or directory
```

### リソースの制限

```
vagrant@ubuntu-bionic:/sys/fs/cgroup/cpu$ UUID=$(uuidgen)
vagrant@ubuntu-bionic:/sys/fs/cgroup/cpu$ sudo mkdir /sys/fs/cgroup/cpu/$UUID
vagrant@ubuntu-bionic:/sys/fs/cgroup/cpu$ sudo echo 50000 | sudo tee /sys/fs/cgroup/cpu/$UUID/cpu.cfs_quota_us <=cfs_quota_usは cgroup内のタスクに与えられるCPU時間(µs)
50000
vagrant@ubuntu-bionic:/sys/fs/cgroup/cpu$ echo $$ | sudo tee /sys/fs/cgroup/cpu/$UUID/cgroup.procs <= procsは cgroupに属するプロセスの登録、確認
9575
```

制限がかかっている様子

```
vagrant@ubuntu-bionic:/sys/fs/cgroup/cpu$ timeout 15s yes >/dev/null &
[1] 10862
vagrant@ubuntu-bionic:/sys/fs/cgroup/cpu$ top

top - 19:55:10 up  9:37,  2 users,  load average: 0.24, 0.07, 0.02
Tasks: 101 total,   2 running,  55 sleeping,   0 stopped,   0 zombie
%Cpu(s): 12.3 us,  7.5 sy,  0.0 ni, 80.2 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  1008600 total,   121196 free,   150520 used,   736884 buff/cache
KiB Swap:        0 total,        0 free,        0 used.   694872 avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND 
10863 vagrant   20   0    7932    820    756 R  50.0  0.1   0:03.62 yes <= yesコマンドのCPU利用量が50%前後で頭打ちになっている
 3122 root      20   0  911924  45800  26112 S   0.7  4.5   2:25.45 containerd
```

### cgroup Namespace


## 参考資料
* LXCを題材にしたcgroupの説明
  * https://gihyo.jp/admin/serial/01/linux_containers/0004 
  * https://gihyo.jp/admin/serial/01/linux_containers/0003 
* https://eh-career.com/engineerhub/entry/2019/02/05/103000#cgroup 
* https://access.redhat.com/documentation/ja-jp/red_hat_enterprise_linux/6/html/resource_management_guide/ch01
* https://ja.wikipedia.org/wiki/Cgroups
* https://man7.org/linux/man-pages/man7/cgroups.7.html
* cgroupsでリソース制限
  * https://christina04.hatenablog.com/entry/2020/02/24/170724
  * https://eh-career.com/engineerhub/entry/2019/02/05/103000#cgroup
  * https://qiita.com/Surgo/items/709a07d68c6eafbad267
