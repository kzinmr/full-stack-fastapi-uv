はい、このドキュメントを日本語に翻訳いたします。

# FastAPIプロジェクト - デプロイメント

Docker Composeを使用してリモートサーバーにプロジェクトをデプロイできます。

このプロジェクトでは、外部との通信とHTTPS証明書を処理するTraefikプロキシがあることを前提としています。

CI/CD（継続的インテグレーションと継続的デプロイメント）システムを使用して自動的にデプロイできます。GitHub Actionsで実行するための設定がすでに用意されています。

ただし、最初にいくつかの設定を行う必要があります。🤓

## 準備

* リモートサーバーが準備され、利用可能な状態であること。
* ドメインのDNSレコードを、作成したサーバーのIPを指すように設定すること。
* ドメインのワイルドカードサブドメインを設定し、異なるサービスに対して複数のサブドメインを持てるようにすること。例：`*.fastapi-project.example.com`。これは`dashboard.fastapi-project.example.com`、`api.fastapi-project.example.com`、`traefik.fastapi-project.example.com`、`adminer.fastapi-project.example.com`などの異なるコンポーネントにアクセスする際に役立ちます。また、`staging`環境用の`dashboard.staging.fastapi-project.example.com`、`adminer.staging.fastapi-project.example.com`などにも使用できます。
* リモートサーバーに[Docker](https://docs.docker.com/engine/install/)をインストールし、設定すること（Docker Engineを使用し、Docker Desktopではありません）。

## パブリックTraefik

着信接続とHTTPS証明書を処理するためにTraefikプロキシが必要です。

以下の手順は一度だけ実行する必要があります。

### Traefik Docker Compose

- TraefikのDocker Composeファイルを保存するリモートディレクトリを作成します：

```bash
mkdir -p /root/code/traefik-public/
```

TraefikのDocker Composeファイルをサーバーにコピーします。ローカルターミナルで`rsync`コマンドを実行して行うことができます：

```bash
rsync -a compose.traefik.yml root@your-server.example.com:/root/code/traefik-public/
```

### Traefikパブリックネットワーク

このTraefikは、スタックとの通信のために`traefik-public`という名前のDocker "パブリックネットワーク"を想定しています。

これにより、外部世界との通信（HTTPとHTTPS）を処理する単一のパブリックTraefikプロキシがあり、その背後に1つまたは複数のスタックを持つことができます。これらのスタックは、同じ単一のサーバー上にあっても異なるドメインを持つことができます。

`traefik-public`という名前のDocker "パブリックネットワーク"を作成するには、リモートサーバーで次のコマンドを実行します：

```bash
docker network create traefik-public
```

### Traefik環境変数

TraefikのDocker Composeファイルは、起動前にターミナルでいくつかの環境変数を設定することを想定しています。リモートサーバーで次のコマンドを実行して設定できます。

- HTTP Basic認証用のユーザー名を作成します。例：

```bash
export USERNAME=admin
```

- HTTP Basic認証用のパスワードを環境変数に設定します。例：

```bash
export PASSWORD=changethis
```

- opensslを使用してHTTP Basic認証用のパスワードの"ハッシュ化"バージョンを生成し、環境変数に保存します：

```bash
export HASHED_PASSWORD=$(openssl passwd -apr1 $PASSWORD)
```

ハッシュ化されたパスワードが正しいことを確認するには、次のように表示できます：

```bash
echo $HASHED_PASSWORD
```

- サーバーのドメイン名を環境変数に設定します。例：

```bash
export DOMAIN=fastapi-project.example.com
```

- Let's Encrypt用のメールアドレスを環境変数に設定します。例：

```bash
export EMAIL=admin@example.com
```

**注意**：異なるメールアドレスを設定する必要があります。`@example.com`のメールアドレスは機能しません。

### TraefikのDocker Composeを起動

リモートサーバーでTraefikのDocker Composeファイルをコピーしたディレクトリに移動します：

```bash
cd /root/code/traefik-public/
```

環境変数を設定し、`compose.traefik.yml`が配置されたら、次のコマンドを実行してTraefikのDocker Composeを起動できます：

```bash
docker compose -f compose.traefik.yml up -d
```

## FastAPIプロジェクトのデプロイ

Traefikが設置されたら、Docker Composeを使用してFastAPIプロジェクトをデプロイできます。

**注意**：GitHub Actionsを使用した継続的デプロイメントのセクションに進むこともできます。

## 環境変数

まず、いくつかの環境変数を設定する必要があります。

`ENVIRONMENT`を設定します。デフォルトは`local`（開発用）ですが、サーバーにデプロイする場合は`staging`や`production`などを設定します：

```bash
export ENVIRONMENT=production
```

`DOMAIN`を設定します。デフォルトは`localhost`（開発用）ですが、デプロイ時には独自のドメインを使用します。例：

```bash
export DOMAIN=fastapi-project.example.com
```

以下のような複数の変数を設定できます：

- `PROJECT_NAME`：プロジェクト名。APIのドキュメントやメールで使用されます。
- `STACK_NAME`：Docker Composeのラベルとプロジェクト名に使用されるスタック名。`staging`、`production`などで異なる必要があります。ドットをハイフンに置き換えたドメインを使用できます（例：`fastapi-project-example-com`や`staging-fastapi-project-example-com`）。
- `BACKEND_CORS_ORIGINS`：許可されるCORSオリジンのリスト（カンマ区切り）。
- `SECRET_KEY`：FastAPIプロジェクトの秘密鍵。トークンの署名に使用されます。
- `FIRST_SUPERUSER`：最初のスーパーユーザーのメールアドレス。このスーパーユーザーが新しいユーザーを作成できます。
- `FIRST_SUPERUSER_PASSWORD`：最初のスーパーユーザーのパスワード。
- `SMTP_HOST`：メール送信用のSMTPサーバーホスト。メールプロバイダー（Mailgun、Sparkpost、Sendgridなど）から提供されます。
- `SMTP_USER`：メール送信用のSMTPサーバーユーザー。
- `SMTP_PASSWORD`：メール送信用のSMTPサーバーパスワード。
- `EMAILS_FROM_EMAIL`：メール送信元のメールアカウント。
- `POSTGRES_SERVER`：PostgreSQLサーバーのホスト名。同じDocker Composeで提供されるデフォルトの`db`のままにできます。サードパーティのプロバイダーを使用しない限り、通常は変更する必要はありません。
- `POSTGRES_PORT`：PostgreSQLサーバーのポート。デフォルトのままにできます。サードパーティのプロバイダーを使用しない限り、通常は変更する必要はありません。
- `POSTGRES_PASSWORD`：Postgresのパスワード。
- `POSTGRES_USER`：Postgresのユーザー名。デフォルトのままにできます。
- `POSTGRES_DB`：このアプリケーションで使用するデータベース名。デフォルトの`app`のままにできます。
- `SENTRY_DSN`：Sentryを使用している場合のDSN。

## GitHub Actions環境変数

GitHub Actionsでのみ使用される環境変数があります：

- `LATEST_CHANGES`：GitHub Action [latest-changes](https://github.com/tiangolo/latest-changes)で使用され、マージされたPRに基づいてリリースノートを自動的に追加します。これは個人用アクセストークンです。詳細はドキュメントを参照してください。
- `SMOKESHOW_AUTH_KEY`：[Smokeshow](https://github.com/samuelcolvin/smokeshow)を使用してコードカバレッジを処理し公開するために使用されます。Smokeshowの指示に従って（無料の）Smokeshowキーを作成してください。

### 秘密鍵の生成

`.env`ファイル内のいくつかの環境変数は、デフォルト値が`changethis`になっています。

これらを秘密鍵に変更する必要があります。秘密鍵を生成するには、次のコマンドを実行できます：

```bash
python -c "import secrets; print(secrets.token_urlsafe(32))"
```

出力内容をコピーし、パスワード/秘密鍵として使用します。別の安全な鍵を生成するには、このコマンドを再度実行します。

### Docker Composeでのデプロイ

環境変数を設定したら、Docker Composeでデプロイできます：

```bash
docker compose -f compose.yml up -d
```

本番環境では`compose.override.yml`のオーバーライドは不要なため、明示的に`compose.yml`をファイルとして指定しています。

## 継続的デプロイメント（CD）

GitHub Actionsを使用してプロジェクトを自動的にデプロイできます。😎

複数の環境デプロイメントを持つことができます。

すでに`staging`と`production`の2つの環境が設定されています。🚀

### GitHub Actions Runnerのインストール

- リモートサーバーで`root`ユーザーとして実行している場合、GitHub Actions用のユーザーを作成します：

```bash
adduser github
```

- `github`ユーザーにDockerの権限を追加します：

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

- [公式ガイドに従って、GitHub Actionのセルフホストランナーをインストールします](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/adding-self-hosted-runners#adding-a-self-hosted-runner-to-a-repository)。

- ラベルについて尋ねられたら、環境のラベル（例：`production`）を追加します。後でラベルを追加することもできます。

インストール後、ガイドではランナーを開始するコマンドを実行するよう指示されますが、そのプロセスを終了したり、サーバーへのローカル接続が失われると停止してしまいます。

起動時に確実に実行され、継続的に実行されるようにするには、サービスとしてインストールできます。そのために、`github`ユーザーを終了し、`root`ユーザーに戻ります：

```bash
exit
```

これを行うと、再び`root`ユーザーになります。また、`root`ユーザーに属する以前のディレクトリにいることになります。

- `github`ユーザーのホームディレクトリ内の`actions-runner`ディレクトリに移動します：

```bash
cd /home/github/actions-runner
```

- セルフホストランナーを`github`ユーザーでサービスとしてインストールします：

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

詳細は公式ガイドを参照してください：[セルフホストランナーアプリケーションをサービスとして設定する](https://docs.github.com/en/actions/hosting-your-own-runners/managing-self-hosted-runners/configuring-the-self-hosted-runner-application-as-a-service)。

### シークレットの設定

リポジトリで、上記で説明した環境変数（`SECRET_KEY`など）のシークレットを設定します。[リポジトリのシークレットを設定する公式GitHubガイド](https://docs.github.com/en/actions/security-guides/using-secrets-in-github-actions#creating-secrets-for-a-repository)に従ってください。

現在のGitHub Actionsワークフローは以下のシークレットを想定しています：

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

## GitHub Actionデプロイメントワークフロー

`.github/workflows`ディレクトリには、環境（ラベル付きのGitHub Actionsランナー）へのデプロイ用にすでに設定されたGitHub Actionワークフローがあります：

- `staging`：`main`ブランチへのプッシュ（またはマージ）後。
- `production`：リリース公開後。

追加の環境が必要な場合は、これらを出発点として使用できます。

## URLs

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
