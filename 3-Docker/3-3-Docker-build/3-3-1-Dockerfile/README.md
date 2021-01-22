# Dockerfile
Dockerイメージを作成する方法はたくさんあります。

単純なshell scriptでも作ることができます。

https://github.com/moby/moby/blob/master/contrib/mkimage-alpine.sh

しかし、こんなことを毎回するのは面倒すぎます。

なので、それらをまとめてやるための仕組みがDockerfileとdocker buildです

## Dockerfile
DockerはDockerfileに記述されている命令を読み込んでDockerイメージを自動的に作成します。

Dockerfileを元にDockerイメージを作るために、docker buildコマンドを利用します。

https://github.com/moby/moby/blob/af14f37a115103f3012ea8fa8255e1628c8008bd/docs/api/v1.40.yaml#L7043

### Dockerfileの記述形式

```
# Comment
INSTRUCTION arguments
```

1命令 + 1引数を一行に書きます。

### Dockerfileで使える命令
https://docs.docker.jp/develop/develop-images/dockerfile_best-practices.html

https://docs.docker.com/develop/develop-images/dockerfile_best-practices/

https://docs.docker.com/engine/reference/builder/

https://docs.docker.jp/engine/reference/builder.html

一部抜粋します。

コマンド名 |形式(代表例) | 詳細 | レイヤーの作成 | 参考
-|-|-|-|-
ARG | | CLI上から引数を代入するキーを指定します。<br>ARG <key> といった感じに書いておいて、 --build-arg <key>=<value> オプションで代入することで、 Dockerfile 内で値は使い回しすることができます。 | x |
FROM | FROM <image> [AS <name>] | イメージビルドのための処理ステージを初期化し、ベース・イメージ(Dockerfile 内で親イメージを持たないもの)を設定します。<br>後続の命令がこれに続きます。<br>正しい Dockerfile は FROM 命令から始める必要があります。<br>マルチステージビルドを行う場合は複数回記述することになります。<br>ARG コマンドは唯一 FROM の前に来ていいコマンドとなります。<br>ARG 命令によって宣言された変数すべてを参照できます。| o |	
RUN | RUN <command>(シェル形式) <br> RUN ["executable", "param1", "param2"](exec形式) | RUN 命令は、現在のイメージの最上位の最新レイヤーにおいて、あらゆるコマンドを実行し、処理結果を確定します。<br>結果が確定したイメージは、Dockerfileの次のステップにおいて利用されていきます。<br>シェル形式はデフォルトで Linux なら /bin/sh -c<br>https://github.com/moby/moby/blob/46cdcd206c56172b95ba5c77b827a722dab426c5/builder/dockerfile/internals.go#L419-L426<br>https://docs.docker.com/engine/reference/builder/#run<br>注釈は読んでおく| o | これは命令に対するargument( コマンド文字列 )がkeyでキャッシュされます。<br>なので、キャッシュを破棄する場合はRUNより前でキャッシュを破棄する命令を書くか、--no-cacheフラグを使う
ADD | ADD [--chown=<user>:<group>] <src>... <dest> <br> ADD [--chown=<user>:<group>] ["<src>",... "<dest>"] | ADD 命令は <src> に示されるファイル、ディレクトリ、リモートファイル URL をコピーして、イメージ内のファイルシステム上のパス <dest> にこれらを加えます。<br> <src> のパス指定は、ビルド コンテキスト内で有効なパスとします(../とか使えない)<br> <src>にはURLも指定できます。<br> <src> が ローカル にある tar アーカイブであって、認識できるフォーマット（gzip、bzip2、xz）である場合、1 つのディレクトリ配下に展開されます。 リモート URL の場合は展開 されません 。 ディレクトリのコピーあるいは展開の仕方は tar -x と同等<br> 動きとしては tar -x と同じ。| o |	キャッシュはsrcのファイル群のchecksum<br>個々のファイルについてチェックサムが計算されます(ファイルの最終更新時刻、最終アクセス時刻は考慮されない) キャッシュを探す際に、このチェックサムと既存イメージのチェックサムが比較されます。 <br>たとえばファイル内容やメタデータが変わっていれば、キャッシュは無効になります。
COPY | | ADDと似ていますが、URLの指定や、自動的な展開などはしないコマンドになります。<br>COPY は単に、基本的なコピー機能を使ってローカルファイルをコンテナにコピーするだけです。<br>一般的にはADDよりCOPYを優先して使ってください。(わかりやすさ) | o | ADDと同じ
ENTRYPOINT | ENTRYPOINT ["executable", "param1", "param2"] exec形式 <br> ENTRYPOINT command param1 param2　shell形式 | 一番初めに実行すべきコマンドのオプション定義です。<br>シェル形式ではCMD や docker run におけるコマンドライン引数は無視します。| x |CMD と ENTRYPOINT の関連について<br>https://docs.docker.jp/engine/reference/builder.html#cmd-entrypoint
CMD |	CMD ["executable","param1","param2"] (exec form, this is the preferred form) <br>CMD command param1 param2 (shell form) <br>CMD ["param1","param2"] (as default parameters to ENTRYPOINT) | CMD 命令の主目的は、コンテナの実行時のデフォルト処理を設定することです。<br>shell形式、exec形式で定義するとイメージが起動されたときに実行するコマンドの指定となります・<br>exec形式が推奨です。 | x | 
ENV | ENV <key>=<value> ... | 環境変数 <key> に <value> という値を設定します。<br>Dockerfile 内で定義して以降、使い回したり、値を上書きできたりします。 | x | 
LABEL | イメージにメタデータを付与するコマンド。バージョンはいくつだとか、作成者の名前とかをキーバリューで記入したりするのに使う。 | x | https://github.com/opencontainers/image-spec/blob/79b036d80240ae530a8de15e1d21c7ab9292c693/annotations.md#back-compatibility-with-label-schema
EXPOSE | コンテナが公開するポートを指定します。<br>ただ、これはコンテナ側から公開するだけなので、実際にホスト側からアクセスするには、 docker run -p 80:80 みたいな感じでバインドする必要があります。 | x |
VOLUME | ホスト側のデータをマウントするディレクトリを指定するコマンド。 | x | 
USER | USER <user>[:<group>] <br> USER <UID>[:<GID>] | USER 命令は、ユーザ名（または UID）と、オプションとしてユーザグループ（または GID）を指定します。 そしてイメージが実行されるとき、Dockerfile 内の後続の RUN、CMD、ENTRYPOINT の各命令においてこの情報を利用します。 | x | 
WORKDIR | WORKDIR /path/to/workdir | WORKDIR 命令はワークディレクトリを設定します。<br> WORKDIR が存在しないときは生成されます。| x |
ONBUILD | | ONBUILD 命令は、イメージに対して トリガ 命令（trigger instruction）を追加します。 トリガ命令は後々実行されるものであり、そのイメージが他のビルドにおけるベースイメージとして用いられたときに実行されます。| x | わかりにくいので正直使わないほうがいいと思っています。 <br>例 : https://github.com/nodejs/docker-node/blob/a8dbfa5c7cac9dca9145c6f429cd2c4f11176707/8/onbuild/Dockerfile
STOPSIGNAL | コンテナーを終了するためのシグナルを指定する。 <br>デフォルトのsignalは 9 、 SIGNAME 、 SIGKILL 等が指定できます。 | x | 
HEALTHCHECK | | HEALTHCHECK 命令は、コンテナが動作していることをチェックする方法を指定するものです。<br>コンテナのヘルスステータスはdocker inspectで確認できます。| x | 
SHELL | | Dockerfile 内で実行されるシェルコマンドの形式を上書きするコマンド。 Linux 環境の場合だと、デフォルトで /bin/sh -c として実行されるが、これを /bin/bash などに変更できます。		

https://spring-mt.hatenablog.com/entry/2020/12/16/032810

http://www.redout.net/data/tar.html tarの構造

## その他使わなさそうなやつ
### パーサ・ディレクティブ
https://docs.docker.jp/engine/reference/builder.html#parser-directives

syntaxディレクティブはBuildKitを使っている場合に使える。

https://docs.docker.com/engine/reference/builder/#syntax

escapeディレクティブを使ってエスケープ文字として用いる文字を指定できる。

デフォルトは\ （バックスラッシュ）

https://docs.docker.com/engine/reference/builder/#escape


## CMDとENETYPOINTの関連
https://docs.docker.jp/engine/reference/builder.html#cmd-entrypoint

シェル形式とexec形式とかシェルとかをちゃんとやるときの参考
https://qiita.com/ukinau/items/410f56b6d777ad1e4e90

https://qiita.com/RyoMa_0923/items/9b5d2c4a97205692a560

https://qiita.com/hyamatan/items/0b7df8927ddbab603104

## Linter
https://github.com/hadolint/hadolint

## 参考
https://github.com/docker-library

