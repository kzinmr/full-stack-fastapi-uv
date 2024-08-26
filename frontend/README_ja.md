# フロントエンド

フロントエンドは以下を使用して構築されています:

- [Vite](https://vitejs.dev/)
- [React](https://reactjs.org/)
- [TypeScript](https://www.typescriptlang.org/)
- [TanStack Query](https://tanstack.com/query)
- [TanStack Router](https://tanstack.com/router)
- [Chakra UI](https://chakra-ui.com/)

## コード構造

フロントエンドコードは以下のように構成されています：

- `frontend/src` - メインのフロントエンドコード。
- `frontend/src/assets` - 静的アセット。
- `frontend/src/client` - 生成された OpenAPI クライアント。
- `frontend/src/components` - フロントエンドの各種コンポーネント。
- `frontend/src/hooks` - カスタムフック。
- `frontend/src/routes` - ページを含むフロントエンドの各種ルート。
- `frontend/src/theme.tsx` - Chakra UI のカスタムテーマ。

## フロントエンドの削除

API のみのアプリを開発していて、フロントエンドを削除したい場合は、簡単に行えます：

- `./frontend`ディレクトリを削除します。

- `compose.yml`ファイルで、`frontend`サービス/セクション全体を削除します。

- `compose.override.yml`ファイルで、`frontend`サービス/セクション全体を削除します。

これで、フロントエンドのない（API のみの）アプリができました。🤓

---

必要に応じて、以下のファイルから`FRONTEND`環境変数も削除できます：

- `.env`
- `./scripts/*.sh`

ただし、これらを削除するのはクリーンアップのためだけであり、残しておいても特に影響はありません。

## フロントエンド開発

開始する前に、Node Version Manager (nvm)または Fast Node Manager (fnm)がシステムにインストールされていることを確認してください。

- fnm をインストールするには、[公式 fnm ガイド](https://github.com/Schniz/fnm#installation)に従ってください。nvm を使用したい場合は、[公式 nvm ガイド](https://github.com/nvm-sh/nvm#installing-and-updating)を使用してインストールできます。

- nvm または fnm をインストールした後、`frontend`ディレクトリに移動します：

```bash
cd frontend
```

- `.nvmrc`ファイルで指定されている Node.js バージョンがシステムにインストールされていない場合は、適切なコマンドを使用してインストールできます：

```bash
# fnmを使用する場合
fnm install

# nvmを使用する場合
nvm install
```

- インストールが完了したら、インストールしたバージョンに切り替えます：

```bash
# fnmを使用する場合
fnm use

# nvmを使用する場合
nvm use
```

- `frontend`ディレクトリ内で、必要な NPM パッケージをインストールします：

```bash
npm install
```

- そして、以下の`npm`スクリプトでライブサーバーを起動します：

```bash
npm run dev
```

- その後、ブラウザで http://localhost:5173/ を開きます。

このライブサーバーは Docker 内で実行されているわけではなく、ローカル開発用であり、これが推奨されるワークフローです。フロントエンドに満足したら、フロントエンドの Docker イメージをビルドして起動し、本番環境に近い環境でテストできます。ただし、変更のたびにイメージをビルドするよりも、ライブリロード機能を持つローカル開発サーバーを実行する方が生産的です。

利用可能な他のオプションについては、`package.json`ファイルを確認してください。

## クライアントの生成

- Docker Compose スタックを起動します。

- `http://localhost/api/v1/openapi.json` から OpenAPI JSON ファイルをダウンロードし、`frontend`ディレクトリのルートに新しいファイル`openapi.json`としてコピーします。

- 生成されたフロントエンドクライアントコードの名前を簡略化するために、以下のスクリプトを実行して`openapi.json`ファイルを修正します：

```bash
node modify-openapi-operationids.js
```

- フロントエンドクライアントを生成するには、以下を実行します：

```bash
npm run generate-client
```

- 変更をコミットします。

バックエンドが変更される（OpenAPI スキーマが変更される）たびに、これらの手順を再度実行してフロントエンドクライアントを更新する必要があることに注意してください。

## リモート API の使用

リモート API を使用したい場合、環境変数 `VITE_API_URL` にリモート API の URL を設定できます。例えば、 `frontend/.env` ファイルに以下のように設定できます：

```env
VITE_API_URL=https://my-remote-api.example.com
```

そうすると、フロントエンドを実行する際に、その URL を API のベース URL として使用します。

## Playwright を使用したエンドツーエンドテスト

フロントエンドには、Playwright を使用した初期のエンドツーエンドテストが含まれています。テストを実行するには、Docker Compose スタックが実行されている必要があります。以下のコマンドでスタックを起動します：

```bash
docker compose up -d
```

その後、以下のコマンドでテストを実行できます：

```bash
npx playwright test
```

また、ブラウザを表示してテストの実行中に操作できる UI モードでテストを実行することもできます：

```bash
npx playwright test --ui
```

Docker Compose スタックを停止して削除し、テストで作成されたデータをクリーンアップするには、以下のコマンドを使用します：

```bash
docker compose down -v
```

テストを更新するには、テストディレクトリに移動し、既存のテストファイルを修正するか、必要に応じて新しいファイルを追加します。

Playwright テストの作成と実行に関する詳細情報については、公式の[Playwright ドキュメント](https://playwright.dev/docs/intro)を参照してください。
