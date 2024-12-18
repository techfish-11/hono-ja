# クライアント コンポーネント

`hono/jsx` は、サーバー側だけでなくクライアント側もサポートしています。つまり、ブラウザーで実行されるインタラクティブな UI を作成できます。これをクライアント コンポーネントまたは `hono/jsx/dom` と呼びます。

これは高速で非常に小さいです。`hono/jsx/dom` のカウンター プログラムは、Brotli 圧縮でわずか 2.8 KB です。しかし、React の場合は 47.8 KB です。

このセクションでは、クライアント コンポーネント固有の機能を紹介します。

## カウンターの例

これは、React と同じコードが動作する、単純なカウンターの例です。

```tsx
import { useState } from 'hono/jsx'
import { render } from 'hono/jsx/dom'

function Counter() {
const [count, setCount] = useState(0)
return (
<div>
<p>Count: {count}</p>
<button onClick={() => setCount(count + 1)}>Increment</button>
</div>
)
}

function App() {
return (
<html>
<body>
<Counter />
</body>
</html>
)
}

const root = document.getElementById('root')
render(<App />, root)
```

## `render()`

`render()` を使用して、指定した HTML 要素内に JSX コンポーネントを挿入できます。

```tsx
render(<Component />, container)
```

## React と互換性のあるフック

hono/jsx/dom には、React と互換性があるか部分的に互換性があるフックがあります。これらの API について詳しくは、[React ドキュメント](https://react.dev/reference/react/hooks) を参照してください。

- `useState()`
- `useEffect()`
- `useRef()`
- `useCallback()`
- `use()`
- `startTransition()`
- `useTransition()`
- `useDeferredValue()`
- `useMemo()`
- `useLayoutEffect()`
- `useReducer()`
- `useDebugValue()`
- `createElement()`
- `memo()`
- `isValidElement()`
- `useId()`
- `createRef()`
- `forwardRef()`
- `useImperativeHandle()`
- `useSyncExternalStore()`
- `useInsertionEffect()`
- `useFormStatus()`
- `useActionState()`
- `useOptimistic()`

## `startViewTransition()` ファミリー

`startViewTransition()` ファミリーには、[View Transitions API](https://developer.mozilla.org/en-US/docs/Web/API/View_Transitions_API) を簡単に処理するための独自のフックと関数が含まれています。以下は、それらの使用方法の例です。

### 1. 最も簡単な例

`document.startViewTransition` を使用して、`startViewTransition()` で簡単に遷移を記述できます。

```tsx
'hono/jsx' から { useState, startViewTransition } をインポート
'hono/css' から { css, Style } をインポート

export default function App() {
const [showLargeImage, setShowLargeImage] = useState(false)
return (
<>
<Style />
<button
onClick={() =>
startViewTransition(() =>
setShowLargeImage((state) => !state)
)
}
>
クリック!
</button>
<div>
{!showLargeImage ? (
<img src='https://hono.dev/images/logo.png' />
) : (
<div
class={css`
background: url('https://hono.dev/images/logo-large.png');
background-size: contain;
background-repeat: no-repeat;
background-position: center;
width: 600px;
height: 600px;
`}
></div>
)}
</div>
</>
)
}
```

### 2. `keyframes()` で `viewTransition()` を使用する

`viewTransition()` 関数を使用すると、一意の `view-transition-name` を取得できます。

`keyframes()` で使用できます。`::view-transition-old()` は `::view-transition-old(${uniqueName))` に変換されます。

```tsx
'hono/jsx' から { useState, startViewTransition } をインポート
'hono/jsx/dom/css' から { viewTransition } をインポート
'hono/css' から { css, keyframes, Style } をインポート

const rotate = keyframes`
{
rotate: 0deg;
}
から {
rotate: 360deg;
}
`

export default function App() {
const [showLargeImage, setShowLargeImage] = useState(false)
const [transitionNameClass] = useState(() =>
viewTransition(css`
::view-transition-old() {
animation-name: ${rotate};
}
::view-transition-new() {
animation-name: ${rotate};
}
`)
)
return (
<>
<Style />
<button
onClick={() =>
startViewTransition(() =>
setShowLargeImage((state) => !state)
)
}
>
クリック!
</button>
<div>
{!showLargeImage ? (
<img src='https://hono.dev/images/logo.png' />
) : (
<div
class={css`
${transitionNameClass}
background: url('https://hono.dev/images/logo-large.png');
background-size: contain;
background-repeat: no-repeat;
background-position: center;
width: 600px;
height: 600px;
`}
></div>
)}
</div>
</>
)
}
```

### 3. `useViewTransition` の使用

アニメーション中にのみスタイルを変更したい場合は、`useViewTransition()` を使用できます。このフックは `[boolean, (callback: () => void) => void]` を返し、それらは `isUpdating` フラグと `startViewTransition()` 関数です。

このフックを使用すると、コンポーネントは次の 2 つの時点で評価されます。

- `startViewTransition()` 呼び出しのコールバック内。

- [`finish` プロミスが満たされたとき](https://developer.mozilla.org/en-US/docs/Web/API/ViewTransition/finished)

```tsx
import { useState, useViewTransition } from 'hono/jsx'
import { viewTransition } from 'hono/jsx/dom/css'
import { css, keyframes, Style } from 'hono/css'

const rotate = keyframes`
from {
rotate: 0deg;
}
to {
rotate: 360deg;
}
`

export default function App() {
const [isUpdating, startViewTransition] = useViewTransition()
const [showLargeImage, setShowLargeImage] = useState(false)
const [transitionNameClass] = useState(() =>
viewTransition(css`
::view-transition-old() {
animation-name: ${rotate};
}
::view-transition-new() {
animation-name: ${rotate};
}
`)
)
return (
<>
<Style />
<button
onClick={() =>
startViewTransition(() =>
setShowLargeImage((state) => !state)
)
}
>
クリック!
</button>
<div>
{!showLargeImage ? (
<img src='https://hono.dev/images/logo.png' />
) : (
<div
class={css`
${transitionNameClass}
background: url('https://hono.dev/images/logo-large.png');
background-size: contain;
background-repeat: no-repeat;
background-position: center;
width: 600px;
height: 600px;
position: relative;
${isUpdating &&
css`
&:before {
content: 'Loading...';
position: absolute;
top: 50%;
left: 50%;
}
`}
`}
></div>
)}
</div>
</>
)
}
```

## `hono/jsx/dom` ランタイム

クライアント コンポーネント用の小さな JSX ランタイムがあります。これを使用すると、`hono/jsx` を使用する場合よりもバンドルされた結果が小さくなります。 `tsconfig.json` で `hono/jsx/dom` を指定します。

```json
{
"compilerOptions": {
"jsx": "react-jsx",
"jsxImportSource": "hono/jsx/dom"
}
}
```