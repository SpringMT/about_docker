# Dockerの概要
ここにきてようやくDockerの話をしていきます。

## Dockerの歴史
https://www.publickey1.jp/blog/17/post_265.html

ここにだいたい書いてありました。

## Dockerの構成要素
Dockerの構成要素は主に3つ(Build Ship Run)に分かれます。

Dockerは "Docker - Build, Ship, and Run any App, Anywhere." って謳っていましたが今は書いてないかも、、

### Dockerイメージ
Docker イメージとは、読み込み専用（read-only）なテンプレートです。

例:  Ubuntu オペレーティング・システムの上に Apache とウェブ・アプリケーションが含まれる

イメージは Docker コンテナの作成時に使われます。

Docker は新しいイメージの構築や、既存イメージの更新を行います。

又は、他の人が既に作成した Docker イメージをダウンロードします。

Docker イメージとは Docker における *構築（build）* コンポーネントです。

### Dockerレジストリ
Docker レジストリはDockerイメージを保持します。

パブリックもしくはプライベートに保管されているイメージのアップロードやダウンロードを行えます。

パブリックな Docker レジストリとして Docker Hub が提供されています。

イメージを自分自身で作れるだけでなく、他人が作成したイメージも利用できます。

Docker レジストリとは Docker における *配布（distribution or ship）* コンポーネントです。

### Dockerコンテナ
Docker コンテナにはアプリケーションの実行に必要な全てが含まれています。

各コンテナは Docker イメージによって作られます。

Docker コンテナは実行・開始・停止・移動・削除できます。

各コンテナは分離されており、安全なアプリケーションのプラットフォームです。

Docker コンテナとは Docker における *実行（run）* コンポーネントです。



## 標準化の話
Linux Foundationプロジェクトの1つにコンテナの標準化をするためにOCI(Open Container Initiative)があります。

https://opencontainers.org/

OCIはコンテナイメージとコンテナランタイムなどの標準化を行っています。

今みていると、OCI v1.0.1出て以降そこまで活発に動いているわけではなさそうです。

### コンテナイメージフォーマットの標準化
OCI Image Format Specification

https://github.com/opencontainers/image-spec

このimage formatを元に作ったコンテナイメージはOCI Runtime Specで定義されたコンテナランタイムで動かすことができます。

https://blog.unasuke.com/2018/read-oci-image-spec-v101/

### コンテナランタイムの標準化
OCI Runtime Specification

https://github.com/opencontainers/runtime-spec

現在の主要なコンテナランタイムのほとんどが準拠しています。

主な実装としては下記があります。

名前 | 説明 | 参照
-|-|-
runc | Dockerの一部として作らてきたランタイム <br> 本番環境で使うことは少なくなってきたような気がする | https://github.com/opencontainers/runc
runsc(gVisor) | Googleが作ったランタイム |	https://gvisor.dev/docs/ <br> [20分でわかるgVisor入門](https://www2.slideshare.net/uzy_exe/201805gvisorintroduciton)
kata containers | Open Stack Foundationのプロジェクト <br> AWSのFirecrackerをサポートしているので注目されていますね | https://katacontainers.io/ <br> https://www.publickey1.jp/blog/19/kata_containerawsvmfirecracker.html

## 参考資料
* http://docs.docker.jp/engine/introduction/understanding-docker.html
* コンテナ未経験新人が学ぶコンテナ技術入門
  * https://www2.slideshare.net/KoheiTokunaga/ss-122754942
* コンテナの標準仕様について調査してみた件
  * https://qiita.com/mamomamo/items/448a8edf6d4ccfc22bbd
* OCI Runtime Specification を読む
  * https://udzura.hatenablog.jp/entry/2016/08/02/155913
* DockerのデフォルトランタイムをrunCからKata containers + Firecrackerに変えるのが簡単すぎてビビった話
  * https://blog.inductor.me/entry/2020/07/24/012945
  
