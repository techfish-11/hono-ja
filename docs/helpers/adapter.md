# アダプタ ヘルパー

アダプタ ヘルパーは、統一されたインターフェイスを通じてさまざまなプラットフォームとシームレスにやり取りする方法を提供します。

## インポート

```ts
import { Hono } from 'hono'
import { env, getRuntimeKey } from 'hono/adapter'
```

## `env()`

`env​​()` 関数は、Cloudflare Workers のバインディングを超えて、さまざまなランタイム間で環境変数を取得するのに役立ちます。`env(c)` で取得できる値は、ランタイムごとに異なる場合があります。

```ts
import { env } from 'hono/adapter'

app.get('/env', (c) => {
// NAME は Node.js または Bun の process.env.NAME
// NAME は Cloudflare の `wrangler.toml` に書き込まれた値
const { NAME } = env<{ NAME: string }>(c)
return c.text(NAME)
})
```

サポートされているランタイム、サーバーレス プラットフォーム、クラウド サービス:

- Cloudflare Workers
- `wrangler.toml`
- Deno
- [`Deno.env`](https://docs.deno.com/runtime/manual/basics/env_variables)
- `.env` ファイル
- Bun
- [`Bun.env`](https://bun.sh/guides/runtime/set-env)
- `process.env`
- Node.js
- `process.env`
- Vercel
- [Vercel の環境変数](https://vercel.com/docs/projects/environment-variables)
- AWS Lambda
- [AWS Lambda の環境変数](https://docs.aws.amazon.com/lambda/latest/dg/samples-blank.html#samples-blank-architecture)
- Lambda@Edge\
Lambda の環境変数は Lambda@Edge では [サポートされていません](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/add-origin-custom-headers.html)。代わりに [Lamdba@Edge イベント](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/lambda-event-structure.html) を使用する必要があります。
- Fastly Compute\
Fastly Compute では、ConfigStore を使用してユーザー定義データを管理できます。
- Netlify\
Netlify では、[Netlify コンテキスト](https://docs.netlify.com/site-deploys/overview/#deploy-contexts) を使用してユーザー定義データを管理できます。

### ランタイムを指定する

ランタイム キーを 2 番目の引数として渡すことで、環境変数を取得するランタイムを指定できます。

```ts
app.get('/env', (c) => {
const { NAME } = env<{ NAME: string }>(c, 'workerd')
return c.text(NAME)
})
```

## `getRuntimeKey()`

`getRuntimeKey()` 関数は、現在のランタイムの識別子を返します。

```ts
app.get('/', (c) => {
if (getRuntimeKey() === 'workerd') {
return c.text('You are on Cloudflare')
} else if (getRuntimeKey() === 'bun') {
return c.text('You are on Bun')
}
...
})
```

### 使用可能なランタイム キー

使用可能なランタイム キーは次のとおりです。使用できないランタイム キー ランタイムはサポートされ、`other` としてラベル付けされる場合があります。一部は [WinterCG のランタイム キー](https://runtime-keys.proposal.wintercg.org/) に触発されています。

- `workerd` - Cloudflare Workers
- `deno`
- `bun`
- `node`
- `edge-light` - Vercel Edge Functions
- `fastly` - Fastly Compute
- `other` - その他の不明なランタイム キー