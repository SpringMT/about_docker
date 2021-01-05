# コンテナ型仮想化とは
いくつか表現を抜き出してみます。

https://eh-career.com/engineerhub/entry/2019/02/05/103000

> コンテナはコンテナランタイムによって作成された、ホストOSのリソースを隔離・制限したプロセスです。

https://note.com/ryoma_0923/n/nd96162cddccb

> コンテナ型仮想化では同一OS上に隔離空間を作りその中にアプリケーションの動作に必要なライブラリなどを同梱することで隔離された空間を作成します。

https://gihyo.jp/admin/serial/01/linux_containers/0002

> 動する全てのプロセスはコンピュータ上にインストールされたOS（ホストOS）上で直接起動します。通常のプロセスの動作と異なるのは，そのプロセスの一部をグループ化し，他のグループやグループに属していないプロセスから隔離した空間で動作させる点です。貨物輸送のコンテナのように，隔離された空間にプロセスが入っているので，この空間を『コンテナ』と呼ぶわけです。実際のコンテナのように，あるコンテナの内部から他のコンテナの内部を見ることはできません。

![](https://cloud.google.com/images/containers-landing/containers-101-2x.png)

参照 https://cloud.google.com/containers

![](https://hpeb.i.lithium.com/t5/image/serverpage/image-id/98754iEF6846F0AEE18604/image-size/large?v=1.0&px=2000)
参照 https://community.hpe.com/t5/hpe-blog-japan/docker%E3%82%B3%E3%83%B3%E3%83%86%E3%83%8A%E3%81%A8%E4%BB%AE%E6%83%B3%E5%8C%96%E3%81%AE%E9%81%95%E3%81%84%E3%81%A8%E3%81%AF-synergy%E3%81%A8devops/ba-p/6980068?profile.language=ja#.X64f5FMzbzJ

これらのように、コンテナというのはホストOS上の *プロセス* です。

このプロセスを、OSの機能を使って隔離して、他のコンテナから見えないようにしています。

## コンテナ型仮想化のメリット・デメリット
ハイパーバイザー型との比較になります。

### メリット

#### リソースのオーバーヘッドの削減
コンテナはホストOS上で動くプロセスにすぎないので、アプリケーションバイナリと、それが動作するのに必要最小限のライブラリさえあればよいので、リソースのオーバーヘッドは抑えられる。

#### 起動時間の短縮
実質的にはプロセスを立ち上げるだけなので、ハイパーバイザ型に比べて起動時間は短縮できます。

![](https://d2l930y2yx77uc.cloudfront.net/production/uploads/images/19151697/picture_pc_da0c0ae9e31395cd92b9bda6ed209828.png)
参照 : https://note.com/ryoma_0923/n/nd96162cddccb

### デメリット

#### 異なるOSのシステム，プログラムは動かせない
コンテナはあくまでプロセスなので、異なるOSのプログラムは動かせません。

#### OSを共有しているので隔離機能がハイパーバイザ型に比べて弱い

#### コンテナから見えるデバイスやロードされているカーネルモジュールはホストOSと同じ
OSを共有しているので、コンテナのカーネルモジュールはホストOSを同じになります。

## 参考資料
* OpenVZ
  * https://www.nic.ad.jp/ja/materials/iw/2012/proceedings/d1/d1-Ebisawa.pdf
  * https://www2.slideshare.net/kentaroebisawa/openvz-linux-containers-26882926
* LXC
  * https://knowledge.sakura.ad.jp/2108/
  * https://gihyo.jp/admin/serial/01/linux_containers?start=40
* 気になる論文
  * [Adding Generic Process Containers to the Linux Kernel](https://www.kernel.org/doc/ols/2007/ols2007v2-pages-45-58.pdf)
    * 2007の論文
  * [Resource Containers: A New Facility for Resource Management in Server Systems](https://www.usenix.org/legacy/publications/library/proceedings/osdi99/full_papers/banga/banga.pdf)
    * 1999の論文
