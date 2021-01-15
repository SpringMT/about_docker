# 仮想化技術
## なぜ生まれたのか？
基本的な目的はハードウェアの利用効率の向上となります。

仮想化技術によって、物理的なサーバーに論理的なサーバーを複数用意し、負荷の許す限り詰め込むことができるようになります。

これはCPUの性能向上などにより小さな物理サーバーであってもリソースを使い切れないような状況が生まれやすくなったためです。

[24時間/365日サーバー/インフラを支える技術](https://amzn.to/3hFELkA) (p316 2008年くらいの話かと)の中で、当時のはてなではXenを使った仮想化を行っており、物理サーバーが350台ほどだったそうですが、仮想的な論理サーバーはその2割り増しの数になったそうです。

## 仮想化するための実装
主にハイパーバイザ型と呼ばれるものを主に列挙します。

### VMWare
https://www.vmware.com/jp.html

### Xen
https://xenproject.org/

AWSのEC2は最初はXenで仮想化されていた(はず)

XenのセキュリティパッチによりEC2が落ちるとか普通にあった記憶

現在は Nitoro Systemに置き換わっています。 https://aws.amazon.com/jp/ec2/nitro/

### KVM
KVMは Kernel-based Virtual Machineの名前の通り、Linux Kernel自体をハイパーバイザとする仕組みです。

https://www.atmarkit.co.jp/ait/articles/0903/12/news120.html

https://www.redhat.com/ja/topics/virtualization/what-is-KVM

GCEはKVMを使っていると思われます

* https://cloud.google.com/blog/ja/products/gcp/7-ways-we-harden-our-kvm-hypervisor-at-google-cloud-security-in-plaintext

### Virtual Box
https://www.virtualbox.org/


## ハイパーバイザ型仮想化技術の内容
ハイパーバイザによって仮想的なハードウェア(仮想マシン)が作成されます。

ハイパーバイザ : https://ja.wikipedia.org/wiki/%E3%83%8F%E3%82%A4%E3%83%91%E3%83%BC%E3%83%90%E3%82%A4%E3%82%B6

![](https://image.itmedia.co.jp/ait/articles/0903/12/r12fig01.jpg)

https://www.atmarkit.co.jp/ait/articles/0903/12/news120.html より抜粋

> 上記のホストOS型はWikipediaではType 2と呼ばれているものになります。Virtual Boxなどがこれにあたります。

仮想マシンに、OS、実行するアプリケーション、ライブラリをインストールし使うことができます。

またハイパーバイザには二種類あります。

Xenの例は下記のようになります

### Xenの例

[Xen (仮想化ソフトウェア)](https://ja.wikipedia.org/wiki/Xen_(%E4%BB%AE%E6%83%B3%E5%8C%96%E3%82%BD%E3%83%95%E3%83%88%E3%82%A6%E3%82%A7%E3%82%A2))

#### 準仮想化 (ParaVirtualization)
実在のハードウェアを完全にエミュレートする代わりに、仮想マシン環境を実現するのに都合の良い仮想的なハードウェアを再定義するものです。

ゲストOSに修正が必要になります。

この仮想ハードウェアは、実在のハードウェアに似ているが、操作をするためにはハイパーバイザコールを呼び出す必要がある。 Xenはこのハイパーバイザコールの要求に応じて、仮想マシン環境に変更を加えます。

この実装手法はエミュレーションのオーバーヘッドを最小限に抑えることができるため、性能面で大きなアドバンテージがあるが、 OSをXen仮想ハードウェア上に移植する必要があります。

#### 完全仮想化 (FullVirtualization)
実ハードウェア用に用意されたOSをそのまま動作させることができます。

ゲストOSを修正せずそのまま稼働できるのがメリットです。

完全仮想化機能が提供する仮想マシン環境内のOSは、特権モードで動作し完全に物理ハードウェアを支配しているかのように振る舞います。

実際には、仮想マシン側のOSが仮想ハードウェアを制御する命令を実行したとき、仮想ハードウェアがそれを検出し、例外のようなものが発生してXenに制御を渡します。

制御を渡されたXenは、OSが行おうとした処理を分析し、仮想ハードウェアの動作をエミュレートすることになります。

完全仮想化の環境は、準仮想化方式に比べると、エミュレーションのためのコストが大きくなりますが、ソフトウェアをユーザの手で変更することが難しいWindowsなどのOSも動作させることができます。

完全仮想化を行うためには Intel VT などのCPU(ハードウェア)の支援が必要となります。

(むかしBIOSでVTがdisableになってハマったな、、、)

## (ハイパーバイザー型)仮想化の問題点
### リソースのオーバーヘッド
仮想マシンはOSも隔離されています。

そのため、仮想マシンは使うOSを動かすぶんのCPU、メモリなどのリソースを使うことになります。

### ハイパーバイザのオーバーヘッド
これはどれくらいあるのかな、

はてなの実績(Xen)だとベアメタルとくらべて2 〜 3%(2010年とか)とありました。

2009年当時ではKVMだと15%のオーバヘッドがあると書いてありますが、現状はもっと抑えられているはずだと思われます。

https://enterprise.watch.impress.co.jp/docs/series/virtual/323949.html

Nitoro Systemは 仮想化オーバーヘッドを排除って書いてありました。

https://aws.amazon.com/jp/about-aws/whats-new/2018/07/amazon-ec2-nitro-system-based-instances-now-support-faster-ebs-optimized-performance/

### 起動時間
仮想マシンは内部にOS(ゲストOS)があります。

![](https://image.itmedia.co.jp/ait/articles/0903/12/r12fig01.jpg)

仮想マシンで目的のアプリケーションを起動するには、

1. OSの立ち上げ
2. アプリケーションの起動

となり、起動には数分かかる場合がほとんどです。

## (ハイパーバイザー型)仮想化のメリット

ハイパーバイザー型仮想化ではOSの隔離されているため、ゲストOSでは異なるOSを立ち上げることができるというメリットがあります。


## 参考資料
* コンテナ型仮想化とサーバ仮想化の比較
  * https://note.com/ryoma_0923/n/nd96162cddccb
* Docker関連
  * https://knowledge.sakura.ad.jp/13265/
  * https://eh-career.com/engineerhub/entry/2019/02/05/103000
  * https://community.hpe.com/t5/hpe-blog-japan/docker%E3%82%B3%E3%83%B3%E3%83%86%E3%83%8A%E3%81%A8%E4%BB%AE%E6%83%B3%E5%8C%96%E3%81%AE%E9%81%95%E3%81%84%E3%81%A8%E3%81%AF-synergy%E3%81%A8devops/ba-p/6980068?profile.language=ja#.X64WiVMzbzJ
* Xenの仮想化のしくみ 
  * https://ja.wikipedia.org/wiki/Xen_(%E4%BB%AE%E6%83%B3%E5%8C%96%E3%82%BD%E3%83%95%E3%83%88%E3%82%A6%E3%82%A7%E3%82%A2)
* KVMの仮想化の仕組み
  * https://www.atmarkit.co.jp/ait/articles/0903/12/news120.html
  * https://gihyo.jp/dev/serial/01/vm_work
  * https://www.fujitsu.com/downloads/JP/archive/imgjp/jmag/vol62-1/paper18.pdf
  * https://enterprise.watch.impress.co.jp/docs/series/virtual/323949.html
* 自分がKVMを使ったときの記録
  * https://spring-mt.tumblr.com/tagged/kvm
* Intel VT
  * https://ja.wikipedia.org/wiki/%E3%82%A4%E3%83%B3%E3%83%86%E3%83%AB_%E3%83%90%E3%83%BC%E3%83%81%E3%83%A3%E3%83%A9%E3%82%A4%E3%82%BC%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%E3%83%BB%E3%83%86%E3%82%AF%E3%83%8E%E3%83%AD%E3%82%B8%E3%83%BC
  * https://www.atmarkit.co.jp/fsys/kaisetsu/085intelvt/intelvt.html


