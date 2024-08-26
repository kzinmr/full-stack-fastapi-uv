# バックエンド

## 要件

- [Docker](https://www.docker.com/)
- Python パッケージと環境管理のための [uv](https://docs.astral.sh/uv/)

## ローカル開発

- Docker Compose でスタックを起動します：

```bash
docker compose up -d
```

- ブラウザを開いて、以下の URL にアクセスできます：
  - フロントエンド: http://localhost
  - バックエンド（OpenAPI ベース）: http://localhost/api/
  - Swagger UI（OpenAPI バックエンド由来）: http://localhost/docs
  - Adminer（DB 管理画面）: http://localhost:8080
  - Traefik UI（プロキシによるルーティング処理）: http://localhost:8090

**注意**: スタックを初めて起動する際、準備が整うまで 1 分ほどかかる場合があります。バックエンドがデータベースの準備を待ち、すべてを設定する間、ログを確認して監視できます。

ログを確認するには、次のコマンドを実行します：

```bash
docker compose logs
```

特定のサービスのログを確認するには、サービス名を追加します：

```bash
docker compose logs backend
```

Docker が localhost 上で実行されていない場合（上記の URL が機能しない場合）、Docker が実行されている IP またはドメインを使用する必要があります。

## バックエンドのローカル開発の詳細

### 一般的なワークフロー

デフォルトでは、依存関係は [uv](https://docs.astral.sh/uv/) で管理されています。uv をインストールしてください。

`./backend/`ディレクトリから、以下のコマンドですべての依存関係をインストールできます：

```console
$ uv sync
```

その後、作成された新しい仮想環境を有効化できます：

```console
$ source .venv/bin/activate
```

エディタが正しい Python 仮想環境を使用していることを確認してください。

`./backend/app/models.py` でデータと SQL テーブル用の SQLModel モデルを変更または追加し、 `./backend/app/api/` で API エンドポイントを、 `./backend/app/crud.py` で CRUD（Create, Read, Update, Delete）ユーティリティを変更します。

### VS Code デバッガー & テスト

バックエンドを VS Code デバッガーで実行するための設定が用意されているので、ブレークポイントを使用したり、変数を一時停止して探索したりできます。

また、VS Code の Python テストタブを通じてテストを実行できるよう設定されています。

### Docker Compose オーバーライド

開発中、ローカル開発環境にのみ影響を与える Docker Compose 設定を`compose.override.yml`ファイルで変更できます。

このファイルへの変更は、ローカル開発環境にのみ影響し、本番環境には影響しません。そのため、開発ワークフローに役立つ「一時的な」変更を追加できます。

#### ライブリロードサーバーによるデバッグ

バックエンドコードのディレクトリは Docker の「ホストボリューム」としてマウントされ、変更したコードをコンテナ内のディレクトリにリアルタイムでマッピングします。これにより、Docker イメージを再ビルドすることなく、変更をすぐにテストでき、開発中は非常に迅速な反復が可能になります。

エラーのあるコード実行がされるとコンテナが停止しますが、以下のコマンドを再実行することでコンテナを再起動できます：

```console
$ docker compose up -d
```

これを手動で行う代わりに、 `--reload` オプションと `sleep infinity ` コマンドを組み合わせることで、便利なライブリロードサーバーの設定が手に入ります。コメントアウトされた`command`オーバーライド(`sleep infinity `)を解除することで、バックエンドコンテナは「何もしない」プロセスを実行しますが、コンテナを生きたままにします。これにより、実行中のコンテナ内に入り、コマンドを実行できます。

`bash`セッションでコンテナ内に入るには、以下のコマンドを実行します：

```console
$ docker compose up -d
$ docker compose exec backend bash
```

結果、コンテナ内の`bash`セッションにおいて、`root`ユーザーとして`/app`ディレクトリにいます。このディレクトリには `/app/app` という別のディレクトリがあり、そこがコンテナ内のコードが存在する場所です。

そこで、`/start-reload.sh`スクリプトを使用してデバッグ用ライブリロードサーバーを実行できます。`start-reload.sh` の内容は以下のコマンドです:

```bash
...
exec uvicorn --reload --host $HOST --port $PORT --log-level $LOG_LEVEL "$APP_MODULE"
```

コンテナ内からこのスクリプトを実行する：

```console
root@7f2607af31c3:/app# bash /start-reload.sh
```

ただし、変更を検出せずに構文エラーがある場合、エラーで停止します。しかし、(`sleep infinity` command により) コンテナはまだ生きていて、Bash セッション内にいるので、エラーを修正した後、同じコマンドを実行して素早く再起動できます。

### バックエンドテスト

バックエンドをテストするには、次のコマンドを実行します：

```console
$ bash ./scripts/test.sh
```

テストは Pytest で実行されます。`./backend/app/tests/`でテストを修正および追加してください。

GitHub Actions を使用している場合、テストは自動的に実行されます。

#### 実行中のスタックのテスト

スタックがすでに起動していて、テストだけを実行したい場合は、次のコマンドを使用できます：

```bash
docker compose exec backend bash /app/tests-start.sh
```

この`/app/tests-start.sh`スクリプトは、スタックの残りの部分が実行されていることを確認した後、`pytest`を呼び出すだけです。`pytest`に追加の引数を渡す必要がある場合、それらをこのコマンドに渡すと転送されます。

例えば、最初のエラーで停止する方法は：

```bash
docker compose exec backend bash /app/tests-start.sh -x
```

#### テストカバレッジ

テストが実行されると、`htmlcov/index.html`ファイルが生成されます。ブラウザでこのファイルを開いて、テストのカバレッジを確認できます。

### マイグレーション

ローカル開発中、アプリディレクトリはコンテナ内のボリュームとしてマウントされているため、コンテナ内で`alembic`コマンドを使用してマイグレーションを実行することもでき、マイグレーションコードはコンテナ内外のアプリディレクトリに存在します。そのため、git リポジトリに追加できます。

モデルを変更するたびに、モデルの「リビジョン」を作成し、そのリビジョンでデータベースを「アップグレード」することを確認してください。これにより、データベース内のテーブルが更新されます。そうしないと、アプリケーションにエラーが発生します。

- バックエンドコンテナで bash セッションを開始します：

```console
$ docker compose exec backend bash
```

- Alembic はすでに`./backend/app/models.py`から SQLModel モデルをインポートするように設定されています。

- モデルを変更した後（例えば、列を追加した後）、コンテナ内でリビジョンを作成します：

```console
$ alembic revision --autogenerate -m "Add column last_name to User model"
```

- alembic ディレクトリで生成されたファイルを git リポジトリにコミットします。

- リビジョンを作成した後、データベースでマイグレーションを実行します（これが実際にデータベースを変更します）：

```console
$ alembic upgrade head
```

#### マイグレーションを使用したくない場合

マイグレーションをまったく使用したくない場合は、`./backend/app/core/db.py`ファイルの以下の行のコメントを解除します：

```python
SQLModel.metadata.create_all(engine)
```

そして、`prestart.sh`ファイルの以下の行をコメントアウトします：

```console
$ alembic upgrade head
```

デフォルトのモデルから始めたくない場合や、最初からモデルを削除/修正したい場合で、以前のリビジョンを持たない場合は、`./backend/app/alembic/versions/`以下のリビジョンファイル（`.py` Python ファイル）を削除できます。その後、上記で説明したように最初のマイグレーションを作成します。

## メールテンプレート

メールテンプレートは `./backend/app/email-templates/` にあります。ここには `build` と `src` の 2 つのディレクトリがあります。`src` ディレクトリには、最終的なメールテンプレートを構築するために使用されるソースファイルが含まれています。`build` ディレクトリには、アプリケーションで使用される最終的なメールテンプレートが含まれています。

続行する前に、VS Code に [MJML 拡張機能](https://marketplace.visualstudio.com/items?itemName=attilabuti.vscode-mjml) がインストールされていることを確認してください。

MJML 拡張機能をインストールしたら、`src` ディレクトリに新しいメールテンプレートを作成できます。新しいメールテンプレートを作成し、`.mjml` ファイルをエディタで開いた後、`Ctrl+Shift+P` でコマンドパレットを開き、`MJML: Export to HTML` を検索します。これにより `.mjml` ファイルが `.html` ファイルに変換され、build ディレクトリに保存できます。
