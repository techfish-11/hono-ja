# Azure Functions

[Azure Functions](https://azure.microsoft.com/en-us/products/functions) は、Microsoft Azure のサーバーレス プラットフォームです。イベントに応じてコードを実行でき、基盤となるコンピューティング リソースが自動的に管理されます。

Hono は当初、Azure Functions 用に設計されていませんでした。ただし、[Azure Functions アダプター](https://github.com/Marplex/hono-azurefunc-adapter) を使用すると、Azure Functions でも実行できます。

Node.js 18 以降で実行される Azure Functions **V4** で動作します。

## 1. CLI をインストールする

Azure Functions を作成するには、まず [Azure Functions Core Tools](https://learn.microsoft.com/en-us/azure/azure-functions/create-first-function-cli-typescript?pivots=nodejs-model-v4#install-the-azure-functions-core-tools) をインストールする必要があります。

macOS の場合

```sh
brew tap azure/functions
brew install azure-functions-core-tools@4
```

他の OS の場合は、このリンクに従ってください:

- [Azure Functions Core Tools をインストールする | Microsoft Learn](https://learn.microsoft.com/en-us/azure/azure-functions/create-first-function-cli-typescript?pivots=nodejs-model-v4#install-the-azure-functions-core-tools)

## 2. セットアップ

現在のフォルダーに TypeScript Node.js V4 プロジェクトを作成します。

```sh
func init --typescript
```

ホストの既定のルート プレフィックスを変更します。このプロパティを `host.json` のルート json オブジェクトに追加します:

```json
"extensions": {
"http": {
"routePrefix": ""
}
}
```

::: info
Azure Functions のデフォルトのルート プレフィックスは `/api` です。上記のように変更しない場合は、すべての Hono ルートを `/api` で開始するようにしてください
:::

これで、次のコマンドを使用して Hono と Azure Functions アダプターをインストールする準備が整いました:

::: code-group

```sh [npm]
npm i @marplex/hono-azurefunc-adapter hono
```

```sh [yarn]
yarn add @marplex/hono-azurefunc-adapter hono
```

```sh [pnpm]
pnpm add @marplex/hono-azurefunc-adapter hono
```

```sh [bun]
bun add @marplex/hono-azurefunc-adapter hono
```

:::

## 3. Hello World

`src/app.ts` を作成します:

```ts
// src/app.ts
import { Hono } from 'hono'
const app = new Hono()

app.get('/', (c) => c.text('Hello Azure Functions!'))

export default app
```

`src/functions/httpTrigger.ts` を作成します:

```ts
// src/functions/httpTrigger.ts
import { app } from '@azure/functions'
import { azureHonoHandler } from '@marplex/hono-azurefunc-adapter'
import honoApp from '../app'

app.http('httpTrigger', {
methods: [
// サポートされているすべての HTTP メソッドをここに追加します
'GET',
'POST',
'DELETE',
'PUT',
],
authLevel: 'anonymous',
route: '{*proxy}',
handler: azureHonoHandler(honoApp.fetch),
})
```

## 4. 実行

開発サーバーをローカルで実行します。次に、Web ブラウザーで `http://localhost:7071` にアクセスします。

::: code-group

```sh [npm]
npm run start
```

```sh [yarn]
yarn start
```

```sh [pnpm]
pnpm start
```

```sh [bun]
bun run start
```

:::

## 5. デプロイ

::: info
Azure にデプロイする前に、クラウド インフラストラクチャにいくつかのリソースを作成する必要があります。 [関数をサポートする Azure リソースを作成する](https://learn.microsoft.com/en-us/azure/azure-functions/create-first-function-cli-typescript?pivots=nodejs-model-v4&tabs=windows%2Cazure-cli%2Cbrowser#create-supporting-azure-resources-for-your-function) に関する Microsoft ドキュメントをご覧ください。
:::

デプロイ用にプロジェクトをビルドします:

::: code-group

```sh [npm]
npm run build
```

```sh [yarn]
yarn build
```

```sh [pnpm]
pnpm build
```

```sh [bun]
bun run build
```

:::

Azure Cloud の関数アプリにプロジェクトをデプロイします。`<YourFunctionAppName>` をアプリの名前に置き換えます。

```sh
func azure functionapp publish <関数アプリ名>
```