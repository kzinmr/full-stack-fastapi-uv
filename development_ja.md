# FastAPIプロジェクト - 開発

## Docker Compose

* Docker Composeでローカルスタックを起動します：

```bash
docker compose watch
```

* ブラウザを開いて以下のURLにアクセスできます：

Dockerでビルドされたフロントエンド（パスに基づいてルーティング）: http://localhost:5173

OpenAPIベースのJSON形式のバックエンドAPI: http://localhost:8000

OpenAPIバックエンドによる自動対話型ドキュメント（Swagger UI）: http://localhost:8000/docs

データベース管理用ウェブインターフェース（Adminer）: http://localhost:8080

プロキシによるルート処理の確認用Traefik UI: http://localhost:8090

**注意**: スタックを初めて起動する際、準備が整うまで1分ほどかかる場合があります。バックエンドがデータベースの準備を待ち、すべての設定を行うためです。ログを確認して進捗を監視できます。

ログを確認するには（別のターミナルで）以下を実行します：

```bash
docker compose logs
```

特定のサービスのログを確認するには、サービス名を追加します。例：

```bash
docker compose logs backend
```

## ローカル開発

Docker Composeファイルは、各サービスが`localhost`の異なるポートで利用できるように設定されています。

バックエンドとフロントエンドは、ローカル開発サーバーと同じポートを使用します。つまり、バックエンドは`http://localhost:8000`、フロントエンドは`http://localhost:5173`です。

これにより、Docker Composeサービスを停止してローカル開発サービスを起動しても、同じポートを使用するため、すべてが正常に動作し続けます。

例えば、Docker Composeの`frontend`サービスを停止し、別のターミナルで以下を実行できます：

```bash
docker compose stop frontend
```

そしてローカルフロントエンド開発サーバーを起動します：

```bash
cd frontend
npm run dev
```

または`backend` Docker Composeサービスを停止できます：

```bash
docker compose stop backend
```

そしてバックエンドのローカル開発サーバーを実行できます：

```bash
cd backend
fastapi dev app/main.py
```

## `localhost.tiangolo.com`でのDocker Compose

Docker Composeスタックを起動すると、デフォルトで`localhost`を使用し、各サービス（バックエンド、フロントエンド、Adminerなど）に異なるポートが割り当てられます。

本番環境（またはステージング環境）にデプロイする際は、各サービスが異なるサブドメインにデプロイされます（例：バックエンドは`api.example.com`、フロントエンドは`dashboard.example.com`）。

[デプロイメント](deployment.md)のガイドでは、設定されたプロキシであるTraefikについて説明しています。これはサブドメインに基づいて各サービスにトラフィックを転送する役割を果たします。

ローカルでこれらが正常に動作しているかテストしたい場合は、ローカルの`.env`ファイルを編集し、以下のように変更できます：

```dotenv
DOMAIN=localhost.tiangolo.com
```

これはDocker Composeファイルでサービスのベースドメインを設定するために使用されます。

Traefikはこれを使用して、`api.localhost.tiangolo.com`へのトラフィックをバックエンドに、`dashboard.localhost.tiangolo.com`へのトラフィックをフロントエンドに転送します。

`localhost.tiangolo.com`ドメイン（およびそのすべてのサブドメイン）は`127.0.0.1`を指すように設定された特別なドメインです。これによりローカル開発に使用できます。

更新後、再度以下を実行します：

```bash
docker compose watch
```

本番環境などにデプロイする場合、メインのTraefikはDocker Composeファイルの外部で設定されます。ローカル開発では、`docker-compose.override.yml`に含まれるTraefikを使用して、`api.localhost.tiangolo.com`や`dashboard.localhost.tiangolo.com`などのドメインが期待通りに動作することをテストできます。

## Docker Composeファイルと環境変数

スタック全体に適用される設定を含むメインの`compose.yml`ファイルがあり、`docker compose`によって自動的に使用されます。

また、開発用のオーバーライドを含む`compose.override.yml`もあります。例えば、ソースコードをボリュームとしてマウントするなどの設定が含まれています。これは`docker compose`によって自動的に使用され、`compose.yml`の上にオーバーライドを適用します。

これらのDocker Composeファイルは、コンテナに注入される環境変数として設定を含む`.env`ファイルを使用します。

また、`docker compose`コマンドを呼び出す前にスクリプトで設定された環境変数からいくつかの追加設定も使用します。

変数を変更した後は、必ずスタックを再起動してください：

```bash
docker compose watch
```

## .envファイル

`.env`ファイルには、すべての設定、生成されたキーやパスワードなどが含まれています。

プロジェクトがパブリックな場合など、ワークフローによってはGitから除外したい場合があります。その場合、プロジェクトのビルドやデプロイ時にCIツールがこれを取得する方法を確保する必要があります。

一つの方法として、各環境変数をCI/CDシステムに追加し、`compose.yml`ファイルを更新して`.env`ファイルを読み取る代わりに特定の環境変数を読み取るようにすることができます。

## プリコミットとコードリンティング

コードのリンティングとフォーマットには[pre-commit](https://pre-commit.com/)というツールを使用しています。

インストールすると、gitでコミットする直前に実行されます。これにより、コードが一貫性を保ち、フォーマットされていることがコミット前に確認されます。

プロジェクトのルートに`.pre-commit-config.yaml`という設定ファイルがあります。

#### 自動実行のためのpre-commitのインストール

`pre-commit`はすでにプロジェクトの依存関係の一部ですが、グローバルにインストールすることもできます。その場合は[公式のpre-commitドキュメント](https://pre-commit.com/)に従ってください。

`pre-commit`ツールがインストールされ利用可能になったら、各コミットの前に自動的に実行されるようにローカルリポジトリに「インストール」する必要があります。

`uv`/`uvx`を使用する場合、以下のようにできます：

```bash
❯ uv tool install pre-commit
❯ uvx pre-commit install
pre-commit installed at .git/hooks/pre-commit
```

これで、コミットしようとする際（例：`git commit`）に、pre-commitが実行され、コミットしようとしているコードをチェックおよびフォーマットし、そのコードを再度gitでステージングするよう求めます。

その後、修正されたファイルを再度`git add`でステージングし、コミットできます。

#### pre-commitフックの手動実行

`pre-commit`を全ファイルに対して手動で実行することもできます。`uvx`を使用して以下のように実行できます：

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

本番環境やステージング環境のURLは、これらと同じパスを使用しますが、独自のドメインを使用します。

### 開発用URL

ローカル開発用のURL。

フロントエンド: http://localhost:5173

バックエンド: http://localhost:8000

自動対話型ドキュメント（Swagger UI）: http://localhost:8000/docs

自動代替ドキュメント（ReDoc）: http://localhost:8000/redoc

Adminer: http://localhost:8080

Traefik UI: http://localhost:8090

### `localhost.tiangolo.com`設定時の開発用URL

ローカル開発用のURL。

フロントエンド: http://dashboard.localhost.tiangolo.com

バックエンド: http://api.localhost.tiangolo.com

自動対話型ドキュメント（Swagger UI）: http://api.localhost.tiangolo.com/docs

自動代替ドキュメント（ReDoc）: http://api.localhost.tiangolo.com/redoc

Adminer: http://localhost.tiangolo.com:8080

Traefik UI: http://localhost.tiangolo.com:8090
