# Dockerイメージの内容
まずはDockerイメージがどういうものかを見てみます。

nginxのDockerイメージを手元に持ってきます。

```
vagrant@ubuntu-bionic:~$ CID=$(sudo docker container create nginx)
Unable to find image 'nginx:latest' locally
latest: Pulling from library/nginx
852e50cd189d: Pulling fs layer
a29b129f4109: Pulling fs layer
b3ddf1fa5595: Pulling fs layer
c5df295936d3: Pulling fs layer
232bf38931fc: Pulling fs layer
c5df295936d3: Waiting
232bf38931fc: Waiting
b3ddf1fa5595: Verifying Checksum
b3ddf1fa5595: Download complete
c5df295936d3: Verifying Checksum
c5df295936d3: Download complete
232bf38931fc: Verifying Checksum
232bf38931fc: Download complete
a29b129f4109: Verifying Checksum
a29b129f4109: Download complete
852e50cd189d: Verifying Checksum
852e50cd189d: Download complete
852e50cd189d: Pull complete
a29b129f4109: Pull complete
b3ddf1fa5595: Pull complete
c5df295936d3: Pull complete
232bf38931fc: Pull complete
Digest: sha256:c3a1592d2b6d275bef4087573355827b200b00ffc2d9849890a4f3aa2128c4ae
Status: Downloaded newer image for nginx:latest
```

## docker export
ではまずexportしてみます。

exportはコンテナ上に存在するファイルシステムをそのままtarでまとめて出力します。

メタ情報は無視されます。

https://docs.docker.com/engine/reference/commandline/export/

```
vagrant@ubuntu-bionic:~/image$ tree -L 2
.
├── bin
│   ├── bash
~
│   ├── zless
│   ├── zmore
│   └── znew
├── boot
├── dev
│   ├── console
│   ├── pts
│   └── shm
├── docker-entrypoint.d
│   ├── 10-listen-on-ipv6-by-default.sh
│   └── 20-envsubst-on-templates.sh
├── docker-entrypoint.sh
├── docker_image.tar
├── etc
│   ├── adduser.conf
│   ├── alternatives
│   ├── apt
~
│   ├── ucf.conf
│   ├── update-motd.d
│   └── xattr.conf
├── home
├── lib
│   ├── init
│   ├── lsb
│   ├── systemd
│   ├── terminfo
│   ├── udev
│   └── x86_64-linux-gnu
├── lib64
│   └── ld-linux-x86-64.so.2 -> /lib/x86_64-linux-gnu/ld-2.28.so
├── media
├── mnt
├── opt
├── proc
├── root
├── run
│   ├── lock
│   └── utmp
├── sbin
│   ├── agetty
│   ├── badblocks
~
│   ├── unix_update
│   ├── wipefs
│   └── zramctl
├── srv
├── sys
├── tmp
├── usr
│   ├── bin
│   ├── games
│   ├── include
│   ├── lib
│   ├── local
│   ├── sbin
│   ├── share
│   └── src
└── var
├── backups
├── cache
├── lib
├── local
├── lock -> /run/lock
├── log
├── mail
├── opt
├── run -> /run
├── spool
└── tmp

80 directories, 187 files
```

そのままですね

## docker save
saveを使うとDockerイメージとして保存されます。

レイヤーやタグといったメタ情報含めてコンテナをtarでまとめます。

https://docs.docker.com/engine/reference/commandline/save/

Dockerfileは下記になります。

https://github.com/nginxinc/docker-nginx/blob/deff8fbe9d3e8613de110265aa932d84d1827acf/mainline/buster/Dockerfile

Dockerイメージの仕様は下記にあります。

https://github.com/moby/moby/blob/master/image/spec/v1.2.md

大事な部分を抜き出して説明します。

### Image JSON
作成日、作者、親イメージの ID などのイメージに関する基本的な情報と、エントリポイント、デフォルト引数、CPU/メモリ共有、ネットワーク、ボリュームなどの実行/実行時の設定が記述されているものです。

このJSONは、イメージが使用する各レイヤーの暗号化ハッシュを参照し、また、それらのレイヤーの履歴情報を含んでいます。

### Image Filesystem Changeset
Dockerイメージはレイヤーで構成されています。

各レイヤーはファイルシステムの変更の集合です。(レイヤーには環境変数やデフォルト引数などの設定メタデータはありません。)

各レイヤーには、親レイヤーに関連して追加・変更・削除されたファイルのアーカイブがあります。

Image Filesystem Changesetは、OverlayFSのようなユニオンファイルシステムを使うことで、各レイヤーを一つのまとまりのあるファイルシステムであるかのように見せることができます。

上記の2つを結びつけるのがmanifest.jsonとなります。

### manifest.json
manifest.jsonにはImage JSONの場所とImage Filesystem ChangesetのLayerが指定されています。

* Config : Image JSONの場所
* RepoTags : このDokcerイメージの置き場所
* Layers : Filesystem Changeset のtarの場所

となります。

## 中を見てみる

```
vagrant@ubuntu-bionic:~$ sudo docker save nginx:latest --output docker_image1.tar
vagrant@ubuntu-bionic:~/image1$ sudo tar -xf docker_image1.tar 
vagrant@ubuntu-bionic:~/image1$ tree -L 2
.
├── 03bb461277c480dcdba1979007c5c1922c97e1b783e23d0377c590b6ef93ae31
│   ├── VERSION
│   ├── json <= 後方互換性のためだけにあるファイルなのでスルーしておｋ
│   └── layer.tar <= これがlayerの実態
├── 18c0be8bd55e1222522249d663065e81a6ca4d6e3b6760ad89808a22f8a04c0a
│   ├── VERSION
│   ├── json
│   └── layer.tar
├── 46c8305194c21f9c7350559b3d6d904eb509aa107f47680db668b26e188cee77
│   ├── VERSION
│   ├── json
│   └── layer.tar
├── 5a1e3f520146c699c41b1a726d51867e2327c41d3a5d56169843796511848eaf
│   ├── VERSION
│   ├── json
│   └── layer.tar
├── 9c228d03d114647df0dd90f9b45ca4ed3b973ea6b114e1e514ff1ce041576c18
│   ├── VERSION
│   ├── json
│   └── layer.tar
├── daee903b4e436178418e41d8dc223b73632144847e5fe81d061296e667f16ef2.json
├── manifest.json
└── repositories
```

64 文字の 16 進数の名前が付けられているディレクトリに各レイヤーの情報が含まれています。

これらのディレクトリには後方互換性ために必要な json という名前のファイルがあったりしますが、本質的なレイヤーの情報としては `layer.tar` のみとなります。

manifest.jsonはこんな感じです。

```
vagrant@ubuntu-bionic:~/image1$ cat manifest.json | jq
[
  {
    "Config": "daee903b4e436178418e41d8dc223b73632144847e5fe81d061296e667f16ef2.json", <= Image JSONの場所
    "RepoTags": [
      "nginx:latest"
    ],
    "Layers": [
      "5a1e3f520146c699c41b1a726d51867e2327c41d3a5d56169843796511848eaf/layer.tar", 
      "9c228d03d114647df0dd90f9b45ca4ed3b973ea6b114e1e514ff1ce041576c18/layer.tar",
      "03bb461277c480dcdba1979007c5c1922c97e1b783e23d0377c590b6ef93ae31/layer.tar",
      "46c8305194c21f9c7350559b3d6d904eb509aa107f47680db668b26e188cee77/layer.tar",
      "18c0be8bd55e1222522249d663065e81a6ca4d6e3b6760ad89808a22f8a04c0a/layer.tar"
    ]
  }
]
```

そこまで複雑ではないですね。

Configで指定された、Image JSONを見てみます。

```
{
  "architecture": "amd64",
  "config": {
    "Hostname": "",
    "Domainname": "",
    "User": "",
    "AttachStdin": false,
    "AttachStdout": false,
    "AttachStderr": false,
    "ExposedPorts": {
      "80/tcp": {}
    },
    "Tty": false,
    "OpenStdin": false,
    "StdinOnce": false,
    "Env": [
      "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
      "NGINX_VERSION=1.19.4",
      "NJS_VERSION=0.4.4",
      "PKG_RELEASE=1~buster"
    ],
    "Cmd": [
      "nginx",
      "-g",
      "daemon off;"
    ],
    "Image": "sha256:4b1d766550263a0d696c77e331fa138d3cbdf5888bea49996f6763e9551720f8",
    "Volumes": null,
    "WorkingDir": "",
    "Entrypoint": [
      "/docker-entrypoint.sh"
    ],
    "OnBuild": null,
    "Labels": {
      "maintainer": "NGINX Docker Maintainers <docker-maint@nginx.com>"
    },
    "StopSignal": "SIGTERM"
  },
  "container": "7e8ca989e54001b9955974e36eb6d679ab4fe015066014645ef927fe88c326ec",
  "container_config": {
    "Hostname": "7e8ca989e540",
    "Domainname": "",
    "User": "",
    "AttachStdin": false,
    "AttachStdout": false,
    "AttachStderr": false,
    "ExposedPorts": {
      "80/tcp": {}
    },
    "Tty": false,
    "OpenStdin": false,
    "StdinOnce": false,
    "Env": [
      "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
      "NGINX_VERSION=1.19.4",
      "NJS_VERSION=0.4.4",
      "PKG_RELEASE=1~buster"
    ],
    "Cmd": [
      "/bin/sh",
      "-c",
      "#(nop) ",
      "CMD [\"nginx\" \"-g\" \"daemon off;\"]"
    ],
    "Image": "sha256:4b1d766550263a0d696c77e331fa138d3cbdf5888bea49996f6763e9551720f8",
    "Volumes": null,
    "WorkingDir": "",
    "Entrypoint": [
      "/docker-entrypoint.sh"
    ],
    "OnBuild": null,
    "Labels": {
      "maintainer": "NGINX Docker Maintainers <docker-maint@nginx.com>"
    },
    "StopSignal": "SIGTERM"
  },
  "created": "2020-11-18T07:48:35.319575714Z",
  "docker_version": "19.03.12",
  "history": [
    {
      "created": "2020-11-17T20:21:17.570073346Z",
      "created_by": "/bin/sh -c #(nop) ADD file:d2abb0e4e7ac1773741f51f57d3a0b8ffc7907348842d773f8c341ba17f856d5 in / "
    },
    {
      "created": "2020-11-17T20:21:17.865210281Z",
      "created_by": "/bin/sh -c #(nop)  CMD [\"bash\"]",
      "empty_layer": true
    },
    {
      "created": "2020-11-18T07:48:00.110721952Z",
      "created_by": "/bin/sh -c #(nop)  LABEL maintainer=NGINX Docker Maintainers <docker-maint@nginx.com>",
      "empty_layer": true
    },
    {
      "created": "2020-11-18T07:48:00.367924531Z",
      "created_by": "/bin/sh -c #(nop)  ENV NGINX_VERSION=1.19.4",
      "empty_layer": true
    },
    {
      "created": "2020-11-18T07:48:00.640093202Z",
      "created_by": "/bin/sh -c #(nop)  ENV NJS_VERSION=0.4.4",
      "empty_layer": true
    },
    {
      "created": "2020-11-18T07:48:00.915776996Z",
      "created_by": "/bin/sh -c #(nop)  ENV PKG_RELEASE=1~buster",
      "empty_layer": true
    },
    {
      "created": "2020-11-18T07:48:32.578151013Z",
      "created_by": "/bin/sh -c set -x     && addgroup --system --gid 101 nginx     && adduser --system --disabled-login --ingroup nginx --no-create-home --home /nonexistent --gecos \"nginx user\" --shell /bin/false --uid 101 nginx     && apt-get update     && apt-get install --no-install-recommends --no-install-suggests -y gnupg1 ca-certificates     &&     NGINX_GPGKEY=573BFD6B3D8FBC641079A6ABABF5BD827BD9BF62;     found='';     for server in         ha.pool.sks-keyservers.net         hkp://keyserver.ubuntu.com:80         hkp://p80.pool.sks-keyservers.net:80         pgp.mit.edu     ; do         echo \"Fetching GPG key $NGINX_GPGKEY from $server\";         apt-key adv --keyserver \"$server\" --keyserver-options timeout=10 --recv-keys \"$NGINX_GPGKEY\" && found=yes && break;     done;     test -z \"$found\" && echo >&2 \"error: failed to fetch GPG key $NGINX_GPGKEY\" && exit 1;     apt-get remove --purge --auto-remove -y gnupg1 && rm -rf /var/lib/apt/lists/*     && dpkgArch=\"$(dpkg --print-architecture)\"     && nginxPackages=\"         nginx=${NGINX_VERSION}-${PKG_RELEASE}         nginx-module-xslt=${NGINX_VERSION}-${PKG_RELEASE}         nginx-module-geoip=${NGINX_VERSION}-${PKG_RELEASE}         nginx-module-image-filter=${NGINX_VERSION}-${PKG_RELEASE}         nginx-module-njs=${NGINX_VERSION}+${NJS_VERSION}-${PKG_RELEASE}     \"     && case \"$dpkgArch\" in         amd64|i386)             echo \"deb https://nginx.org/packages/mainline/debian/ buster nginx\" >> /etc/apt/sources.list.d/nginx.list             && apt-get update             ;;         *)             echo \"deb-src https://nginx.org/packages/mainline/debian/ buster nginx\" >> /etc/apt/sources.list.d/nginx.list                         && tempDir=\"$(mktemp -d)\"             && chmod 777 \"$tempDir\"                         && savedAptMark=\"$(apt-mark showmanual)\"                         && apt-get update             && apt-get build-dep -y $nginxPackages             && (                 cd \"$tempDir\"                 && DEB_BUILD_OPTIONS=\"nocheck parallel=$(nproc)\"                     apt-get source --compile $nginxPackages             )                         && apt-mark showmanual | xargs apt-mark auto > /dev/null             && { [ -z \"$savedAptMark\" ] || apt-mark manual $savedAptMark; }                         && ls -lAFh \"$tempDir\"             && ( cd \"$tempDir\" && dpkg-scanpackages . > Packages )             && grep '^Package: ' \"$tempDir/Packages\"             && echo \"deb [ trusted=yes ] file://$tempDir ./\" > /etc/apt/sources.list.d/temp.list             && apt-get -o Acquire::GzipIndexes=false update             ;;     esac         && apt-get install --no-install-recommends --no-install-suggests -y                         $nginxPackages                         gettext-base                         curl     && apt-get remove --purge --auto-remove -y && rm -rf /var/lib/apt/lists/* /etc/apt/sources.list.d/nginx.list         && if [ -n \"$tempDir\" ]; then         apt-get purge -y --auto-remove         && rm -rf \"$tempDir\" /etc/apt/sources.list.d/temp.list;     fi     && ln -sf /dev/stdout /var/log/nginx/access.log     && ln -sf /dev/stderr /var/log/nginx/error.log     && mkdir /docker-entrypoint.d"
    },
    {
      "created": "2020-11-18T07:48:33.126543316Z",
      "created_by": "/bin/sh -c #(nop) COPY file:e7e183879c35719c18aa7f733651029fbcc55f5d8c22a877ae199b389425789e in / "
    },
    {
      "created": "2020-11-18T07:48:33.60445567Z",
      "created_by": "/bin/sh -c #(nop) COPY file:13577a83b18ff90a0f97a15cd6380790a5f5288c651fa08708ff64d3f1595861 in /docker-entrypoint.d "
    },
    {
      "created": "2020-11-18T07:48:33.991896505Z",
      "created_by": "/bin/sh -c #(nop) COPY file:0fd5fca330dcd6a7de297435e32af634f29f7132ed0550d342cad9fd20158258 in /docker-entrypoint.d "
    },
    {
      "created": "2020-11-18T07:48:34.337710288Z",
      "created_by": "/bin/sh -c #(nop)  ENTRYPOINT [\"/docker-entrypoint.sh\"]",
      "empty_layer": true
    },
    {
      "created": "2020-11-18T07:48:34.683272118Z",
      "created_by": "/bin/sh -c #(nop)  EXPOSE 80",
      "empty_layer": true
    },
    {
      "created": "2020-11-18T07:48:35.012156675Z",
      "created_by": "/bin/sh -c #(nop)  STOPSIGNAL SIGTERM",
      "empty_layer": true
    },
    {
      "created": "2020-11-18T07:48:35.319575714Z",
      "created_by": "/bin/sh -c #(nop)  CMD [\"nginx\" \"-g\" \"daemon off;\"]",
      "empty_layer": true
    }
  ],
  "os": "linux",
  "rootfs": {
    "type": "layers",
    "diff_ids": [
      "sha256:f5600c6330da7bb112776ba067a32a9c20842d6ecc8ee3289f1a713b644092f8",
      "sha256:32048dd980c7efcd45a1b76aa93dafa21d8724a8e62841d7f0731808ee220186",
      "sha256:e3a971c30b120662d8171a70b19deb6b41b8fcd174f1197b1fc2090155c4ae66",
      "sha256:5887d03dfc3db0dc701d01d75591873a2514cf744e0a89bdf6be531cf8c0c664",
      "sha256:b9e73ac5343ec2ad6fef829b05505e8a393b17541f3eae96e9e88b0a72124605"
    ]
  }
}
```
Dockerfileに書かれている内容とdocker build時の情報が含まれています。

historyを見てると、Dockerfileに書かれた命令文が実行されている様子が見えます。

empty_layer: trueになっているhistoryにはlayerが生成されないので、それ以外のものを数えてみるとちょうど5つになります。

rootfsにはDockerイメージが使用するレイヤーの内容のアドレスを示しています。

diff_idsにはレイヤーのコンテンツハッシュ (DiffIDs) の配列です。

下から上へと順に並べていきます。

**ここが重要でこれがないとユニオンできないです!**

実際にsha256sumを取ってみると値が一致します。

```
vagrant@ubuntu-bionic:~/image1$ sha256sum */layer.tar 
e3a971c30b120662d8171a70b19deb6b41b8fcd174f1197b1fc2090155c4ae66  03bb461277c480dcdba1979007c5c1922c97e1b783e23d0377c590b6ef93ae31/layer.tar
b9e73ac5343ec2ad6fef829b05505e8a393b17541f3eae96e9e88b0a72124605  18c0be8bd55e1222522249d663065e81a6ca4d6e3b6760ad89808a22f8a04c0a/layer.tar
5887d03dfc3db0dc701d01d75591873a2514cf744e0a89bdf6be531cf8c0c664  46c8305194c21f9c7350559b3d6d904eb509aa107f47680db668b26e188cee77/layer.tar
f5600c6330da7bb112776ba067a32a9c20842d6ecc8ee3289f1a713b644092f8  5a1e3f520146c699c41b1a726d51867e2327c41d3a5d56169843796511848eaf/layer.tar
32048dd980c7efcd45a1b76aa93dafa21d8724a8e62841d7f0731808ee220186  9c228d03d114647df0dd90f9b45ca4ed3b973ea6b114e1e514ff1ce041576c18/layer.tar
```
CMDの定義などもあるので、このレイヤーをうまく合わせてCMDを実行すればDockerコンテナとして動作可能です。(多分)

## 参考資料
* https://ja.wikipedia.org/wiki/Tar
* https://stackoverflow.com/questions/36216220/what-is-different-of-config-and-containerconfig-of-docker-inspect
* https://github.com/moby/moby/blob/master/image/spec/v1.2.md
* https://blog.unasuke.com/2018/read-oci-image-spec-v101/
