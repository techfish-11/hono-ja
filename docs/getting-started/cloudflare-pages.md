# Cloudflare Pages

[Cloudflare Pages](https://pages.cloudflare.com) は、フルスタック Web アプリケーション向けのエッジ プラットフォームです。

Cloudflare Workers が提供する静的ファイルと動的コンテンツを提供します。

Hono は Cloudflare Pages を完全にサポートしています。

開発者にとって快適なエクスペリエンスを提供します。Vite の開発サーバーは高速で、Wrangler を使用したデプロイも非常に迅速です。

## 1. セットアップ

Cloudflare Pages のスターターが利用可能です。
「create-hono」コマンドでプロジェクトを開始します。

この例では、`cloudflare-pages` テンプレートを選択します。

::: code-group

```sh [npm]
npm create hono@latest my-app
```

```sh [yarn]
yarn create hono my-app
```

```sh [pnpm]
pnpm create hono my-app
```

```sh [bun]
bunx create-hono my-app
```

```sh [deno]
deno init --npm hono my-app
```

:::

`my-app` に移動して依存関係をインストールします。

::: code-group

```sh [npm]
cd my-app
npm i
```

```sh [yarn]
cd my-app
yarn
```

```sh [pnpm]
cd my-app
pnpm i
```

```sh [bun]
cd my-app
bun i
```

:::

以下は基本的なディレクトリ構造です。

```text
./
├── package.json
├── public
│   └── static // 静的ファイルを配置します。
│   └── style.css // `/static/style.css` として参照できます。
├── src
│   ├── index.tsx // サーバーサイドのエントリ ポイント。
│   └── renderer.tsx
├── tsconfig.json
└── vite.config.ts
```

## 2. Hello World

`src/index.tsx` を次のように編集します:

```tsx
import { Hono } from 'hono'
import { renderer } from './renderer'

const app = new Hono()

app.get('*', renderer)

app.get('/', (c) => {
return c.render(<h1>Hello, Cloudflare Pages!</h1>)
})

export default app
```

## 3. 実行

開発サーバーをローカルで実行します。次に、Web ブラウザーで `http://localhost:5173` にアクセスします。

::: code-group

```sh [npm]
npm run dev
```

```sh [yarn]
yarn dev
```

```sh [pnpm]
pnpm dev
```

```sh [bun]
bun run dev
```

:::

## 4. デプロイ

Cloudflare アカウントをお持ちの場合は、Cloudflare にデプロイできます。`package.json` で、`$npm_execpath` を選択したパッケージ マネージャーに変更する必要があります。

::: code-group

```sh [npm]
npm run deploy
```

```sh [yarn]
yarn deploy
```

```sh [pnpm]
pnpm run deploy
```

```sh [bun]
bun run deploy
```

:::

### GitHub を使用して Cloudflare ダッシュボードからデプロイする

1. [Cloudflare ダッシュボード](https://dash.cloudflare.com) にログインし、アカウントを選択します。

2. アカウント ホームで、[Workers & Pages] > [アプリケーションの作成] > [Pages] > [Git に接続] を選択します。

3. GitHub アカウントを承認し、リポジトリを選択します。[ビルドとデプロイの設定] で、次の情報を入力します。

| 構成オプション | 値 |

| -------------------- | --------------- |

| 本番ブランチ | `main` |

| ビルド コマンド | `npm run build` |

| ビルド ディレクトリ | `dist` |

## バインディング

変数、KV、D1 などの Cloudflare バインディングを使用できます。

このセクションでは、変数と KV を使用します。

### `wrangler.toml` の作成

まず、ローカル バインディング用の `wrangler.toml` を作成します:

```sh
touch wrangler.toml
```

`wrangler.toml` を編集します。`MY_NAME` という名前の変数を指定します。

```toml
[vars]
MY_NAME = "Hono"
```

### KV の作成

次に、KV を作成します。次の `wrangler` コマンドを実行します:

```sh
wrangler kv namespace create MY_KV --preview
```

次の出力として `preview_id` を書き留めます:

```
{ binding = "MY_KV", preview_id = "abcdef" }
```

Bindings の名前 `MY_KV` で `preview_id` を指定します:

```toml
[[kv_namespaces]]
binding = "MY_KV"
id = "abcdef"
```

### `vite.config.ts` を編集します

`vite.config.ts` を編集します:

```ts
import devServer from '@hono/vite-dev-server'
import adaptor from '@hono/vite-dev-server/cloudflare'
import build from '@hono/vite-cloudflare-pages'
import { defineConfig } from 'vite'

export default defineConfig({
plugins: [
devServer({
entry: 'src/index.tsx',
adaptor, // Cloudflare Adapter
}),
build(),
],
})
```

### アプリケーションでバインディングを使用する

アプリケーションで変数と KV を使用します。タイプを設定します。

```ts
type Bindings = {
MY_NAME: string
MY_KV: KVNamespace
}

const app = new Hono<{ Bindings: Bindings }>()
```

これらを使用します:

```tsx
app.get('/', async (c) => {
await c.env.MY_KV.put('name', c.env.MY_NAME)
const name = await c.env.MY_KV.get('name')
return c.render(<h1>Hello! {name}</h1>)
})
```

### 本番環境

Cloudflare Pages の場合、ローカル開発には `wrangler.toml` を使用しますが、本番環境ではダッシュボードで Bindings を設定します。

## クライアント側

クライアント側スクリプトを記述し、Vite の機能を使用してアプリケーションにインポートできます。

`/src/client.ts` がクライアントのエントリ ポイントである場合は、スクリプト タグに記述するだけです。

さらに、`import.meta.env.PROD` は、開発サーバー上で実行されているのか、ビルド フェーズで実行されているのかを検出するのに役立ちます。

```tsx
app.get('/', (c) => {
return c.html(
<html>
<head>
{import.meta.env.PROD ? (
<script type='module' src='/static/client.js'></script>
) : (
<script type='module' src='/src/client.ts'></script>
)}
</head>
<body>
<h1>Hello</h1>
</body>
</html>
)
})
```

スクリプトを適切にビルドするには、以下に示すように、サンプル構成ファイル `vite.config.ts` を使用できます。

```ts
import pages from '@hono/vite-cloudflare-pages'
import devServer from '@hono/vite-dev-server'
import { defineConfig } from 'vite'

export default defineConfig(({ mode }) => {
if (mode === 'client') {
return {
build: {
rollupOptions: {
input: './src/client.ts',
output: {
entryFileNames: 'static/client.js',
},
},
},
}
} else {
return {
plugins: [
pages(),
devServer({
entry: 'src/index.tsx',
}),
],
}
}
})
```

次のコマンドを実行して、サーバーおよびクライアント スクリプトをビルドできます。

```sh
vite build --mode client && vite build
```

## Cloudflare Pages ミドルウェア

Cloudflare Pages は、Hono のミドルウェアとは異なる独自の [ミドルウェア](https://developers.cloudflare.com/pages/functions/middleware/) システムを使用します。次のように、`_middleware.ts` という名前のファイルに `onRequest` をエクスポートすることで、これを有効にできます。

```ts
// functions/_middleware.ts
export async function onRequest(pagesContext) {
console.log(`You are accessing ${pagesContext.request.url}`)
return await pagesContext.next()
}
```

`handleMiddleware` を使用すると、Hono のミドルウェアを Cloudflare Pages ミドルウェアとして使用できます。

```ts
// functions/_middleware.ts
import { handleMiddleware } from 'hono/cloudflare-pages'

export const onRequest = handleMiddleware(async (c, next) => {
console.log(`${c.req.url} にアクセスしています`)
await next()
})
```

Hono の組み込みミドルウェアやサードパーティ ミドルウェアを使用することもできます。たとえば、基本認証を追加するには、[Hono の基本認証ミドルウェア](/docs/middleware/builtin/basic-auth) を使用できます。

```ts
// functions/_middleware.ts
import { handleMiddleware } from 'hono/cloudflare-pages'
import { basicAuth } from 'hono/basic-auth'

export const onRequest = handleMiddleware(
basicAuth({
username: 'hono',
password: 'acoolproject',
})
)
```

複数のミドルウェアを適用する場合は、次のように記述できます:

```ts
import { handleMiddleware } from 'hono/cloudflare-pages'

// ...

export const onRequest = [
handleMiddleware(middleware1),
handleMiddleware(middleware2),
handleMiddleware(middleware3),
]
```

### `EventContext` へのアクセス

次の URL にアクセスできます[`EventContext`](https://developers.cloudflare.com/pages/functions/api-reference/#eventcontext) オブジェクトは、`handleMiddleware` の `c.env` を介して取得されます。

```ts
// functions/_middleware.ts
import { handleMiddleware } from 'hono/cloudflare-pages'

export const onRequest = [
handleMiddleware(async (c, next) => {
c.env.eventContext.data.user = 'Joe'
await next()
}),
]
```

その後、ハンドラーの `c.env.eventContext` を介してデータ値にアクセスできます:

```ts
// functions/api/[[route]].ts
import type { EventContext } from 'hono/cloudflare-pages'
import { handle } from 'hono/cloudflare-pages'

// ...

type Env = {
Bindings: {
eventContext: EventContext
}
}

const app = new Hono<Env>()

app.get('/hello', (c) => {
return c.json({
message: `Hello, ${c.env.eventContext.data.user}!`, // 'Joe'
})
})

export const onRequest = handle(app)
```