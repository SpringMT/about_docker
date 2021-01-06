# Network Namespace(ネットワーク名前空間)
ネットワーク名前空間は各種のネットワークリソースを名前空間ごとに隔離します。

ネットワークデバイス，IPアドレス，ルーティングテーブル，ポート番号，フィルタリングテーブルなどを隔離できます。

## 隔離

まず、unsahreコマンドを使ってネットワーク名前空間を隔離します。

```
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
vagrant@ubuntu-bionic:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 02:ad:b8:da:8b:56 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 66816sec preferred_lft 66816sec
    inet6 fe80::ad:b8ff:feda:8b56/64 scope link 
       valid_lft forever preferred_lft forever
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 02:42:05:66:18:d2 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
vagrant@ubuntu-bionic:~$ route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         10.0.2.2        0.0.0.0         UG    100    0        0 enp0s3
10.0.2.0        0.0.0.0         255.255.255.0   U     0      0        0 enp0s3
10.0.2.2        0.0.0.0         255.255.255.255 UH    100    0        0 enp0s3
172.17.0.0      0.0.0.0         255.255.0.0     U     0      0        0 docker0
vagrant@ubuntu-bionic:~$ sudo iptables -L -n -v
Chain INPUT (policy ACCEPT 31001 packets, 42M bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain FORWARD (policy DROP 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 DOCKER-USER  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
    0     0 DOCKER-ISOLATION-STAGE-1  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
    0     0 ACCEPT     all  --  *      docker0  0.0.0.0/0            0.0.0.0/0            ctstate RELATED,ESTABLISHED
    0     0 DOCKER     all  --  *      docker0  0.0.0.0/0            0.0.0.0/0           
    0     0 ACCEPT     all  --  docker0 !docker0  0.0.0.0/0            0.0.0.0/0           
    0     0 ACCEPT     all  --  docker0 docker0  0.0.0.0/0            0.0.0.0/0           

Chain OUTPUT (policy ACCEPT 14309 packets, 722K bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain DOCKER (1 references)
 pkts bytes target     prot opt in     out     source               destination         

Chain DOCKER-ISOLATION-STAGE-1 (1 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 DOCKER-ISOLATION-STAGE-2  all  --  docker0 !docker0  0.0.0.0/0            0.0.0.0/0           
    0     0 RETURN     all  --  *      *       0.0.0.0/0            0.0.0.0/0           

Chain DOCKER-ISOLATION-STAGE-2 (1 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 DROP       all  --  *      docker0  0.0.0.0/0            0.0.0.0/0           
    0     0 RETURN     all  --  *      *       0.0.0.0/0            0.0.0.0/0           

Chain DOCKER-USER (1 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 RETURN     all  --  *      *       0.0.0.0/0            0.0.0.0/0           

# ネットワーク名前空間を隔離する
vagrant@ubuntu-bionic:~$ sudo unshare --net /bin/bash

root@ubuntu-bionic:~# ls -l /proc/$$/ns/
total 0
lrwxrwxrwx 1 root root 0 Nov 16 07:30 cgroup -> 'cgroup:[4026531835]'
lrwxrwxrwx 1 root root 0 Nov 16 07:30 ipc -> 'ipc:[4026531839]'
lrwxrwxrwx 1 root root 0 Nov 16 07:30 mnt -> 'mnt:[4026531840]'
lrwxrwxrwx 1 root root 0 Nov 16 07:30 net -> 'net:[4026532176]' # before 4026531993
lrwxrwxrwx 1 root root 0 Nov 16 07:30 pid -> 'pid:[4026531836]'
lrwxrwxrwx 1 root root 0 Nov 16 07:30 pid_for_children -> 'pid:[4026531836]'
lrwxrwxrwx 1 root root 0 Nov 16 07:30 user -> 'user:[4026531837]'
lrwxrwxrwx 1 root root 0 Nov 16 07:30 uts -> 'uts:[4026531838]'
root@ubuntu-bionic:~# ip a
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
root@ubuntu-bionic:~# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
root@ubuntu-bionic:~# iptables -L -n -v
Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain OUTPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         
```

unshareコマンドで作った新しいネットワーク名前空間には、loopback interfaceしかできないため、親の名前空間とも通信できません。

> loopback interface
> 自身とのみ通信可能なインターフェース

ルーティングテーブルもフィルタリングテーブルも空のものができます。

ここから、名前空間同士を繋ぐ仮想インターフェースを作ったりして、ネットワークがつながるようにします。

## プライベートネットワークの作成
隔離したネットワーク名前空間でプライベートネットワークを作ってみます。

ネットワークブリッジを作ります。

[ブリッジ (ネットワーク機器)](https://ja.wikipedia.org/wiki/%E3%83%96%E3%83%AA%E3%83%83%E3%82%B8_(%E3%83%8D%E3%83%83%E3%83%88%E3%83%AF%E3%83%BC%E3%82%AF%E6%A9%9F%E5%99%A8))

```
vagrant@ubuntu-bionic:~$ sudo ip link add name br0 type bridge
vagrant@ubuntu-bionic:~$ sudo ip addr add 192.168.77.1/24 broadcast 192.168.77.255 label br0 dev br0
vagrant@ubuntu-bionic:~$ sudo ip link set dev br0 up
vagrant@ubuntu-bionic:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 02:ad:b8:da:8b:56 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 63522sec preferred_lft 63522sec
    inet6 fe80::ad:b8ff:feda:8b56/64 scope link 
       valid_lft forever preferred_lft forever
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 02:42:05:66:18:d2 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
4: br0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN group default qlen 1000
    link/ether 9a:a1:11:30:da:af brd ff:ff:ff:ff:ff:ff
    inet 192.168.77.1/24 brd 192.168.77.255 scope global br0
       valid_lft forever preferred_lft forever
    inet6 fe80::98a1:11ff:fe30:daaf/64 scope link 
       valid_lft forever preferred_lft forever
```

ip netnsコマンドでネットワーク名前空間(ns0)を作成します。

```
sudo ip netns add ns0
```

vethペア(veth0, veth0_peer)を作成し、片方のmasterをネットワークブリッジ(br0)に追加、もう片方を隔離したNetwork Namespace(ns0)に移動します。

```
vagrant@ubuntu-bionic:~$ sudo ip link add veth0 type veth peer name veth0_peer
vagrant@ubuntu-bionic:~$ sudo ip link set dev veth0 master br0
vagrant@ubuntu-bionic:~$ sudo ip link set dev veth0 up
vagrant@ubuntu-bionic:~$ sudo ip link set dev veth0_peer netns ns0
vagrant@ubuntu-bionic:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 02:ad:b8:da:8b:56 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 63323sec preferred_lft 63323sec
    inet6 fe80::ad:b8ff:feda:8b56/64 scope link 
       valid_lft forever preferred_lft forever
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 02:42:05:66:18:d2 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
4: br0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default qlen 1000
    link/ether ca:94:d4:4a:b9:fa brd ff:ff:ff:ff:ff:ff
    inet 192.168.77.1/24 brd 192.168.77.255 scope global br0
       valid_lft forever preferred_lft forever
    inet6 fe80::98a1:11ff:fe30:daaf/64 scope link 
       valid_lft forever preferred_lft forever
6: veth0@if5: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue master br0 state LOWERLAYERDOWN group default qlen 1000
    link/ether ca:94:d4:4a:b9:fa brd ff:ff:ff:ff:ff:ff link-netnsid 0
```

ns0のveth(veth0_peer -> eth0)にアドレスを割り当て、デフォルトゲートウェイを設定します。

```
vagrant@ubuntu-bionic:~$ sudo ip netns exec ns0 ip link set dev veth0_peer name eth0
vagrant@ubuntu-bionic:~$ sudo ip netns exec ns0 ip addr add 192.168.77.2/24 dev eth0
vagrant@ubuntu-bionic:~$ sudo ip netns exec ns0 ip link set dev eth0 up
vagrant@ubuntu-bionic:~$ sudo ip netns exec ns0 ip route add default via 192.168.77.1
vagrant@ubuntu-bionic:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 02:ad:b8:da:8b:56 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 63245sec preferred_lft 63245sec
    inet6 fe80::ad:b8ff:feda:8b56/64 scope link 
       valid_lft forever preferred_lft forever
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 02:42:05:66:18:d2 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
4: br0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether ca:94:d4:4a:b9:fa brd ff:ff:ff:ff:ff:ff
    inet 192.168.77.1/24 brd 192.168.77.255 scope global br0
       valid_lft forever preferred_lft forever
    inet6 fe80::98a1:11ff:fe30:daaf/64 scope link 
       valid_lft forever preferred_lft forever
6: veth0@if5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master br0 state UP group default qlen 1000
    link/ether ca:94:d4:4a:b9:fa brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::c894:d4ff:fe4a:b9fa/64 scope link 
       valid_lft forever preferred_lft forever
```

これでネットワークブリッジ(br0)と隔離したNamespace(ns0)のvethが通信できるようになりました。

ip netns execコマンドで隔離したNamespaceを参照するコンテナを作成し、br0(192.168.77.1)やホストアドレスにpingして疎通を確認します。

```
vagrant@ubuntu-bionic:~$ sudo ip netns exec ns0 ping -c 3 192.168.77.1
PING 192.168.77.1 (192.168.77.1) 56(84) bytes of data.
64 bytes from 192.168.77.1: icmp_seq=1 ttl=64 time=0.179 ms
64 bytes from 192.168.77.1: icmp_seq=2 ttl=64 time=0.064 ms
64 bytes from 192.168.77.1: icmp_seq=3 ttl=64 time=0.051 ms

--- 192.168.77.1 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2041ms
rtt min/avg/max/mdev = 0.051/0.098/0.179/0.057 ms
```

コンテナ内から外部ネットワークと通信する場合はホスト側で適宜NATを設定する必要があります。

ns0を削除するとvethペアは自動的に削除されます。

```
vagrant@ubuntu-bionic:~$ sudo ip netns del ns0
vagrant@ubuntu-bionic:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 02:ad:b8:da:8b:56 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 63120sec preferred_lft 63120sec
    inet6 fe80::ad:b8ff:feda:8b56/64 scope link 
       valid_lft forever preferred_lft forever
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 02:42:05:66:18:d2 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
4: br0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default qlen 1000
    link/ether 00:00:00:00:00:00 brd ff:ff:ff:ff:ff:ff
    inet 192.168.77.1/24 brd 192.168.77.255 scope global br0
       valid_lft forever preferred_lft forever
    inet6 fe80::98a1:11ff:fe30:daaf/64 scope link 
       valid_lft forever preferred_lft forever
vagrant@ubuntu-bionic:~$ sudo ip link del dev br0
vagrant@ubuntu-bionic:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 02:ad:b8:da:8b:56 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 63112sec preferred_lft 63112sec
    inet6 fe80::ad:b8ff:feda:8b56/64 scope link 
       valid_lft forever preferred_lft forever
3: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default 
    link/ether 02:42:05:66:18:d2 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
```

## 参考資料
* https://gihyo.jp/admin/serial/01/linux_containers/0002?page=3
* https://eh-career.com/engineerhub/entry/2019/02/05/103000#Network-Namespace%E3%82%92%E4%BD%BF%E3%81%A3%E3%81%9F%E3%83%97%E3%83%A9%E3%82%A4%E3%83%99%E3%83%BC%E3%83%88%E3%83%8D%E3%83%83%E3%83%88%E3%83%AF%E3%83%BC%E3%82%AF%E3%81%AE%E4%BD%9C%E6%88%90
* https://tech.retrieva.jp/entry/2019/04/16/155828#net-namespace
