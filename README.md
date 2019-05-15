# トライアンフ ワークフローシステム フロントエンド

## ローカルで起動

### 準備
予めNode.js, npmはbrew経由等でインストールしておく

### ソースコード取得
```bash
$ git clone https://github.com/topgate/tr21-offside-client
```

### 必要パッケージのインストール  
この`tr21-offside-client`ディレクトリの階層と`server`ディレクトリの階層で以下のコマンドを実施する
コマンド実行後、各階層に`node_modules`フォルダが生成されます  

```bash
$ npm install
```

### コードフォーマッターのインストール
以下のコマンドで`prettier`をインストールします  

```bash
$ npm install --global prettier
```

インストールが完了したら、`prettier`のプラグインもインストールし、`IntelliJ`を再起動させる  
再起動し終わったら、`IntelliJ`の設定に`prettier`のインストール先のパスを登録する  
※`windows`の場合、`IntelliJ`を再起動する前に起動しているサービスを止める必要がある  

### 単体の起動
```bash
$ ng serve
```

`http://localhost:4200`で確認できる(データは取得できない)  

### モックサーバとの連携
以下のコマンドを実行するとモックサーバを経由してローカルで起動することができる  

```bash
$ ng start
$ ng run start:mock-local3000 
```

### APIサーバとの連携
ローカルのAPIサーバと連携してデータを読み書きする場合は、APIサーバをポート8080で起動させて以下のコマンドを実行する  

```bash
$ ng serve --proxy-config proxy.conf.json
```

GAEにデプロイ済みのAPIサーバにアクセスする場合は proxy.conf.json 内のURLをGAEのURLに書き換える

## ビルド

```bash
$ ng build
```

直下の`/dist`配下にビルドされる

# GAEデプロイ

## 運用ビルド

```bash
$ ng build --prod
```

直下の`/dist`配下にビルドされる

## APIサーバへの組み込み
運用ビルドで出来た /dist 以下のファイルをAPIサーバのソースディレクトリの src/main/resources/static にコピーする

コピー後APIサーバのデプロイを行う

```bash
$ gradle appengineDeploy
```

GAEの[URL](https://workflow-dot-tryumph21-wf-development-pj.appspot.com/ )にアクセスしてページの表示を確認する

## 自分だけのバージョンでGAEにデプロイする方法 (例：開発環境のデプロイ)

- APIの向け先を変更  
`src/environments/environments.dev.ts`ファイルのAPIの向け先を`dev2`のように修正する  

```src/environments/environments.dev.ts
apiBasePath: 'https://dev2-dot-tryumph21-wf-development-pj.appspot.com/api/v1',
```

-  サービスを変更  
`server/app.yaml`ファイルのサービスを上記と合わせる  

```server/app.yaml
service: dev2
```

- ビルドする  
```bash
$ npm run build:gae-dev
```

- デプロイする  
```bash
$ npm run deploy
```

- 確認する  
上記の場合、`https://dev3-dot-tryumph21-wf-development-pj.appspot.com/company` にデプロイされているのでアクセスして確認する。
