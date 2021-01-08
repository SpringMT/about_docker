# User Namespace(ユーザー名前空間)
ユーザ名前空間はUID、GIDの空間を名前空間ごとに独立して持つことができます。

名前空間内のUID、GIDとホストOS上のUID、GIDとが紐づけられ、名前空間の中と外で異なるUID、GIDを持つことができます。

この機能が最も効果を発揮する場面がコンテナ内のrootユーザを扱うときです。

ホストOS上のUID、GIDと紐づけを行うことができるので、コンテナ内ではUID、GID共に0のrootユーザを、ホストOS上ではたとえばUID、GID共に1000である一般ユーザとして扱うことができます。

コンテナのrootがホストOS上では特権を持たないことになるので、セキュアにコンテナ環境を提供できるようになります。

ユーザ名前空間は他の名前空間と異なり、上記の特徴があるため一般ユーザで作成できます。

```
vagrant@ubuntu-bionic:~$ whoami ; id -u ; id -g
vagrant
1000
1000

vagrant@ubuntu-bionic:~$ unshare --user

nobody@ubuntu-bionic:~$ whoami ; id -u ; id -g
nobody
65534 <= Uidが変わっている
65534　<= Gidが変わっている
nobody@ubuntu-bionic:~$ id
uid=65534(nobody) gid=65534(nogroup) groups=65534(nogroup)
nobody@ubuntu-bionic:~$ echo $$ 
10318
nobody@ubuntu-bionic:~$  ls -ld /proc/10318 
dr-xr-xr-x 9 nobody nogroup 0 Nov 16 08:16 /proc/10318
nobody@ubuntu-bionic:~$ grep "[U|G]id" /proc/10318/status
Uid:	65534	65534	65534	65534
Gid:	65534	65534	65534	65534
```

このように新しいUID、GIDが振られます。

新しいUIDとGIDはホストマッピングされてないため、UIDとして/proc/sys/kernel/overflowuidの値、GIDとして/proc/sys/kernel/overflowgidの値が設定されます。

親の名前空間ではどうなっているかというと

```
vagrant@ubuntu-bionic:~$ grep "[U|G]id" /proc/10318/status
Uid:	1000	1000	1000	1000
Gid:	1000	1000	1000	1000
```

となり、デフォルトでは親のIDがそのままマッピングされています。

新しく作ったユーザー名前空間内のUID、GIDを0に設定してrootにしつつ、親の名前空間のUID GID 1000/1000に割り当ててみます。

これは親の名前空間から実行する必要があります。

```
vagrant@ubuntu-bionic:~$ echo '0 1000 1' > /proc/10318/uid_map
vagrant@ubuntu-bionic:~$ echo '0 1000 1' > /proc/10318/uid_map
-bash: echo: write error: Operation not permitted <= 二回目は失敗する
vagrant@ubuntu-bionic:~$ echo '0 1000 1' > /proc/10318/gid_map
-bash: echo: write error: Operation not permitted <= 一般ユーザ権限では書き込みできない
vagrant@ubuntu-bionic:~$ echo '0 1000 1' | sudo tee /proc/10318/gid_map
0 1000 1
```

子の名前空間で確認してみます

```
nobody@ubuntu-bionic:~$ grep "[U|G]id" /proc/$$/status
Uid:	0	0	0	0
Gid:	0	0	0	0
```

rootになっていることがわかります。

試しにファイルを作ってみます。

```
nobody@ubuntu-bionic:~$ pwd
/home/vagrant
nobody@ubuntu-bionic:~$ touch root.txt
nobody@ubuntu-bionic:~$ ls -al root.txt 
-rw-rw-r-- 1 root root 0 Nov 16 08:51 root.txt
```

このファイルを親の名前空間から見てみます。

```
vagrant@ubuntu-bionic:~$ ls -al root.txt 
-rw-rw-r-- 1 vagrant vagrant 0 Nov 16 08:51 root.txt
```
親の名前空間上のユーザの所有のファイルとして見えています。

## 参考資料
* https://gihyo.jp/admin/serial/01/linux_containers/0016
* https://tech.retrieva.jp/entry/2019/06/04/130134
