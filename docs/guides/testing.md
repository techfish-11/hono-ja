# テスト

テストは重要です。

実際には、Hono のアプリケーションをテストするのは簡単です。

テスト環境の作成方法はランタイムごとに異なりますが、基本的な手順は同じです。

このセクションでは、Cloudflare Workers と Jest を使用してテストしてみましょう。

## リクエストとレスポンス

リクエストを作成し、それを Hono アプリケーションに渡してレスポンスを検証するだけです。

また、便利なメソッド `app.request` を使用できます。

::: ヒント
型付けされたテスト クライアントについては、[テスト ヘルパー](/docs/helpers/testing) を参照してください。
:::

たとえば、次の REST API を提供するアプリケーションを考えてみましょう。

```ts
app.get('/posts', (c) => {
return c.text('Many posts')
})

app.post('/posts', (c) => {
return c.json(
{
message: 'Created',
},
201,
{
'X-Custom': 'Thank you',
}
)
})
```

`GET /posts` へのリクエストを作成し、応答をテストします。

```ts
describe('Example', () => {
test('GET /posts', async () => {
const res = await app.request('/posts')
expect(res.status).toBe(200)
expect(await res.text()).toBe('Many posts')
})
})
```

`POST /posts` へのリクエストを行うには、次の操作を行います。

```ts
test('POST /posts', async () => {
const res = await app.request('/posts', {
method: 'POST',
})
expect(res.status).toBe(201)
expect(res.headers.get('X-Custom')).toBe('Thank you')
expect(await res.json()).toEqual({
message: 'Created',
})
})
```

`JSON` データを使用して `POST /posts` へのリクエストを行うには、次の操作を行います。

```ts
test('POST /posts', async () => {
const res = await app.request('/posts', {
method: 'POST',
body: JSON.stringify({ message: 'hello hono' }),
headers: new Headers({ 'Content-Type': 'application/json' }),
})
expect(res.status).toBe(201)
expect(res.headers.get('X-Custom')).toBe('Thank you')
expect(await res.json()).toEqual({
message: 'Created',
})
})
```

`multipart/form-data` データを使用して `POST /posts` へのリクエストを行うには、次の操作を行います。

```ts
test('POST /posts', async () => {
const formData = new FormData()
formData.append('message', 'hello')
const res = await app.request('/posts', {
method: 'POST',
body: formData,
})
expect(res.status).toBe(201)
expect(res.headers.get('X-Custom')).toBe('Thank you')
expect(await res.json()).toEqual({
message: 'Created',
})
})
```

Request クラスのインスタンスを渡すこともできます。

```ts
test('POST /posts', async () => {
const req = new Request('http://localhost/posts', {
method: 'POST',
})
const res = await app.request(req)
expect(res.status).toBe(201)
expect(res.headers.get('X-Custom')).toBe('Thank you')
expect(await res.json()).toEqual({
message: 'Created',
})
})
```

このようにして、エンドツーエンドのようにテストできます。

## Env

テスト用に `c.env` を設定するには、`app.request` の 3 番目のパラメータとして渡します。これは、[Cloudflare Workers Bindings](https://hono.dev/getting-started/cloudflare-workers#bindings) のような値をモックするのに便利です:

```ts
const MOCK_ENV = {
API_HOST: 'example.com',
DB: {
prepare: () => {
/* mocked D1 */
},
},
}

test('GET /posts', async () => {
const res = await app.request('/posts', {}, MOCK_ENV)
})
```