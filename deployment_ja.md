# デプロイ

Docker Compose を使用して、サーバーにプロジェクトをデプロイできます。

このプロジェクトでは、外部世界とのコミュニケーションと HTTPS 証明書を処理する [Traefik](https://traefik.io/) プロキシがあることを想定しています。

CI/CD システムを使用して自動的にデプロイすることができます。GitHub Actions で行うための設定がすでに用意されています。

ただし、最初にいくつかの設定を行う必要があります。🤓

## 準備

- リモートサーバーを用意し、利用可能な状態にします。
- ドメインの DNS レコードを設定し、作成したサーバーの IP を指すようにします。
- ドメインのワイルドカードサブドメインを設定し、異なるサービスに対して複数のサブドメインを持てるようにします（例：`*.fastapi-project.example.com`）。
  - これは`traefik.fastapi-project.example.com`、`adminer.fastapi-project.example.com`などの異なるコンポーネントにアクセスする際に役立ちます。
  - また、`staging.fastapi-project.example.com`、`staging.adminer.fastapi-project.example.com`などのステージング環境にも使用できます。
- リモートサーバーに[Docker](https://docs.docker.com/engine/install/)をインストールし、設定します（Docker Desktop ではなく、Docker Engine です）。

## パブリック Traefik

incoming 接続と HTTPS 証明書を処理するために Traefik プロキシが必要です。

次のステップは一度だけ行う必要があります。

### Traefik Docker Compose

- Traefik の Docker Compose ファイルを保存するリモートディレクトリを作成：

```bash
mkdir -p /root/code/traefik-public/
```

Traefik の Docker Compose ファイルをサーバーにコピーするために、ローカルターミナルで`rsync`コマンドを実行すします：

```bash
rsync -a compose.traefik.yml root@your-server.example.com:/root/code/traefik-public/
```

### Traefik パブリックネットワーク

この Traefik は、スタックとの通信のために`traefik-public`という名前の Docker "パブリックネットワーク" を期待します。

これにより、外部世界との通信（HTTP と HTTPS）を処理する単一のパブリック Traefik プロキシがあり、その背後に、同じサーバー上にあっても、異なるドメインを持つ複数のスタックを持つことができます。

`traefik-public`という名前の Docker "パブリックネットワーク" を作成するには、リモートサーバーで次のコマンドを実行します：

```bash
docker network create traefik-public
```

### Traefik 環境変数

Traefik の Docker Compose ファイルは、開始する前にターミナルでいくつかの環境変数が設定されていることを期待します。リモートサーバーで次のコマンドを実行することでこれを行えます。

- HTTP Basic 認証用のユーザー名を作成します。例：

```bash
export USERNAME=admin
```

- HTTP Basic 認証用のパスワードを環境変数に設定します。例：

```bash
export PASSWORD=changethis
```

- openssl を使用して HTTP Basic 認証用のパスワードの"ハッシュ化"バージョンを生成し、環境変数に保存します：

```bash
export HASHED_PASSWORD=$(openssl passwd -apr1 $PASSWORD)
```

ハッシュ化されたパスワードが正しいことを確認するには、次のように出力できます：

```bash
echo $HASHED_PASSWORD
```

- サーバーのドメイン名を環境変数に設定します。例：

```bash
export DOMAIN=fastapi-project.example.com
```

- Let's Encrypt 用のメールアドレスを環境変数に設定します。例：

```bash
export EMAIL=admin@example.com
```

**注意**: `@example.com`のメールアドレスは機能しないため、別のメールアドレスを設定する必要があります。

### Traefik の Docker Compose を開始

リモートサーバーで Traefik の Docker Compose ファイルをコピーしたディレクトリに移動します：

```bash
cd /root/code/traefik-public/
```

環境変数を設定し、`compose.traefik.yml`を配置したら、次のコマンドを実行して Traefik の Docker Compose を開始できます：

```bash
docker compose -f compose.traefik.yml up -d
```

## FastAPI プロジェクトのデプロイ

Traefik を配置したら、Docker Compose で FastAPI プロジェクトをデプロイできます。

**注意**: GitHub Actions を使用した継続的デプロイメントのセクションに進みたい場合もあるかもしれません。

## 環境変数

まず、いくつかの環境変数を設定する必要があります。

`ENVIRONMENT`を設定します。デフォルトは`local`（開発用）ですが、サーバーにデプロイする場合は`staging`や`production`などを設定します：

```bash
export ENVIRONMENT=production
```

`DOMAIN`を設定します。デフォルトは`localhost`（開発用）ですが、デプロイする場合は自分のドメインを使用します。例：

```bash
export DOMAIN=fastapi-project.example.com
```

他にも以下のような変数を設定できます：

Server

- `PROJECT_NAME`: プロジェクト名。API のドキュメントやメールで使用。
- `STACK_NAME`: Docker Compose のラベルとプロジェクト名に使用されるスタック名。
  - `staging`、`production`などで異なる必要があります。
  - ドットをダッシュに置き換えたドメインを使用できます（例：`fastapi-project-example-com`や`staging-fastapi-project-example-com`）。
- `BACKEND_CORS_ORIGINS`: コンマで区切られた許可される CORS オリジンのリスト。
- `SECRET_KEY`: FastAPI プロジェクトの秘密鍵。トークンの署名に使用。
- `FIRST_SUPERUSER`: 最初のスーパーユーザーのメールアドレス。このスーパーユーザーが新しいユーザーを作成できます。
- `FIRST_SUPERUSER_PASSWORD`: 最初のスーパーユーザーのパスワード。

Mailing

- `SMTP_HOST`: メール送信用の SMTP サーバーホスト。メールプロバイダ（Sendgrid, Mailgun, Sparkpost, など）から提供されます。
- `SMTP_USER`: メール送信用の SMTP サーバーユーザー。
- `SMTP_PASSWORD`: メール送信用の SMTP サーバーパスワード。
- `EMAILS_FROM_EMAIL`: メール送信元のメールアカウント。

DB

- `POSTGRES_SERVER`: PostgreSQL サーバーのホスト名。サードパーティのプロバイダを使用しない限り、通常は変更する必要はありません。Docker Compose で提供されるデフォルトの`db`のままにできます。
- `POSTGRES_PORT`: PostgreSQL サーバーのポート。サードパーティのプロバイダを使用しない限り、通常は変更する必要はありません。
- `POSTGRES_PASSWORD`: Postgres のパスワード。
- `POSTGRES_USER`: Postgres のユーザー。
- `POSTGRES_DB`: このアプリケーションで使用するデータベース名。デフォルトの`app`のままにできます。

Logging&Monitoring

- `SENTRY_DSN`: Sentry を使用している場合の DSN。

## GitHub Actions 環境変数

GitHub Actions でのみ使用される環境変数がいくつかあります：

- `LATEST_CHANGES`: マージされた PR に基づいてリリースノートを自動的に追加する GitHub Action [latest-changes](https://github.com/tiangolo/latest-changes) で使用されます。これはパーソナルアクセストークンです。詳細はドキュメントを参照してください。
- `SMOKESHOW_AUTH_KEY`: コードカバレッジを処理・公開する GHA [Smokeshow](https://github.com/samuelcolvin/smokeshow)で使用されます。Smokeshow の指示に従って Smokeshow キーを作成してください。

### シークレットキーの生成

`.env`ファイル内のいくつかの環境変数のデフォルト値は`changethis`で、これらをシークレットキーに変更する必要があります。シークレットキーを生成するには、次のコマンドを実行できます：

```bash
python -c "import secrets; print(secrets.token_urlsafe(32))"
```

出力をコピーし、それをパスワード/シークレットキーとして使用してください。

### Docker Compose でのデプロイ

環境変数を設定したら、Docker Compose でデプロイできます：

```bash
docker compose -f compose.yml up -d
```

本番環境では`compose.override.yml`のオーバーライドは不要なので、明示的に`compose.yml`を使用するファイルとして指定しています。

## 継続的デプロイメント（CD）

GitHub Actions を使用してプロジェクトを自動的にデプロイできます。😎

複数の環境へのデプロイメントを持つことができます。

すでに`staging`と`production`の 2 つの環境が設定されています。🚀

### GitHub Actions Runner のインストール

- リモートサーバーで`root`ユーザーとして実行している場合、GitHub Actions 用のユーザーを作成します：

```bash
adduser github
```

- `github`ユーザーに Docker の権限を追加します：

```bash
usermod -aG docker github
```

- 一時的に`github`ユーザーに切り替えます：

```bash
su - github
```

- `github`ユーザーのホームディレクトリに移動します：

```bash
cd
```

- 公式ガイド[Adding a self-hosted runner to a repository](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/adding-self-hosted-runners#adding-a-self-hosted-runner-to-a-repository)に従って GitHub Action self-hosted runner をインストールします。

- ラベルについて尋ねられたら、環境用のラベル（例：`production`）を追加します。ラベルは後で追加することもできます。

インストール後、ガイドはランナーを開始するコマンドを実行するよう指示します。しかし、そのプロセスを終了したり、サーバーへのローカル接続が失われたりすると停止してしまいます。

確実に起動時に実行され、継続して実行されるようにするには、サービスとしてインストールできます。そのためには、`github`ユーザーを終了し、`root`ユーザーに戻ります：

```bash
exit
```

- `github`ユーザーのホームディレクトリ内の`actions-runner`ディレクトリに移動します：

```bash
cd /home/github/actions-runner
```

- `github`ユーザーで self-hosted runner をサービスとしてインストールします：

```bash
./svc.sh install github
```

- サービスを開始します：

```bash
./svc.sh start
```

- サービスのステータスを確認します：

```bash
./svc.sh status
```

詳細は公式ガイド[Configuring the self-hosted runner application as a service](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/configuring-the-self-hosted-runner-application-as-a-service)を参照してください。

### シークレットの設定

リポジトリで、上記で説明した環境変数（`SECRET_KEY`など）のシークレットを設定します。[リポジトリシークレットを設定するための公式 GitHub ガイド](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions#creating-secrets-for-a-repository)に従ってください。

現在の Github Actions ワークフローは以下のシークレットを期待しています：

- `DOMAIN_PRODUCTION`
- `DOMAIN_STAGING`
- `STACK_NAME_PRODUCTION`
- `STACK_NAME_STAGING`
- `EMAILS_FROM_EMAIL`
- `FIRST_SUPERUSER`
- `FIRST_SUPERUSER_PASSWORD`
- `POSTGRES_PASSWORD`
- `SECRET_KEY`
- `LATEST_CHANGES`
- `SMOKESHOW_AUTH_KEY`

## GitHub Action デプロイメントワークフロー

`.github/workflows`ディレクトリには、以下の環境（指定されたラベルを持つ GitHub Actions ランナー）にデプロイするための GitHub Action ワークフローがすでに設定されています：

- `staging`: `main`ブランチへのプッシュ（またはマージ）後。
- `production`: リリース公開後。

追加の環境が必要な場合は、これらを出発点として使用できます。

## URL

`fastapi-project.example.com`を自分のドメインに置き換えてください。

### メイン Traefik ダッシュボード

Traefik UI: `https://traefik.fastapi-project.example.com`

### 本番環境

フロントエンド: `https://fastapi-project.example.com`

バックエンド: `https://fastapi-project.example.com/api/`

Swagger UI: `https://fastapi-project.example.com/docs`

Adminer: `https://adminer.fastapi-project.example.com`

### ステージング環境

フロントエンド: `https://staging.fastapi-project.example.com`

バックエンド: `https://staging.fastapi-project.example.com/api/`

Swagger UI: `https://staging.fastapi-project.example.com/docs`

Adminer: `https://adminer.staging.fastapi-project.example.com`
