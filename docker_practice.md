# Dockerについて学習する

## 概要
```control

用語:
    Docker: 
        ・ホストのカーネルを利用し、プロセスやユーザを隔離することで、別のマシンが動いているように見せかけるもの
        ・仮想マシンではないので、軽量
        ・ミドルウェアのインストールや、各種環境設定をコード化して管理できる(IaC)
        ・同じ環境をすぐに作れる。配布も容易(サーバの各種設定やインストールなどをコードで共有できるので、)
        ・よって、環境構築等の無駄な時間をなくせる
        ・コンテナ内での、アプリのパッケージ化と、実行機能を提供する
        ・クラスタ構成が容易(コンテナ名を変えるだけ)
        ・コード化したものをCI/CDツールで毎日実行すれば、リリースサイクルも容易になる

    Dockerデーモン(dockerd):
        Docker APIリクエストを受け付け、イメージやコンテナなどのDockerオブジェクトを管理する
        他のデーモンとも通信を行う

    Dockerクライアント(docker)
        docker run等のコマンドを通じ、dockerdに命令を伝える
        複数のデーモンと通信可能

    Dockerデスクトップ
        コンテナ化したアプリケーションと、マイクロサービスを構築、共有可能
        dockerd, docker, Docker Compose, Docker Content Trust, Kubernetes, CredentialHelperが含まれる
    
    Dockerレジストリ
        Dockerイメージを保管する(GHCR等もこの一種)
        デフォルトでは、DockerHubのイメージを探す
        docker pull -> 設定されたレジストリからイメージを取得
        docker push -> イメージを指定したレジストリに送信

    Dockerイメージ
        ・Dockerコンテナを作成する命令が入っている読み込み専用のテンプレート
        ・他のイメージ(たとえばubuntu)をベースとして、カスタマイズしたイメージを利用する
        ・例) ubuntuイメージをベースにapacheをインストールし、その設定を加えたもの
        
        イメージを作成する場合
            Dockerfileを作成
            ※Dockerfileを書き換えた場合、イメージは再構築されるが、その時は変更されたレイヤのみを再生成するので、ほかの仮想化技術に比べて軽量になっている

    コンテナ
        ・Dockerイメージが実行状態となったインスタンス
        ・生成、開始、停止などを行いたい場合、Docker APIやCLIを用いる
        ・複数のネットワークへの接続やストレージ追加、現時点の状態をもとにした新たなイメージの生成が可能。
        ・デフォルトではコンテナ同士は分離される
        ・コンテナを削除すると永続的なストレージに保存されていないものは消失する

    docker runコマンドの例
        docker run -it ubuntu /bin/bash

    上記の説明
        0. docker run = docker pull + docker create + docker startのようなもの
        1. docker run ubuntu
            ubuntuイメージがlocalになければ設定察れているレジストリからイメージ取得=docker pull ubuntu
        2. Dockerは新しいコンテナを取得=docker create
        3. Dockerはコンテナに対し、読み書き可能なファイルシステムを最後のレイヤとして割り当てる。=実行中のコンテナは、ファイルの変更が可能
        4. Dockerはネットワークインターフェースを生成し、コンテナを接続する。コンテナに対するIPアドレスの割り当てもなされる
        5. Dockerはコンテナを起動し、/bin/bashを実行する。
            -i 標準入力を受け付ける
            -t 標準入出力となっている端末デバイスを指定
            上記のオプションがあるので対話的に行える
        6. exitで/bin/bashコマンドは終了する(停止状態であり、削除は行われない)
    
    なぜコンテナが隔離された作業空間を準備できるのか
        namespacesを用いているため


    
具体例:
    1. 作業環境を共有するために、開発段階で用いる
    2. テストも、テスト環境のDockerで行う
    3. バグがあったら開発環境に戻し、テスト環境に再デプロイする
    4. ユーザに修正版を配布する

仕組み:
    ・クライアントはdocker build等の命令を通して、デーモンにDockerコンテナの配布、構築、実行などを行わせる
    ・デーモンとクライアントは必ずしも同一システム上にある必要はない
    ・Dockerクライアントとデーモンの間の通信にはREST APIが用いられる

```
## Docker Desktopのインストールに際して
```
- WSL2が有効化されている
- 何らかのLinuxディストリビューションがsetupされている

windows11なら以下のコマンドを打ち込むだけでいい
    wsl --install
```

## Getting Started
```
docker run -dp 80:80 docker/getting-started

    -d : コンテナをバックグラウンドで実行
    -p 80:80 : ホストの80番ポートをコンテナの80番ポートに割り当てる
    docker/getting-started : 使用するdockerイメージ

-> 実行すると、Docker DesktopのDashBoardに情報が追加された
-> Desktop上から使用ポートを確認したり、停止が可能
```

## 用語補足
```
sandbox: 
    通常利用する領域から隔離され、保護された空間に構築された仮想環境

隔離をなぜ実現できるのか
    カーネルのnamespaceとcgroupの活用
    コンテナ: 単一のホスト上で実行されるプロセスの分離されたグループ

    cgroup:
        ・プロセスをグループ化し、リソース制御を行う
        ・CPU,メモリ,ディスクI/O,ネットワーク帯域幅などのリソースを制御し、各プロセスグループに割り当てる
        ・Dockerコンテナはcgroupsを使用して、各コンテナのリソース制御を管理する。これによりコンテナ間でのリソース競合を防ぐ
    Namespaces:
        ・プロセスやリソースの隔離を実現するためのLinuxカーネル機能
        ・異なるコンテナ間でプロセスやファイルシステム、ネットワークなどの名前空間を分離
        ・Dockerコンテナは異なる名前空間で実行されるので、各コンテナは独自のプロセスID,ファイルシステム、ホスト名などを持つ。よって互いの影響を受けずに実行可能
        ・例) PID Namespaces->プロセスIDを隔離する

    cgroups + Namespaces:
        Dockerは名前空間でプロセスグループを隔離し、cgroupsでプロセスグループ(コンテナ)にリソースを制御、割り当てている。
        それによってリソースの競合はおきず、独自の環境が保たれる

    

Docker利点まとめ
    local,仮想マシン上で実行可能
    クラウドにもデプロイ可能
    多くのOSで実行可能
    imageには、依存関係、設定ファイル、スクリプト、バイナリ、環境変数、メタデータ等が含まれる


```

## アプリケーションのコンテナ化

```
制作物
    Node.jsで動作するシンプルなTodoリスト

注意点
    ・Git Bashで動かす際は、\記号をエスケープする必要がある
    ・docker buildの実行はUnix環境なので、行末はrnではなく、nとする
```

```powershell
手順

1. package.jsonがあるディレクトリ内でDockerfileを作成

2. Dockerfileに以下のように記述

# syntax=docker/dockerfile:1
# ベースとなるDockerイメージを指定
FROM node:18-alpine
# それ以降の操作を指定したディレクトリで行う
WORKDIR /app
# コンテナへファイルをコピー(ADDの場合、tarの展開まで行う)
# COPY <コピー元> <Dockerイメージ内の展開先>
COPY . .
# OSのコマンドを実行する際に使用
# package.jsonとyarn.lockをもとにyarn installして状態を再現する
# yarnの場合、ソース管理はpackage.jsonとyarn.lockで行うのでnode_modulesが不要となる
# --productionをつけると、本番用のインストールが実行され、開発用パッケージを含めなくなる
RUN yarn install --production
# コンテナ起動時に実行するコマンド
CMD ["node", "src/index.js"]
# Dockerコンテナのポートを公開する
# 使いどころ:docker run -PでEXPOSEしているすべてのポートを公開するなど
EXPOSE 3000

補足.
    package.jsonについて
        ・yarn installでpackage.jsonファイルに記載された依存パッケージをインストール。手作業でversionを変えてyarn installを行うと環境が変わる。
        ・yarn add [パッケージ名]@[バージョン指定]で新しいパッケージを追加可能
        ・例) reactのバージョン^16.3.1をinstall -> yarn add 'react@^16.13.1'
        ・^は以上という意味を持つ -> 16.3.1以上のversionをinstall
        ・追加すると、package.jsonが更新される
        ・実際にインストールしたバージョンはyarn.lockで確認
        ・パッケージをさらに増やした場合、パッケージマネージャはパッケージ同士の依存関係を解消しながらinstallを行う。
        ・なお、npm installはyarn installとyarn addを1つのコマンドでこなしている
        ・しかし、yarnは高速なパッケージ追加やセキュリティ面に強みがある。
        ・uninstallの際、yarn removeであることに注意
        ・yarn add --devコマンドの場合、開発時に必要なパッケージとして追加される。--productionを指定すると、installされなくなる。

        つまり....
            package.jsonとyarn.lockがあれば、yarn installすることで、同じ状態を再現可能。
        
        依存関係を新しくしたい場合
            yarn upgrade(lockファイルのみ変化)
            yarn upgrade --latest(package.jsonも変化)

        自分のプログラムからinstallしたものを読み込みたい場合
            importを用いる
        
        scriptsについて
            コマンドを登録できる
            たとえば今回の場合、prettifyという記載があるので、yarn prettifyで実行可能

    CMDとENTRYPOINTについて
        CMD: コンテナ実行時のデフォルトを指定->Optionを指定すれば上書きされる
             今回の場合、デフォルトでnode src/index.jsが行われる
        ENTRYPOINT:コンテナ実行時に必ず実行される

3. コンテナイメージを構築する
$ docker build -t getting-started .

docker build : 
    Dockerfileを使い、新しいコンテナimageを構築
    Dockerfile内の構文が解釈される（上記補足参照）
-t:
    imageにタグをつける。今回はgetting-started
    今後はこのimage名でコンテナを実行可能

.:
    Dockerに対して、現在のディレクトリ内にあるDockerfileを探せと命令
    WORKDIRの影響を受けていることに注意

4. コンテナを起動する
$ docker run -dp 127.0.0.1:3000:3000 getting-started

docker run
    1. 指定されたimage(getting-started)に、書き込み可能なコンテナをcreateする
    2. 指定されたコマンドを使ってstartする
    補足1. 二回目以降は、docker startで再起動可能
    補足2. 全てのコンテナはdocker ps -aで確認可能
        (実行中のコンテナのみ表示したい場合は、aオプションを付けない)
-d
    コンテナをバックグラウンドで実行
-p
    ホストとコンテナ間でポートを関連付ける
    HOST:CONTAINERなので、今回はコンテナのポート3000番を、、ホスト上の127.0.0.1:3000へ割り当てた(公開した)
    ポート割り当てを行わなければ、ホスト上からアプリケーションへは接続できない

getting-started
    起動するimageをタグ名を用いて指定している

```

## アプリケーションの更新
```
1. ソースコードを更新する

2. イメージを更新し、これをbuildする
$ docker build -t getting-started .

3. 古いコンテナが3000番を使用しているので停止して削除する
    3-1. docker psでコンテナのIDを調べる
    3-2. docker stop <コンテナID> で停止
    3-3. docker rm <コンテナID>で削除
    3-4. Dockerダッシュボードで削除してもいい

4. docker runで更新したアプリを起動
$ docker run -dp 127.0.0.1:3000:3000 getting-started

問題点:
    todoリストに追加していたアイテムが消えてしまう
    -> 再構築を必要としないコードの編集方法
    -> 変更するたびに新しくコンテナを起動する方法については以下で学ぶ
```

## アプリケーションの共有
```
概要
    ・Dockerイメージを共有するにはDockerレジストリを使う
    ・GHCRやDockerHub等にリポジトリを作成しよう
    ⇒今回はDockerHubを用いる

リポジトリ作成
    1. DockerHubにサインイン
    2. Create Repository
    3. Repository-nameを設定
    4. create

imageを送信
    1. docker login -u <USERNAME>でログイン
    2. docker tag <local_image> <repository_tag>で新しい名前を付ける
    3. docker push <USERNAME>/getting-startedで新しい名前を付けたイメージをレジストリに送信
        ※tagnameは省略した場合latest(最新)となる
        ※docker-hubのリポジトリ名とイメージ名は同一なのか?
        ※つまりpushする際には、リモートとローカルのリポジトリ名が同一でないといけなそう。だからtagをつけるのか

新しいイメージを実行
    1. https://labs.play-with-docker.com/を用いる
    2. 新しいインスタンスを立ち上げる
    3. $docker run -dp 0.0.0.0:3000:3000 <USERNAME>/getting-started

    補足:
        ・0.0.0.0はホスト上のすべてのインターフェースでlistenする
        ・ つまり、同一ネットワーク上の別ホストからアクセス可能
        ・127.0.0.1は他ホストからはアクセス不可。コンテナのlocalhostと、別のコンテナのlocalhostは異なるので
        ・今回ホストはDockerコンテナなので、ローカルマシン（コンテナの外）からアクセスしたい場合、0.0.0.0の方がいい。
        ・なお、0.0.0.0が宛先となっていた場合、これは書き換えられる。

Next:
    再起動してもデータを保持できる方法について学ぶ
```

## データベースの保持
```
コンテナのファイルシステム
    ・コンテナはファイルの作成、更新、削除するための権限を持っている
    ・同じイメージを使っていたとしても、コンテナ内のファイルシステムに対する変更は、他のコンテナからは閲覧できない

実際に確認する
    1. ubuntuコンテナを起動し、/data.txtという名前のファイルを作成。1~10000までのランダムな数を入れる
    $ docker run -d ubuntu bash -c "shuf -i 1-10000 -n 1 -o /data.txt && tail -f /dev/null"
        ※bashシェルを開始する
        ※-c <string>: コマンドをstringから読み込み、bashで実行
        ※shuf:入力行をシャッフルする
        ※-i 1-10000: 範囲
        ※-n 1: 行数(1行)
        ※ -o /data.txt: 結果を/data.txtに書きだす
        ※tail -f : ファイルが変更されてもtailをし続ける
        ※tail -f /dev/null: 終了されないのでコンテナが起動し続ける
    
    2. コマンドラインからコンテナにアクセスする
    docker exec <CONTAINER ID> cat /data.txt

    3. 他のubuntuコンテナ(同じイメージ)を起動しても同じファイルは見えない
    $ docker run -it ubuntu ls /
    ⇒ファイルを書きだしたのは1つ目のコンテナのスクラッチ領域なので

コンテナのボリューム
    ・つまり、コンテナを削除したら、それらの変更は消失する
    ・そのためボリュームを使う
    ・ボリューム：コンテナ内で指定したファイルシステムのパスをホストマシン上へと接続できる機能を備えたもの
    ・コンテナ内にディレクトリをマウントすれば、ディレクトリに対する変更がホストマシンから閲覧できる。
    ・また同一ディレクトリをマウントすれば再起動後も同じファイルが見える

todoデータを保持するためには？
    ・今回の場合、デフォルトで/etc/todo/todb.dbに保存される
    ・ホスト上のボリュームにtodo.dbを置いておき、再起動した際に、データを保管するディレクトリに取り付ける（マウントする）

ボリュームの作成とコンテナの起動
    1. docker volume create <volume_name> でボリュームを作成
    2. todoアプリのコンテナを作り直す(現在は保存するボリュームを使わずに起動しているので)
    3. todoアプリのコンテナを起動する際に、ボリュームのマウントを指定する--mountオプションを追加する
    $ docker run -dp 127.0.0.1:3000:3000 --mount type=volume,src=todo-db,target=/etc/todos getting-started
    4. 実際にコンテナを削除、起動し保存されていることを確認する
    -> OK

    ボリュームの格納位置に関する補足
        ・WSL2を用いている場合、注意が必要
        docker volume inspect todo-dbで格納位置を尋ねたところ、こう返答があった
        [
            {
                "CreatedAt": "2023-10-13T07:30:33Z",
                "Driver": "local",
                "Labels": null,
                "Mountpoint": "/var/lib/docker/volumes/todo-db/_data",
                "Name": "todo-db",
                "Options": null,
                "Scope": "local"
            }
        ]
        しかし、実際には格納先は以下の通りであった
        "\\wsl.localhost\docker-desktop-data\data\docker\volumes\todo-db\_data\todo.db"

Next
    現在は変更を加えるたび、再構築をしている
    これを改善する
```

##　バインドマウントを使う


# 便利コマンド
```
docker image prune
    dangling状態のimageのみ削除する

docker rm -f <CONTAINER ID>
    コンテナを削除
```