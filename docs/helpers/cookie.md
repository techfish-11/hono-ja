# Cookie ヘルパー

Cookie ヘルパーは、Cookie を管理するための簡単なインターフェイスを提供し、開発者がシームレスに Cookie を設定、解析、削除できるようにします。

## インポート

```ts
import { Hono } from 'hono'
import {
getCookie,
getSignedCookie,
setCookie,
setSignedCookie,
deleteCookie,
} from 'hono/cookie'
```

## 使用方法

**注意**: 署名付き Cookie の設定と取得は、HMAC SHA-256 署名の作成に使用される WebCrypto API の非同期性により、Promise を返します。

```ts
const app = new Hono()

app.get('/cookie', (c) => {
const allCookies = getCookie(c)
const yummyCookie = getCookie(c, 'yummy_cookie')
// ...
setCookie(c, 'delicious_cookie', 'macha')
deleteCookie(c, 'delicious_cookie')
//
})

app.get('/signed-cookie', async (c) => {
const secret = 'secret ingredients'
// `getSignedCookie` は、署名が改ざんされているか無効である場合、指定された Cookie に対して `false` を返します
const allSignedCookies = await getSignedCookie(c, secret)
const fortuneCookie = await getSignedCookie(
c,
secret,
'fortune_cookie'
)
// ...
const anotherSecret = 'secret chocolate tips'
await setSignedCookie(c, 'great_cookie', 'blueberry', anotherSecret)
deleteCookie(c, 'great_cookie')
//
})
```

## オプション

### `setCookie` および `setSignedCookie`

- domain: `string`
- expires: `Date`
- httpOnly: `boolean`
- maxAge: `number`
- path: `string`
- secure: `boolean`
- sameSite: `'Strict'` | `'Lax'` | `'None'`
- prefix: `secure` | `'host'`
- パーティション化: `boolean`

例:

```ts
// 通常のクッキー
setCookie(c, 'great_cookie', 'banana', {
path: '/',
secure: true,
domain: 'example.com',
httpOnly: true,
maxAge: 1000,
expires: new Date(Date.UTC(2000, 11, 24, 10, 30, 59, 900)),
sameSite: 'Strict',
})

// 署名付きクッキー
await setSignedCookie(
c,
'fortune_cookie',
'lots-of-money',
'secret ingredients',
{
path: '/',
secure: true,
domain: 'example.com',
httpOnly: true,
maxAge: 1000、
有効期限: new Date(Date.UTC(2000, 11, 24, 10, 30, 59, 900)),
sameSite: 'Strict',
}
)
```

### `deleteCookie`

- path: `string`
- secure: `boolean`
- domain: `string`

例:

```ts
deleteCookie(c, 'banana', {
path: '/',
secure: true,
domain: 'example.com',
})
```

`deleteCookie` は削除された値を返します:

```ts
const deleteCookie = deleteCookie(c, 'delicious_cookie')
```

## `__Secure-` および `__Host-` プレフィックス

Cookie ヘルパーは `__Secure-` およびクッキー名のプレフィックス `__Host-`。

クッキー名にプレフィックスがあるかどうかを確認する場合は、プレフィックス オプションを指定します。

```ts
const securePrefixCookie = getCookie(c, 'yummy_cookie', 'secure')
const hostPrefixCookie = getCookie(c, 'yummy_cookie', 'host')

const securePrefixSignedCookie = await getSignedCookie(
c,
secret,
'fortune_cookie',
'secure'
)
const hostPrefixSignedCookie = await getSignedCookie(
c,
secret,
'fortune_cookie',
'host'
)
```

また、クッキーを設定するときにプレフィックスを指定する場合は、プレフィックス オプションの値を指定します。

```ts
setCookie(c, 'delicious_cookie', 'macha', {
prefix: 'secure', // または `host`
})

await setSignedCookie(
c,
'delicious_cookie',
'macha',
'secret choco tips',
{
prefix: 'secure', // または `host`
}
)
```

## ベスト プラクティスに従う

新しい Cookie RFC (別名 cookie-bis) と CHIPS には、開発者が従うべき Cookie 設定のベスト プラクティスがいくつか含まれています。

- [RFC6265bis-13](https://datatracker.ietf.org/doc/html/draft-ietf-httpbis-rfc6265bis-13)
- `Max-Age`/`Expires` 制限
- `__Host-`/`__Secure-` プレフィックス制限
- [CHIPS-01](https://www.ietf.org/archive/id/draft-cutler-httpbis-partitioned-cookies-01.html)
- `Partitioned` 制限

Hono はベスト プラクティスに従っています。

Cookie ヘルパーは、次の条件下で Cookie を解析するときに `Error` をスローします:

- Cookie 名が `__Secure-` で始まっているが、`secure` オプションが設定されていない。
- Cookie 名が `__Host-` で始まっているが、`secure` オプションが設定されていない。
- クッキー名は `__Host-` で始まりますが、`path` は `/` ではありません。
- クッキー名は `__Host-` で始まりますが、`domain` が設定されています。
- `maxAge` オプションの値が 400 日を超えています。
- `expires` オプションの値が現在の時刻より 400 日後です。