# 2. コンテナ型仮想化の仕組み
コンテナ仮想型において、コンテナはあくまでホストOSのプロセスとなります。

そのままでは、プロセスなので他のコンテナから独立はできていません。

そこで、独立した実行環境にするために、ホストOSのリソースを隔離・制限を行います。

ここからは主にLinuxの話になります。

VirtualBox + Vagrantで作った環境で検証していきます。

## 確認環境の立ち上げ方
VirtualBoxのinstall  https://www.virtualbox.org/

Vagrantのinstall  https://www.vagrantup.com/

Vagrantfile

参照 : https://eh-career.com/engineerhub/entry/2019/02/05/103000#%E3%82%B3%E3%83%B3%E3%83%86%E3%83%8A%E3%82%92%E4%BD%9C%E3%81%A3%E3%81%A6%E3%81%BF%E3%82%88%E3%81%86

```
# -*- mode: ruby -*-
# vi: set ft=ruby :
 
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/bionic64"
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "1024"
    vb.cpus = 2
  end
  # cgdb https://cgdb.github.io/
  # cgroup-tools https://christina04.hatenablog.com/entry/2020/02/24/170724 cgroupに関する操作
  config.vm.provision "shell", inline: <<-SHELL
    apt-get update
 
    apt-get install apt-transport-https ca-certificates curl software-properties-common jq
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
    add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
    apt-get update
    apt-get install -y docker-ce
 
    apt-get install -y cgdb
    apt-get install -y cgroup-tools
 
    apt-get install -y make gcc
    git clone git://git.kernel.org/pub/scm/linux/kernel/git/morgan/libcap.git /usr/src/libcap
    (cd /usr/src/libcap && make && make install)
  SHELL
end
```

上記の内容を `Vagrantfile` としておいて、 `vagrant up` を実行すれば立ち上がります。

`vagrant ssh` でVMにsshで入れます。

`vagrant suspend` でVMをsupendできます。
