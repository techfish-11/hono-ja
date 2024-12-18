# Service Worker

[Service Worker](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API) は、キャッシュやプッシュ通知などのタスクを処理するためにブラウザのバックグラウンドで実行されるスクリプトです。Service Worker アダプターを使用すると、Hono で作成されたアプリケーションをブラウザ内で [FetchEvent](https://developer.mozilla.org/en-US/docs/Web/API/FetchEvent) ハンドラーとして実行できます。

このページでは、[Vite](https://vitejs.dev/) を使用してプロジェクトを作成する例を示します。

## 1. セットアップ

まず、プロジェクト ディレクトリを作成してそこに移動します:

```sh
mkdir my-app
cd my-app
```

プロジェクトに必要なファイルを作成します。以下の内容で `package.json` ファイルを作成します:

```json
{
"name": "my-app",
"private": true,
"scripts": {
"dev": "vite dev"
},
"type": "module"
}
```

同様に、以下の内容で `tsconfig.json` ファイルを作成します:

```json
{
"compilerOptions": {
"target": "ES2020",
"module": "ESNext",
"lib": ["ES2020", "DOM", "WebWorker"],
"moduleResolution": "bundler"
},
"include": ["./"],
"exclude": ["node_modules"]
}
```

次に、必要なモジュールをインストールします。

::: code-group

```sh [npm]
npm i hono
npm i -D vite
```

```sh [yarn]
yarn add hono
yarn add -D vite
```

```sh [pnpm]
pnpm add hono
pnpm add -D vite
```

```sh [bun]
bun add hono
bun add -D vite
```

:::

## 2. Hello World

`index.html` を編集します:

```html
<!doctype html>
<html>
<body>
<a href="/sw">Hello World by Service Worker</a>
<script type="module" src="/main.ts"></script>
</body>
</html>
```

`main.ts` は、Service を登録するためのスクリプトですワーカー:

```ts
function register() {
navigator.serviceWorker
.register('/sw.ts', { scope: '/sw', type: 'module' })
.then(
function (_registration) {
console.log('Service Worker の登録: 成功')
},
function (_error) {
console.log('Service Worker の登録: エラー')
}
)
}
function start() {
navigator.serviceWorker
.getRegistrations()
.then(function (registrations) {
for (const registration of registrations) {
console.log('Service Worker の登録解除')
registration.unregister()
}
register()
})
}
start()
```

`sw.ts` で、Hono を使用してアプリケーションを作成し、Service Worker アダプターの `fetch` イベントに登録します。 `handle` 関数。これにより、Hono アプリケーションは `/sw` へのアクセスを傍受できます。

```ts
// 型をサポートするには
// https://github.com/microsoft/TypeScript/issues/14877
declare const self: ServiceWorkerGlobalScope

import { Hono } from 'hono'
import { handle } from 'hono/service-worker'

const app = new Hono().basePath('/sw')
app.get('/', (c) => c.text('Hello World'))

self.addEventListener('fetch', handle(app))
```

## 3. 実行

開発サーバーを起動します。

::: code-group

```sh [npm]
npm run dev
```

```sh [yarn]
yarn dev
```

```sh [pnpm]
pnpm run dev
```

```sh [bun]
bun run dev
```

:::

デフォルトでは、開発サーバーはポート `5173` で実行されます。ブラウザで `http://localhost:5173/` にアクセスして、Service Worker の登録を完了します。次に、`/sw` にアクセスして、Hono アプリケーションからの応答を確認します。