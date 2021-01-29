# Dockerのコンポーネント
Dockerといってもいろいろなコンポーネントが組み合わさって動作しています。

![](https://image.slidesharecdn.com/what-is-docker-doing-191213003129/95/docker-43-638.jpg?cb=1576197252)

参照 : https://www2.slideshare.net/zembutsu/what-isdockerdoing

## Dockerクライアント
Dockerクライアントはユーザーとのやり取りのインターフェースになります。

Dockerクライアントはコマンドを受け取るとDockerエンジンにその内容を Dockerエンジンが提供するRESTful APIに投げます。

Docker CLIはここに含まれます。

> The Docker Engine API is a RESTful API accessed by an HTTP client such as wget or curl, or the HTTP library which is part of most modern programming languages.

https://docs.docker.com/engine/api/

## Dockerエンジン(dockerd)
イメージの管理、ファイルシステムの管理、ネットワークの管理を行います。

これがDockerの本体とも言えるかと思います。

現在はmobyとよばれています。

[Moby」ベースとなったオープンソース版Dockerの最新状況](https://knowledge.sakura.ad.jp/11068/)

[2018年のDocker・Mob](https://www2.slideshare.net/AkihiroSuda/japan-container-days-2018dockermoby)

イメージの管理はcontainerdに移そうとしたが結局頓挫している？？

https://github.com/moby/moby/issues/38043

## コンテナランタイム(containerd)
コンテナのライフサイクルを管理します。

containerdはCNCFにホストされているコンポーネントになります。

https://containerd.io/

https://github.com/containerd/containerd

![](https://containerd.io/img/architecture.png)

コンテナのランチアムの役割は2つあります。

### 高レベルランタイム( containerd cri-o rkt)
DockerやKubernetesと通信するコンポーネント

CRI(Container Runtime Interface)ランタイムとも呼ばれることがあります。

そもそもはkubernetesのkubeletとコンテナランタイム間の通信のインターフェースになります。

https://github.com/kubernetes/kubernetes/blob/242a97307b34076d5d8f5bbeb154fa4d97c9ef1d/docs/devel/container-runtime-interface.md

CRIに準拠したコンテナランタイムであれば、kubernetesのコンテナランタイムとして使うことができます。

containerdはCRI APIも用意していますが、Dockerとの通信にはCRI APIは使いません。

高レベルランタイムはイメージの管理をします。

また、受け取った命令を低レベルランタイムに受け渡して、コンテナの生成を依頼します。

### 低レベルランタイム( runc runsc(gVisor) kata-containers )
高レベルランタイムの命令を受けてコンテナを生成します。

高レベルランタイムとのインターフェースはOCIのruntime-specとして標準化されています。

https://github.com/opencontainers/runtime-spec

この低レベルランタイムが実際にLinux Namespaceやcgroupを使って環境の隔離を行います。

パフォーマンス向上やセキュリティの担保のために様々な低レベルランタイムがあります。

## 参考資料
* コンテナランタイムの動向を整理してみた件
  * https://qiita.com/mamomamo/items/ed5db2ab1555078f8a24
* コンテナランタイムの仕組みと、Firecracker、gVisor、Unikernelが注目されている理由。 Container Runtime Meetup #2
  * https://www.publickey1.jp/blog/20/firecrackergvisorunikernel_container_runtime_meetup_2.html
  * https://speakerdeck.com/inductor/container-runtime-101?slide=15
* コンテナユーザなら誰もが使っているランタイム「runc」を俯瞰する[Container Runtime Meetup #1発表レポート]
  * https://medium.com/nttlabs/runc-overview-263b83164c98
* Dockerだけじゃないコンテナ runtime 徹底比較
  * https://speakerdeck.com/makocchi/about-container-runtimes-japan-container-days-v18-dot-04
* 今話題のいろいろなコンテナランタイムを比較してみた
  * https://www2.slideshare.net/KoheiTokunaga/ss-123664087
* https://kubernetes.io/blog/2016/12/container-runtime-interface-cri-in-kubernetes/
* https://www.publickey1.jp/blog/17/kubernetesdockerkubernetescri-o10.html

