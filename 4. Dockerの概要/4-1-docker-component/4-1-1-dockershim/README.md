# dockershim問題
これはコンテナを作成、動作させるときにの話です

DockerはCRIに対応していません。

kubernetesのkubelet(クラスター内の各ノードで実行されるエージェントです。各コンテナがPodで実行されていることを保証します。)がCRIインターフェースしかサポートしていません。

そこで、kuberletとDocker(dockerd)との間にCRI → Dockerをする dockershimをおいていました。

https://github.com/dims/cri-dockerd

![](https://www.publickey1.jp/2018/containerd1101.gif)

これが下記のようにできるようになりました。

![](https://www.publickey1.jp/2018/containerd1103.gif)

https://kubernetes.io/blog/2018/05/24/kubernetes-containerd-integration-goes-ga/

https://www.publickey1.jp/blog/18/kubernetescontainerd_11cridocker.html

で、これを踏まえてdockershimを廃止したのが最近の出来事です。

https://github.com/kubernetes/kubernetes/pull/94624

で混乱したわけです。

https://kubernetes.io/blog/2020/12/02/dont-panic-kubernetes-and-docker/

https://blog.inductor.me/entry/2020/12/03/061329

https://thinkit.co.jp/article/18024

kubernetesでdockershimがなくなっても、ほとんどの場合は何も影響はありません。

Docker imageはcontainerdでも使えます。

影響があるとすると、自分でkubernetesクラスタを運用している場合で、dockdershimに依存している場合です。

## GKE
### cos の場合
dockershim的なものはそもそもなかった

root     3674309  0.0  0.0 108716  5916 ?        Sl   Dec14   0:06  \_ containerd-shim -namespace moby -workdir /var/lib/containerd/io.containerd.runtime.v1.linux/moby/841e46c6c427462d464e41473385405c796c050348010444a3d70307052f9ae7 -address /run/containerd/containerd.sock -containerd-binary /usr/bin/containerd -runtime-root /var/run/docker/runtime-runc
root     3674325  0.0  0.0   4636   788 ?        Ss   Dec14   0:00  |   \_ /bin/sh -c ruby -v bundle exec ruby shared/scripts/setup.rb bundle exec unicorn_rails -c config/unicorn/production.rb 
root     3674343  0.0  0.8 373360 125524 ?       S    Dec14   0:05  |       \_ unicorn_rails master -c config/unicorn/production.rb
root     3674523  0.0  0.9 434052 147164 ?       Sl   Dec14   1:12  |           \_ unicorn_rails worker[0] -c config/unicorn/production.rb
root     3674524  0.0  0.9 433984 147896 ?       Sl   Dec14   1:09  |           \_ unicorn_rails worker[1] -c config/unicorn/production.rb

### cos_containerdの場合
https://cloud.google.com/kubernetes-engine/docs/concepts/using-containerd

root        3596  0.0  0.1 111712 10636 ?        Sl   Dec15   0:36 /usr/bin/containerd-shim-runc-v1 -namespace k8s.io -id 45b8ad4092505c461df77804ba2ce23a0e1b7724cb80eb21000be548c6a2562b -address /run/containerd/containerd.sock
root        3617  0.0  0.0   4636   836 ?        Ss   Dec15   0:00  \_ /bin/sh -c ruby -v bundle exec ruby shared/scripts/setup.rb bundle exec unicorn_rails -c config/unicorn/production.rb 
root        3648  0.0  1.8 438352 152424 ?       S    Dec15   0:07      \_ unicorn_rails master -c config/unicorn/production.rb
root        3854  0.0  3.4 594496 279896 ?       Sl   Dec15   0:07          \_ unicorn_rails worker[0] -c config/unicorn/production.rb
root        3855  0.0  3.4 596480 285936 ?       Sl   Dec15   0:07          \_ unicorn_rails worker[1] -c config/unicorn/production.rb


## 参考資料
* https://kubernetes.io/ja/docs/concepts/overview/components/#kubelet
* https://kubernetes.io/blog/2018/05/24/kubernetes-containerd-integration-goes-ga/
* https://github.com/containerd/containerd/tree/master/cmd ここにcontainerd-shimなどがおいてある
