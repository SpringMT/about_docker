# Restriction(権限の制約)
権限の制約は、Capabilityの話となります。

Capabilityはroot権限を細分化してプロセスやファイルに設定する機能です。

> 権限のチェックを行う観点から見ると、伝統的な UNIX の実装では プロセスは二つのカテゴリーに分類できる: 特権 プロセス (実効ユーザーID が 0 のプロセス。ユーザーID 0 は スーパーユーザーや root と呼ばれる) と 非特権 プロセス (実効ユーザーID が 0 以外のプロセス) である。 非特権プロセスでは、プロセスの資格情報 (通常は、実効UID 、実効GID と追加のグループリスト) に基づく権限チェックが行われるのに対し、 特権プロセスでは全てのカーネルの権限チェックがバイパスされる。

> バージョン 2.2 以降の Linux では、 これまでスーパーユーザーに結び付けられてきた権限を、 いくつかのグループに分割している。これらのグループは ケーパビリティ(capability) と呼ばれ、グループ毎に独立に有効、無効を設定できる。 ケーパビリティはスレッド単位の属性である。  

参照 : https://linuxjm.osdn.jp/html/LDP_man-pages/man7/capabilities.7.html

Capabilityによってrootの権限を制限することで、実行しているプログラムに脆弱性があった場合の影響範囲を狭めることができます。

## Capabilityの種類
下記参照

https://man7.org/linux/man-pages/man7/capabilities.7.html

## 参考資料
* https://eh-career.com/engineerhub/entry/2019/02/05/103000#Capability
* https://udzura.hatenablog.jp/entry/2016/06/24/181852
