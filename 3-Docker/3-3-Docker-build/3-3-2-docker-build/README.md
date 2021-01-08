# docker buildの流れ

## 流れ

1. docker build . コマンドを打つ
2. Dockerfileとdockerignoreを読み込み
    * Dockerfileのparse
    * https://github.com/moby/buildkit/blob/dec4da0920fed46125ad94d582dc127dcab8b6e4/frontend/dockerfile/parser/parser.go#L250
3. ビルドコンテキストで指定されたパス配下をdockerのファイルシステムにcopyする
    * dockerignoreで指定されたファイル群は除かれる
    * ビルドコンテキストに大量にファイルあるとここが重くなる
4. Dockerfileで指定された命令が実行される

## ソースの流れ
CLIからdocker buildが叩かれる

https://github.com/moby/moby/blob/46cdcd206c56172b95ba5c77b827a722dab426c5/client/image_build.go

ここからdockerdのエンドポイントを叩いてdockerdの処理が走る

https://github.com/moby/moby/blob/46cdcd206c56172b95ba5c77b827a722dab426c5/api/server/router/build/build.go

https://github.com/moby/moby/blob/46cdcd206c56172b95ba5c77b827a722dab426c5/api/server/router/build/build_routes.go#L217

Buildメソッドはdockerdを立ち上げるときに決まる

https://github.com/moby/moby/blob/816fbcd306274b9561c62ae076bdff71062ebe85/cmd/dockerd/daemon.go 

で実体は下記

https://github.com/moby/moby/blob/816fbcd306274b9561c62ae076bdff71062ebe85/builder/builder-next/builder.go#L203

Solveの実体はbuildkit

https://github.com/moby/buildkit/blob/master/control/control.go#L216

dockerfileもある

https://github.com/moby/moby/tree/816fbcd306274b9561c62ae076bdff71062ebe85/builder/dockerfile


## キャッシュ判定
https://spring-mt.hatenablog.com/entry/2020/12/16/032810
