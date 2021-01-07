# OverlayFS
同じホストで同じ構成のコンテナを複数作成する場合、各コンテナ同士では多くの共有可能なバイナリやライブラリがあります。

コンテナごとにファイルシステムをまるごと用意するとディスクスペースを無駄に消費することになります。

また、大きなコンテナの場合、ファイルシステムの作成にも時間がかかりコンテナの起動が遅くなってしまう可能性もあります。

こうした無駄を改善するのが、COW（Copy On Write）ファイルシステム [UnionFS](https://en.wikipedia.org/wiki/UnionFS) と呼ばれるものです。

OverlayFSは、UnionFSの実装の1つで、ディレクトリを重ね合わせて1つのディレクトリツリーが構成できます。

Linux カーネル 3.18から導入されました。(https://github.com/torvalds/linux/commit/e9be9d5e76e34872f0c37d72e25bc27fe9e2c54c)

## OverlayFSは

OverlayFSは下記のディレクトリで構成されます。

* lowerdir: 下位に位置するディレクトリでファイルシステムのベースとなるディレクトリです。このディレクトリは読み取り専用です。
* upperdir: 上位に位置するディレクトリで、lowerdirに対して新規作成、変更、削除されたファイルが書き出されるディレクトリです。
* workdir: OverlayFSの内部で使用される作業用ディレクトリです。
* merged: lowerdirとupperdirが結合されたディレクトリです。OverlayFSをマウントしたプロセスでは、作業者が閲覧するのはこのディレクトリとなります。

参照 : https://gihyo.jp/admin/serial/01/linux_containers/0018

OverlayFSは通常のファイルシステム同様、mountシステムコールやmountコマンドで操作します。

### 試してみる
以下はmountコマンドでOverlayFSを利用する例です。

```
# 規定のdirを作る
vagrant@ubuntu-bionic:~$ mkdir lower upper merged work
vagrant@ubuntu-bionic:~$ sudo mount -t overlay -o lowerdir=lower,upperdir=upper,workdir=work overlay merged
vagrant@ubuntu-bionic:~$ mount -l | grep overlay
overlay on /home/vagrant/merged type overlay (rw,relatime,lowerdir=lower,upperdir=upper,workdir=work)
```

mergedディレクトリに新しくファイルを作成してみます。

```
vagrant@ubuntu-bionic:~$ touch merged/testfile_overlay
vagrant@ubuntu-bionic:~$ ls -F merged/
testfile_overlay
vagrant@ubuntu-bionic:~$ ls -F lower/
vagrant@ubuntu-bionic:~$ ls -F upper/ <= 上層であるupperdirディレクトリにファイルの変更がある
testfile_overlay
```

unmountしてみます

```
vagrant@ubuntu-bionic:~$ sudo umount overlay
vagrant@ubuntu-bionic:~$ mount -l | grep overlay
vagrant@ubuntu-bionic:~$ tree ./
./
├── lower
├── merged
├── upper
│   └── testfile_overlay
└── work
    └── work [error opening dir] # <= rootになっている

5 directories, 1 file
```


それでは、上層と下層にファイルがある状態で試してみます。

```
vagrant@ubuntu-bionic:~$ touch lower/testfile_lower <= 下層にファイル
vagrant@ubuntu-bionic:~$ touch upper/testfile_upper <= 上層にファイル
vagrant@ubuntu-bionic:~$ tree ./
./
├── lower
│   └── testfile_lower
├── merged
├── upper
│   ├── testfile_overlay
│   └── testfile_upper
└── work
    └── work [error opening dir]

5 directories, 3 files
vagrant@ubuntu-bionic:~$ sudo mount -t overlay -o lowerdir=lower,upperdir=upper,workdir=work overlay merged
vagrant@ubuntu-bionic:~$ tree ./
./
├── lower
│   └── testfile_lower
├── merged
│   ├── testfile_lower <= mergeされている
│   ├── testfile_overlay
│   └── testfile_upper
├── upper
│   ├── testfile_overlay
│   └── testfile_upper
└── work
    └── work [error opening dir]

5 directories, 6 files
```

下層側に変更をしてみます。

```
vagrant@ubuntu-bionic:~$ echo "test" > merged/testfile_lower  # 下層側のファイル
vagrant@ubuntu-bionic:~$ cat lower/testfile_lower <= 変更はされてない
vagrant@ubuntu-bionic:~$ cat upper/testfile_lower
test
vagrant@ubuntu-bionic:~$ tree ./
./
├── lower
│   └── testfile_lower
├── merged
│   ├── testfile_lower
│   ├── testfile_overlay
│   └── testfile_upper
├── upper
│   ├── testfile_lower <= new!
│   ├── testfile_overlay
│   └── testfile_upper
└── work
    └── work [error opening dir]

5 directories, 7 files
```

下層側に変更が加わると、下層自体に変更はなく、上層にその変更が反映されます。

変更は常にupperdirオプションで指定した上層側ディレクトリ以下になされることがわかります。

上層への変更はどうでしょう？

```
vagrant@ubuntu-bionic:~$ echo "test" > merged/testfile_upper
vagrant@ubuntu-bionic:~$ cat merged/testfile_upper
test
vagrant@ubuntu-bionic:~$ tree ./
./
├── lower
│   └── testfile_lower
├── merged
│   ├── testfile_lower
│   ├── testfile_overlay
│   └── testfile_upper
├── upper
│   ├── testfile_lower
│   ├── testfile_overlay
│   └── testfile_upper
└── work
    └── work [error opening dir]

5 directories, 7 files
vagrant@ubuntu-bionic:~$ cat upper/testfile_upper
test
```

上層にあるファイルに変更を加えると，そのまま上層にあるファイルが変更されています。

上層側のファイルを消してみます。

```
vagrant@ubuntu-bionic:~$ rm merged/testfile_overlay
vagrant@ubuntu-bionic:~$ tree ./
./
├── lower
│   └── testfile_lower
├── merged
│   ├── testfile_lower
│   └── testfile_upper
├── upper
│   ├── testfile_lower
│   └── testfile_upper
└── work
    └── work [error opening dir]

5 directories, 5 files
```

先ほどまで上層側に存在していたtestdir_overlayとtestfile_overlayがマウントされたディレクトリでも上層側のディレクトリでも消去されています。

下層側のファイルを削除してみます。

```
vagrant@ubuntu-bionic:~$ rm merged/testfile_lower
vagrant@ubuntu-bionic:~$ tree ./
./
├── lower
│   └── testfile_lower
├── merged
│   └── testfile_upper
├── upper
│   ├── testfile_lower
│   └── testfile_upper
└── work
    └── work [error opening dir]

5 directories, 4 files
```

下層にも上層にも存在していたtestfile_lowerファイルを消してみました。
すると，下層側には *そのままの状態* でファイルが残っています。 => Dockerで上層側のファイルを消しても容量が減らないのはここですね

上層側のファイルは、特殊ファイルになっていました。

```
vagrant@ubuntu-bionic:~$ ls -al upper/
total 12
drwxrwxr-x 2 vagrant vagrant 4096 Nov 17 02:34 .
drwxr-xr-x 9 vagrant vagrant 4096 Nov 17 02:08 ..
c--------- 1 root    root    0, 0 Nov 17 02:34 testfile_lower <= キャラクタ型デバイスファイル（特殊ファイル）になっている
-rw-rw-r-- 1 vagrant vagrant    5 Nov 17 02:27 testfile_upper
```

この状態でumountしてみます。

```
vagrant@ubuntu-bionic:~$ sudo umount overlay
vagrant@ubuntu-bionic:~$ tree ./
./
├── lower
│   └── testfile_lower
├── merged
├── upper
│   ├── testfile_lower
│   └── testfile_upper
└── work
    └── work [error opening dir]

5 directories, 3 files
```

## 参考資料
* 実際に触るときに参考にした資料
  * https://gihyo.jp/admin/serial/01/linux_containers/0018?page=1
  * https://eh-career.com/engineerhub/entry/2019/02/05/103000#Copy-On-WriteCOW%E3%83%95%E3%82%A1%E3%82%A4%E3%83%AB%E3%82%B7%E3%82%B9%E3%83%86%E3%83%A0
  * https://tenforward.hatenablog.com/entry/2017/08/16/193831
* https://qiita.com/DQNEO/items/ee0caf80b056487cb762
* https://docs.docker.com/storage/storagedriver/overlayfs-driver/
* https://www.school.ctc-g.co.jp/columns/nakai/nakai69.html
* https://sites.google.com/site/kandamotohiro/linux/overlayfs-txt
