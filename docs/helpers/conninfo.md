# ConnInfo ヘルパー

ConnInfo ヘルパーは接続情報を取得するのに役立ちます。たとえば、クライアントのリモート アドレスを簡単に取得できます。

## インポート

::: code-group

```ts [Cloudflare Workers]
import { Hono } from 'hono'
import { getConnInfo } from 'hono/cloudflare-workers'
```

```ts [Deno]
import { Hono } from 'hono'
import { getConnInfo } from 'hono/deno'
```

```ts [Bun]
import { Hono } from 'hono'
import { getConnInfo } from 'hono/bun'
```

```ts [Vercel]
import { Hono } from 'hono'
import { getConnInfo } from 'hono/vercel'
```

```ts [Lambda@Edge]
import { Hono } from 'hono'
import { getConnInfo } from 'hono/lambda-edge'
```

```ts [Node.js]
import { Hono } from 'hono'
import { getConnInfo } from '@hono/node-server/conninfo'
```

:::

## 使用方法

```ts
const app = new Hono()

app.get('/', (c) => {
const info = getConnInfo(c) // info is `ConnInfo`
return c.text(`Your remote address is ${info.remote.address}`)
})
```

## 型定義

`getConnInfo()` から取得できる値の型定義は次のとおりです:

```ts
type AddressType = 'IPv6' | 'IPv4' |未定義

type NetAddrInfo = {
/**
* トランスポート プロトコル タイプ
*/
transport?: 'tcp' | 'udp'
/**
* トランスポート ポート番号
*/
port?: number

address?: string
addressType?: AddressType
} & (
| {
/**
* IP Addr などのホスト名
*/
address: string

/**
* ホスト名 タイプ
*/
addressType: AddressType
}
| {}
)

/**
* HTTP 接続情報
*/
interface ConnInfo {
/**
* リモート情報
*/
remote: NetAddrInfo
}
```