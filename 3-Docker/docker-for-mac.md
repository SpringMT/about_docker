# Docker for Mac 
https://docs.docker.com/docker-for-mac/install/

Docker for Macは HyperKitというツールを使って仮想マシンを立てます。

https://github.com/moby/hyperkit

その他にVPNKit DataKitというツールと協調してDockerを実行しています。

https://github.com/moby/vpnkit

https://github.com/moby/datakit

HyperKitはxHyve/ bHyveを基盤としています。

https://bhyve.org/ → BSD むけのハイパーバイザ

https://github.com/machyve/xhyve → bhyve のmacOS ポート これがHypervisor frameworkを利用して動作している。

![](http://img.scoop.it/FolwNhBi_86hmC1HZFWLB7nTzqrqzN7Y9aBZTaXoQ8Q=)

https://www.docker.com/blog/docker-unikernels-open-source/ より抜粋
HyperKit(というより基盤になっているxHyve)はmacOS 10.10以上から提供されているHypervisor frameworkを利用しています。

https://developer.apple.com/documentation/hypervisor

これにより、VirtualBoxなどをインストールしなくてもDockerがMac上で動作可能になりました。

https://www.docker.com/blog/docker-unikernels-open-source/

Apple Silicon対応

https://news.mynavi.jp/article/osxhack-269/

https://github.com/moby/hyperkit/issues/303

まだちょっとバグってそう

# Docker for Windows
また今度、、

## 参考資料
* http://docs.docker.jp/engine/introduction/understanding-docker.html
* 以前使われていたツール
  * https://docs.docker.com/machine/
  * https://qiita.com/t-imada/items/ed6a76f5b257f5608ad0
* HyperKitとかの話
  * https://www.publickey1.jp/blog/16/docker_hyperkit.html
  * https://www.docker.com/blog/docker-unikernels-open-source/
* Hypervisor framework
  * https://speakerdeck.com/mach0xff/hypervisor-dot-framework-hvf-falsezhong-shen
  * https://ascii.jp/elem/000/001/043/1043010/

