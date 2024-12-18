# 検証

Hono は非常に薄い検証機能のみを提供します。

ただし、サードパーティの検証機能と組み合わせると強力になります。

さらに、RPC 機能を使用すると、型を通じて API 仕様をクライアントと共有できます。

## 手動検証機能

まず、サードパーティの検証機能を使用せずに、受信した値を検証する方法を紹介します。

`hono/validator` から `validator` をインポートします。

```ts
import { validator } from 'hono/validator'
```

フォーム データを検証するには、最初の引数として `form` を指定し、2 番目の引数としてコールバックを指定します。

コールバックでは、値を検証し、最後に検証された値を返します。

`validator` はミドルウェアとして使用できます。

```ts
app.post(
'/posts',
validator('form', (value, c) => {
const body = value['body']
if (!body || typeof body !== 'string') {
return c.text('Invalid!', 400)
}
return {
body: body,
}
}),
//...
```

ハンドラー内では、`c.req.valid('form')` を使用して検証済みの値を取得できます。

```ts
, (c) => {
const { body } = c.req.valid('form')
// ... 何かを実行します
return c.json(
{
message: 'Created!',
},
201
)
}
```

検証対象には、`json`、`query`、`header`、`param` などがあります。 `form` に加えて `cookie` も必要です。

::: 警告
`json` を検証する場合、リクエストには `Content-Type: application/json` ヘッダーが含まれている必要があります。含まれていない場合、リクエスト本文は解析されず、警告が表示されます。

[`app.request()`](../api/request.md) を使用してテストする場合は、`content-type` ヘッダーを設定することが重要です。

次のようなアプリケーションがあるとします。

```ts
const app = new Hono()
app.post(
'/testing',
validator('json', (value, c) => {
// パススルー検証
戻り値
}),
(c) => {
const body = c.req.valid('json')
return c.json(body)
}
)
```

テストは次のように記述できます。

```ts
// ❌ これは動作しません
const res = await app.request('/testing', {
method: 'POST',
body: JSON.stringify({ key: 'value' }),
})
const data = await res.json()
console.log(data) // undefined

// ✅ これは動作します
const res = await app.request('/testing', {
method: 'POST',
body: JSON.stringify({ key: 'value' }),
headers: new Headers({ 'Content-Type': 'application/json' }),
})
const data = await res.json()
console.log(data) // { key: 'value' }
```

:::

::: 警告
`header` を検証する場合、**小文字** の名前を使用する必要があります。キー。

`Idempotency-Key` ヘッダーを検証する場合は、キーとして `idempotency-key` を使用する必要があります。

```ts
// ❌ これは動作しません
app.post(
'/api',
validator('header', (value, c) => {
// idempotencyKey は常に undefined です
// そのため、このミドルウェアは予想外に常に 400 を返します
const idempotencyKey = value['Idempotency-Key']

if (idempotencyKey == undefined || idempotencyKey === '') {
throw HTTPException(400, {
message: 'Idempotency-Key is required',
})
}
return { idempotencyKey }
}),
(c) => {
const { idempotencyKey } = c.req.valid('header')
// ...
}
)

// ✅ これは動作します
app.post(
'/api',
validator('header', (value, c) => {
// 期待どおりにヘッダーの値を取得できます
const idempotencyKey = value['idempotency-key']

if (idempotencyKey == undefined || idempotencyKey === '') {
throw HTTPException(400, {
message: 'Idempotency-Key is required',
})
}
return { idempotencyKey }
}),
(c) => {
const { idempotencyKey } = c.req.valid('header')
// ...
}
)
```

:::

## 複数のバリデータ

リクエストのさまざまな部分を検証するために、複数のバリデータを含めることもできます:

```ts
app.post(
'/posts/:id',
validator('param', ...),
validator('query', ...),
validator('json', ...),
(c) => {
//...
}
```

## Zod を使用する場合

サードパーティのバリデーターの 1 つである [Zod](https://zod.dev) を使用できます。

サードパーティのバリデーターを使用することをお勧めします。

Npm レジストリからインストールします。

::: code-group

```sh [npm]
npm i zod
```

```sh [yarn]
yarn add zod
```

```sh [pnpm]
pnpm add zod
```

```sh [bun]
bun add zod
```

:::

`zod` から `z` をインポートします。

```ts
import { z } from 'zod'
```

スキーマを記述します。

```ts
const schema = z.object({
body: z.string(),
})
```

コールバック関数でスキーマを使用して検証し、検証された値を返すことができます。

```ts
const route = app.post(
'/posts',
validator('form', (value, c) => {
const parsed = schema.safeParse(value)
if (!parsed.success) {
return c.text('Invalid!', 401)
}
return parsed.data
}),
(c) => {
const { body } = c.req.valid('form')
// ... 何かする
return c.json(
{
message: 'Created!',
},
201
)
}
)
```

## Zod Validator ミドルウェア

[Zod Validator ミドルウェア](https://github.com/honojs/middleware/tree/main/packages/zod-validator) を使用すると、さらに簡単になります。

::: コード グループ

```sh [npm]
npm i @hono/zod-validator
```

```sh [yarn]
yarn add @hono/zod-validator
```

```sh [pnpm]
pnpm add @hono/zod-validator
```

```sh [bun]
bun add @hono/zod-validator
```

:::

`zValidator` をインポートします。

```ts
import { zValidator } from '@hono/zod-validator'
```

次のように記述します。

```ts
const route = app.post(
'/posts',
zValidator(
'form',
z.object({
body: z.string(),
})
),
(c) => {
const validated = c.req.valid('form')
// ... 検証済みのデータを使用します
}
)
```