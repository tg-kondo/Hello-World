# Hello-World

## GitHubテスト勉強用です。


# Edge TPU ドライブレコーダー システム
Edge TPU と IoT Core を組み合わせて端末ごとのドライブレコーダーの情報を保存。

ドライブレコーダーのリアルタイムな情報をWeb画面より確認できる。

# GCP
### プロジェクト名
本番環境: toyota-edge-tpu-prod

開発環境: toyota-edge-tpu-dev

# Gitワークフロー
## 機能開発手順
1. master ブランチから feature/* ブランチを作成する。
2. feature/* ブランチから master ブランチに対して Pull Request を作成する。
3. 開発完了後、 PR のレビューを依頼する。
4. レビュー完了後、 develop ブランチに PR をマージし、feature/* ブランチは削除する。
# システム構成図
![システム構成図](https://github.com/tg-kondo/git-tutorial/blob/test/image.png?raw=true)

# 下準備
## Cloud SDKの導入
### Cloud SDKのダウンロードとインストール
下記ページより自分のOS環境(32bit or 64bit)に合うバージョンのアーカイブファイルをダウンロードします。

- mac : https://cloud.google.com/sdk/docs/quickstart-mac-os-x
- linux : https://cloud.google.com/sdk/docs/quickstart-linux

自分のOS環境がわからない場合はターミナルから下記コマンドで確認します。

```
uname -a
```

「i386」「i686」「x86」といった文字が確認できる場合は32bit。

「x86_64」「amd64」といった文字が確認できる場合は64bit。


## Javaの環境構築
### JDKのインストール
 Oracle の [ダウンロードページ](https://www.oracle.com/technetwork/java/javase/downloads/java-archive-javase8-2177648.html) にアクセスし、利用規約に同意する(Accept License Agreement)ラジオボックスにチェックを入れて、端末にあったファイルをクリックしてインストーラをダウンロードします。
 
ダウンロードしたインストーラを起動してインストールが完了したら以下のコマンドでバージョンを確認します。

```
java -version
```

インストールされた Java のバージョンが表示されたら完了です。
`$JAVA_HOME` の設定も行う。

ex) /Library/Java/JavaVirtualMachines/jdk1.8.0_201.jdk/Contents/Home


### Mavenのインストール
以下のコマンドでMavenをインストールする。

```
brew install maven
```

`mvn -v` でバージョンを確認する。


## Goの環境構築
以下のコマンドでGoをインストールする。
```
brew install go
```

`go version` でバージョンを確認する。

# システム構成フロー
## IoT Core の準備
GCPコンソール > IoT Core画面でレジストリを作成する。
プロトコルは `MQTT` と `HTTP`
この時、Pub/Subトピックを新規作成。（デバイス状態のトピックは省略。）
端末の追加で、自端末で公開鍵を作成し、登録する。

#### 公開鍵の作成方法
```
openssl req -x509 -newkey rsa:2048 -keyout rsa_private.pem -nodes  -out rsa_cert.pem -subj "/CN=unused"
```

## BigQuery の準備
ぷプロジェクトのBigQueryからデータセット（iot）作成し、以下Jsonファイルを用意して以下のコマンドを実行する。

<details><summary>jsonコード</summary><div>

\```schema.json
[
{
"description": "",
"name": "date",
"type": "TIMESTAMP",
"mode": "NULLABLE"
},
{
"description": "",
"name": "jsondata",
"type": "STRING",
"mode": "NULLABLE"
},
{
"description": "",
"name": "longitude",
"type": "STRING",
"mode": "NULLABLE"
},
{
"description": "",
"name": "latitude",
"type": "STRING",
"mode": "NULLABLE"
},
{
"description": "",
"name": "meta",
"type": "STRING",
"mode": "NULLABLE"
},
{
"description": "",
"name": "feature",
"type": "STRING",
"mode": "NULLABLE"
},
{
"description": "",
"name": "deviceId",
"type": "STRING",
"mode": "NULLABLE"
}
]
\```
</div></details>



- コマンド（環境に合わせて「toyota-edge-tpu-dev」は書き換える）

```
bq mk --table toyota-edge-tpu-dev:iot.edge schema.json
```

## WebUIの準備
以下のファイルを環境に合わせて修正する。

`webui/main.py`

IoT Core準備で作成したレジストリ、リージョン、プロジェクトIDを更新する。

```
APPID = "toyota-edge-tpu-dev"  # os.getenv('APPLICATION_ID')
REGION = "asia-east1"
REGISTRY = "btf-iot-core-test"
```

`webui/templates/index.html`

86行目にある以下の端末名をIoT Coreで登録した端末名に修正する。

```
<ul class="mdl-menu mdl-menu--bottom-left mdl-js-menu mdl-js-ripple-effect" for="demo-menu-lower-left" id="deviceList">
<li class="mdl-menu__item">iot-mako</li>
<li class="mdl-menu__item">iot-kondo</li>
</ul>
```

gcloudコマンドが扱える環境で以下のコマンドを実行し、GAEを立ち上げる。

```
cd webui
gcloud app deploy
```

- WebUI画面

https://toyota-edge-tpu-dev.appspot.com/

## Dataflow の準備
ターミナル等で以下のディレクトリに移動してシェルスクリプトを実行する。

```
cd dataflow/scripts
./deploy.sh {IoT Coreで作成した時のsubscription名}
```

ex) 
`./deploy.sh btf-test-subscription`

実行すると、Dataflowがストリーミングでジョブが立ち上がる


## Edge起動方法
以下を実行するとランダムなpoint情報のJSONデータが流れ、Dataflowを経由してFirestoreとBigQueryにデータが保存されます。

Firestoreに保存されたデータをWebUI画面からリアルタイムに確認することができます。

WebUIからモードを切り替えることによってコンフィグファイルが書き換えられます。

### edgeディレクトリ直下に以下のファイルを準備
- roots.pem
- rsa_cert.pem
- rsa_private.pem 

roots.pemはこちらのリンクからダウンロードできます。

https://pki.google.com/roots.pem

rsaファイルはIoT Core準備で登録した端末の公開鍵です。

ex) 対応する引数を指定してGoファイルを実行
```
go run main.go \
-device=btf-iot-device-test \
-project=toyota-edge-tpu-dev \
-registry=btf-iot-core-test \
-private_key=rsa_private.pem \
-ca_certs=root.pem \
-region=asia-east1
```
