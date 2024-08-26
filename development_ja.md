# 開発

## カスタムドメインを使用した `localhost` での開発

`localhost`以外のドメインを使用したい場合があるかもしれません。例えば、サブドメインを必要とするクッキーで問題が発生し、Chrome が`localhost`の使用を許可しない場合などです。

その場合、2 つの選択肢があります：

1. 後述の**カスタム IP を使用した開発**の指示に従ってシステムの`hosts`ファイルを修正する
2. `localhost.tiangolo.com`を使用する。これは`localhost`（IP `127.0.0.1`）とそのすべてのサブドメインを指すように設定されています。

プロジェクト生成時にデフォルトの CORS 有効ドメインを使用した場合、`localhost.tiangolo.com`は許可されるように設定されています。これは、`.env`ファイルの`BACKEND_CORS_ORIGINS`変数のリストで指定されています。

スタック内で設定するには、以下の**開発用"ドメイン"の変更**セクションに従い、ドメイン`localhost.tiangolo.com`を使用してください。

これらの手順を実行後、http://localhost.tiangolo.com を開くことができ、`localhost`のスタックによって提供されます。

利用可能な URL の一覧は最後のセクションで確認できます。

## カスタム IP を使用した開発

Docker を`127.0.0.1`（`localhost`）以外の "カスタム IP アドレス" で実行している場合、追加の手順が必要です。これは、カスタム仮想マシンを実行している場合や、Docker がネットワーク上の別のマシンにある場合に該当します。

この場合、偽のローカルドメイン（`dev.example.com`など）を使用し、そのドメインがカスタム IP（例：`192.168.99.150`）によって提供されていると、コンピュータに認識させる必要があります。

そのようなカスタムドメインがある場合、`.env`ファイルの`BACKEND_CORS_ORIGINS`変数のリストに追加する必要があります。

- 管理者権限でテキストエディタを使用して`hosts`ファイルを開きます：

  - **Mac と Linux 向け注意**: `hosts`ファイルは通常`/etc/hosts`にあります。
  - **Windows 向け注意**: Windows の場合、`c:\Windows\System32\Drivers\etc\`ディレクトリにあります。

- 既存の内容に加えて、`カスタムIP ダミーローカルドメイン`（例：`192.168.99.150 dev.example.com`）を含む新しい行を追加します。

- ファイルを保存します。
  - **Windows 向け注意**: ファイルを拡張子なしの「すべてのファイル」として保存してください。

これにより、コンピュータはダミーのローカルドメインがそのカスタム IP によって提供されていると認識し、ブラウザでその URL を開くと、`dev.example.com`に行くよう要求された際に、実際にはコンピュータ上で動作しているローカルサーバーと直接通信します。

スタック内で設定するには、以下の**開発用"ドメイン"の変更**セクションに従い、ドメイン`dev.example.com`を使用してください。

これらの手順を実行後、 `http://dev.example.com` を開くことができ、`192.168.99.150`のスタックによって提供されます。

利用可能な URL の一覧は最後のセクションで確認できます。

## 開発用"ドメイン"の変更

ローカルスタックを`localhost`以外のドメインで使用する必要がある場合、使用するドメインが、スタックが設定されている IP を指していることを確認する必要があります。

例えば、API docs（Swagger UI）が API の場所を認識するように、Docker Compose 設定を簡素化するには、compose に API のドメインを開発用に使用していることを知らせる必要があります。これは`DOMAIN` 環境変数に指定することで Docker Compose が認識し実現されます。

- `./.env` の以下の行を：

```
DOMAIN=localhost
```

- 使用するドメインに変更します。例：

```
DOMAIN=localhost.tiangolo.com
```

その後、以下のコマンドでスタックを再起動できます：

```bash
docker compose up -d
```

対応する利用可能な URL は最後のセクションで確認できます。

## Docker Compose ファイルと環境変数

スタック全体に適用される全ての設定を含むメイン `compose.yml` ファイルがあり、`docker compose`によって自動的に使用されます。

また、開発用のオーバーライド（例：ソースコードをボリュームとしてマウントするなど）を含む`compose.override.yml`もあります。これは`docker compose`によって自動的に使用され、`compose.yml`の上にオーバーライドを適用します。

これらの Docker Compose ファイルは、コンテナに注入される環境変数設定を含む`.env`ファイルを使用します。

また、`docker compose`コマンドを呼び出す前にスクリプトで設定された追加の環境変数設定も使用します。

## .env ファイル

`.env`ファイルは、すべての設定、生成されたキーやパスワードなどを含むファイルです。

ワークフローによっては、例えばプロジェクトが公開されている場合、このファイルを Git から除外したい場合があります。その場合、プロジェクトのビルドやデプロイ時に CI ツールがこのファイルを取得する方法を確保する必要があります。

一つの方法として、各環境変数を CI/CD システムに追加し、`.env`ファイルを読み取る代わりに特定の環境変数を読み取るように`compose.yml`ファイルを更新することができます。

### pre-commit と lint

コードの lint と format には[pre-commit](https://pre-commit.com/)というツールを使用しています。

インストールすると、git でコミットする直前に実行されます。これにより、コードがコミットされる前に一貫性があり、フォーマットされていることが保証されます。

プロジェクトのルートに設定ファイル`.pre-commit-config.yaml`があります。

#### 自動実行のための pre-commit のインストール

`pre-commit`はすでにプロジェクトの依存関係の一部ですが、[公式の pre-commit ドキュメント](https://pre-commit.com/)に従ってグローバルにインストールすることもできます。

`pre-commit`ツールがインストールされ利用可能になったら、各コミットの前に自動的に実行されるようにローカルリポジトリに「インストール」する必要があります。

uv/uvx を使用する場合、以下のように実行できます：

```bash
❯ uvx pre-commit install
pre-commit installed at .git/hooks/pre-commit
```

これで、例えば以下のようにコミットしようとすると：

```bash
git commit
```

...pre-commit が実行され、コミットしようとしているコードをチェックしフォーマットし、修正されたコードを再度 git で追加（ステージング）するよう求めます。

その後、修正/修正されたファイルを再度`git add`し、コミットできます。

#### pre-commit フックの手動実行

すべてのファイルに対して`pre-commit`を手動で実行することもできます。 uv/uvx を使用して以下のように実行できます：

```bash
❯ uvx pre-commit run --all-files
check for added large files..............................................Passed
check toml...............................................................Passed
check yaml...............................................................Passed
ruff.....................................................................Passed
ruff-format..............................................................Passed
eslint...................................................................Passed
prettier.................................................................Passed
```

## URL

本番環境やステージング環境の URL は、同じパスを使用しますが、自身のドメインを使用します。

### 開発用 URL

フロントエンド: http://localhost

バックエンド: http://localhost/api/

Swagger UI: http://localhost/docs

ReDoc: http://localhost/redoc

Adminer: http://localhost:8080

Traefik UI: http://localhost:8090

### カスタムドメインを使用した開発用 URL

フロントエンド: http://localhost.tiangolo.com

バックエンド: http://localhost.tiangolo.com/api/

Swagger UI: http://localhost.tiangolo.com/docs

ReDoc: http://localhost.tiangolo.com/redoc

Adminer: http://localhost.tiangolo.com:8080

Traefik UI: http://localhost.tiangolo.com:8090
