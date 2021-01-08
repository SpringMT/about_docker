# ベストプラクティス
zembutsuさんのまとめがわかりやすい

https://www2.slideshare.net/zembutsu/explaining-best-practices-for-writing-dockerfiles

https://www2.slideshare.net/zembutsu/dockerfile-bestpractices-19-and-advice



## Docker公式
https://docs.docker.com/develop/develop-images/dockerfile_best-practices/

https://docs.docker.jp/develop/develop-images/dockerfile_best-practices.html (日本語訳なので古い)

## GKEからでているbuild container best practice
https://cloud.google.com/solutions/best-practices-for-building-containers

### 1コンテナ1アプリ

### PID 1、シグナル、ゾンビプロセスのハンドリング
ベストプラクティスは起動時にコンテナにシェルスクリプトを起動させること
専用のinitシステム使う

### Dockerのbuild cacheをうまく使う

### いらないツールいれない
kubernetesだと `readOnlyRootFilesystem` にする

### imageはなるべく小さくする
こんなのもある https://github.com/GoogleContainerTools/distroless
Docker imageで共通のlayerとかもうまく作れれば、キャッシュがうまく効く

### レジストリで脆弱性チェック

### 適切なタグ管理

### publicイメージ使うときはよく確認してから使う
ツールとかもあるから色々駆使して確認する
https://github.com/GoogleContainerTools/container-diff
https://github.com/GoogleContainerTools/container-structure-test
https://cloudplatform-jp.googleblog.com/2017/10/introducing-grafeas-open-source-api-.html
https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#validatingadmissionwebhook-alpha-in-18-beta-in-19
https://kubernetes.io/docs/concepts/policy/pod-security-policy/

## リファレンス
* https://docs.docker.com/engine/reference/builder/
* https://docs.docker.jp/engine/reference/builder.html (日本語)
* http://docs.docker.jp/v17.06/engine/reference/commandline/build.html
