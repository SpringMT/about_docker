# Dockerfileの書き方
## linterを使ってDockerfileの構文チェック
hadolintなどがおすすめです。

https://github.com/hadolint/hadolint

## ベースイメージの選定
どのディストリビューションを使うのかなどの検討が必要です。

Dockerイメージを小さくするためにalpineの利用を検討されている方も多いかと思いますが、alpineを使う場合の注意点は [こちら](https://github.com/SpringMT/about_docker/blob/main/3-Docker/3-3-Docker-build/3-3-4-best-practice/dockerfile.md#alpine%E3%82%92%E4%BD%BF%E3%81%86%E3%81%AE%E3%81%8B) にまとめました。

## 使える命令を整理する
下記をご参照ください。

https://github.com/SpringMT/about_docker/tree/main/3-Docker/3-3-Docker-build/3-3-1-Dockerfile

# コンテナの実行について

## 1コンテナ1アプリ
アプリとは一つの親プロセスとそこから作成される子プロセス群のこと。

1つのコンテナに複数のアプリが入っている場合、それらのアプリのライフサイクルが異なっていたり、状態が異なってしまいます。

複数のアプリが入っていると、コンテナの構成管理ツールが、コンテナがhealtyであるかの判断が難しくなります。

複数アプリが入っていると、お互いに状態を管理しなければならなくなります。。

コンテナは起動、停止を頻繁に行えることがメリットであり、このメリットを活かすためには、ステートレスであることが重要です。

他にもリソース分離、関心分離などの観点からも1コンテナ１アプリを徹底する必要があります。

### PID 1、シグナル、ゾンビプロセスのハンドリング
コンテナを実行すると、アプリケーションのプロセスが PID 1（プロセス番号が1番）で実行されるため、コンテナに対して `SIGTERM` などのシグナルを送信してもコンテナ内のプロセスが正常に終了しない問題があります。

* アプリケーションが明示的にシグナルをハンドリングするようにする
* PID 1 で実行されないようにする
  * アプリケーションプロセスが PID 1 で実行されないようにする場合、Docker では Tini のような軽量 init を使う
* Docker 1.13 以上の場合は docker run の --init オプションを使うで問題を回避できる
* Kubernetes では Pod shareProcessNamespace を使うことで問題を回避できる

[Docker/Kubernetes で PID 1 問題を回避する](https://text.superbrothers.dev/200328-how-to-avoid-pid-1-problem-in-kubernetes/)

# Dockerのビルド
ビルドは速いことが重要。

## .dockerignoreを使って不要なファイルを転送しないようにする
docker build時に指定したビルドコンテキストに含まれるディレクトリ、ファイルがDockerfileに関係なくDockerに送られます。

ここで不要なファイルなどがあると、docker buildに時間がかかってしまいます。

`.dockerignore` を利用して不要なファイルの転送を抑制することができます。

https://docs.docker.com/engine/reference/builder/#dockerignore-file

## ビルドキャッシュを使う
Dockerイメージはレイヤー構成になっています。

レイヤーが追加される操作は こちら にあります。

レイヤーはそれぞれ、キャッシュされています。

レイヤーが変更されると、それ以降のレイヤーのキャッシュは破棄され、あたらしくレイヤーを作り直します。

そこで、依存があるものは一つの命令にまとめ、レイヤーキャッシュをうまく使うことでDockerイメージのビルドの高速化を目指します。

### CircleCIでDockerイメージをビルドする場合
CircleCI上のイメージキャッシュはマシン毎になるので、Dockerfileを更新したりするとビルドの時間がばらつきます。

CircleCI上でDockerイメージをビルドする場合にイメージキャッシュを効かせるためには、config.yamlで下記が必要になります。

* https://circleci.com/docs/2.0/docker-layer-caching/

#### プライマリの Executor がdockerコンテナである場合
例
```
jobs:
  build_and_push_api_docker_image:
    docker:
      - image: nginx
    steps:
      - checkout
      - setup_remote_docker:
          docker_layer_caching: true
```

#### machine Executoreの場合
例
```
machine:
  docker_layer_caching: true 
```

## 秘匿情報をADDしない
秘匿情報をADDしてしまって、あとの操作で削除しても、ADDをしたレイヤーには秘匿情報が残ってしまいます。

ビルド時に必要な秘匿情報はADDでファイルを使ってDockerイメージに入れるのではなく、環境変数で渡すようにします。

## イメージのサイズをできる限り小さくする

### できればマルチステージビルドを使う
マルチステージビルドによって、ビルドには必要だが、実行には不要なライブラリなどを取り除くことができます。

これにより、イメージのサイズを小さくすることが可能です。

### 不要なライブラリ、ファイルを削除する
不要なファイルは消しましょう。

パッケージ管理ツールによって入ってしまう不要なファイルの消しかたの例です。

```
apt-get clean && rm -f /var/lib/apt/lists/*_*
```

```
rm -rf /var/cache/yum/* && yum clean all
```

## レジストリでの脆弱性チェックを行う
作成したDockerイメージに脆弱性がないかはアプリケーションを実行する上でとても重要です。

OSSでは [trivy](https://github.com/aquasecurity/trivy) などがあります。

各クラウドプロバイダーにもあるサービスを利用するのもよいでしょう。

## タグを適切に付けましょう
それぞれのDockerイメージが識別できるようにタグを適切につけましょう。

gitのコミットハッシュをつけ、どのコードベースから生成されたか判明できるようにすることが推奨です。

最新のDockerイメーjにlatestをつけて運用する場合、思わぬタイミングでdeployされてしまうので気をつけましょう。

## publicイメージ使うときはよく確認してから使いましょう

# Dockerイメージの分析
diveなどのツールを使ってDockerイメージの容量が大きい要因などを分析可能です。

https://github.com/wagoodman/dive

# コンテナの運用
## ログの運用

## イミュータブルかつステートレス

## 特権コンテナを割ける

## モニタリングしやすいようにする

## ヘルスチェック

## FileSystemをread-onlyにする

kubernetesだと `readOnlyRootFilesystem` にするなどの対応があります。

## rootでの起動を避ける

## Dockerイメージのバージョンを注意深く管理する



# 参考資料
zembutsuさんのまとめがわかりやすい

https://www2.slideshare.net/zembutsu/explaining-best-practices-for-writing-dockerfiles

https://www2.slideshare.net/zembutsu/dockerfile-bestpractices-19-and-advice

## Docker公式
https://docs.docker.com/develop/develop-images/dockerfile_best-practices/

* 日本語訳ですが少し古いです。
https://docs.docker.jp/develop/develop-images/dockerfile_best-practices.html
