# Dockerfileの書き方ベストトプラクティス
まずは [Dockerfile を書くためのベストプラクティス解説編](https://www.slideshare.net/zembutsu/explaining-best-practices-for-writing-dockerfiles) これを読みましょう。

これを前提にしてさらに追加でノウハウを書いていきます。

# ベースイメージの選定
## alpineを使うのか？
[alipine](https://alpinelinux.org/) はサイズが小さいLinuxディストリビューションです。 <br>
サイズがかなり小さいので、サイズを小さくすることが重要なDockerの運用においてはalpineを採用する事例が結構あります。 <br>
しかし、libcがmuslという独自のlibcを使っていたりしてハマりどころも多いです(grpcのコンパイルはできなかったです) <br>
またパフォーマンスの面でも、mallocが遅いなどの報告もあり、パフォーマンス測定などをやった上で本番環境利用をしたほうがよさそうです。 <br>
* 事例 : https://superuser.com/questions/1219609/why-is-the-alpine-docker-image-over-50-slower-than-the-ubuntu-image

そもそも入っているパッケージも少ないく、なんだかんだインストールしていくとDockerイメージは大きくなっていき、結局Ubuntu使ったほうが小さくなるといった事例も報告されています。
* 事例 : https://pythonspeed.com/articles/alpine-docker-python/

## どのベースイメージを使うのがよいか？
まず、Docker hubに上がっている公式イメージ([Docker Offical Image](https://docs.docker.com/docker-hub/official_images/) )から探すのがよいかと思います。

### Ubuntu
UbuntuであればGoogleが出しているイメージもあります。<br>
このイメージはパッケージの頻繁に更新されていて、最新版を使えばGoogleの脆弱性スキャンはだいたい大丈夫になります(マッチポップ感はありますが)
* Registry : https://console.cloud.google.com/gcr/images/gcp-runtimes/GLOBAL/ubuntu_16_0_4?gcrImageListsize=30

春山はGoogleのUbuntuのイメージにRubyをインストールした独自のイメージをベースイメージにしてサービスで使っています。<br>
https://github.com/SpringMT/google-ruby

# ビルド
開発イテレーションを高速に回すために、Dockerイメージのビルドのスピードは重要です。

## イメージキャッシュ
ビルドの高速にするために、イメージキャッシュをうまく使う必要があります。

### GCPのCloud Buildを使ってDockerイメージをビルドする
kanikoを使いましょう。

* https://cloud.google.com/cloud-build/docs/kaniko-cache?hl=ja

設定の一例は下記の通りです。

```
steps:	
- name: gcr.io/kaniko-project/executor	
  id: api	
  args:	
  - --dockerfile=Dockerfile	
  - --destination=asia.gcr.io/$PROJECT_ID/$REPO_NAME/api/$BRANCH_NAME:$COMMIT_SHA	
  - --cache-repo=asia.gcr.io/$PROJECT_ID/$REPO_NAME/cache	
  - --cache=true	
  - --cache-ttl=6h
  waitFor: ['-']
- name: gcr.io/cloud-builders/curl	
  id: curl
  args: ['--data-urlencode', "payload={\"text\":\"Images $BRANCH_NAME $COMMIT_SHA\",\"attachments\":[{\"title\":\"api at rerep-server\",\"fields\":[{\"title\":\"images\",\"value\":\"$BRANCH_NAME:$COMMIT_SHA\"},{\"title\":\"url\",\"value\":\"asia.gcr.io/$PROJECT_ID/$REPO_NAME/api/$BRANCH_NAME:$COMMIT_SHA\"}]}]}", "-X",  "POST", 'https://hooks.slack.com/services/xxxxx']	
  waitFor:	
  - api
```


### CircleCIでDockerイメージをビルドする場合
社内モジュールなどを取り込む必要があるなど、Cloud上でビルドができない場合も結構あるかと思います。<br>
その場合は、CircleCIでDockerイメージをビルドしてpushするのがよいでしょう。<br>
CircleCI上のイメージキャッシュはマシン毎になるので、Dockerfileを更新したりするとビルドの時間がばらつきます<br>

CircleCI上でDockerイメージをビルドする場合にイメージキャッシュを効かせるためには、config.yamlで下記が必要になります。<br>

* https://circleci.com/docs/2.0/docker-layer-caching/

#### プライマリの Executor がdockerコンテナである場合
例
```
jobs:
  build_and_push_api_docker_image:
    docker:
      - image: google/cloud-sdk
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



CircleCIはAWSとの連携についての設定がSetting -> AWS permissionからできるので、AWSのECRにpushする場合credentialとかを埋め込まなくても良いはずです。
* https://aws.amazon.com/jp/ecr/


## 参照
* https://www.slideshare.net/zembutsu/explaining-best-practices-for-writing-dockerfiles
* https://docs.docker.com/develop/develop-images/dockerfile_best-practices/
