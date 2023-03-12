# importの仕方（ECMAScript Modules）

最新のlibを使いながらfrontendを開発していると、以下のエラーコードに出会いやすい

- `unexpected token import`
- `require of ES Module`

↑問題の原因はimportにいます

## importの歴史

### モジュールが存在する以前のjsのimport

```html
<!-- global jQuery objectを定義 -->
<script src="https://cdn.com/jquery.js"></script>
<!-- global lodash objectを定義 -->
<script src="https://cdn.com/lodash.js"></script>
<!-- global objectを参照して使用 -->
<script>
    jQuery(document).ready(()=>{
        lodash.get(obj, 'foo');
    });
</script>
```

上記の問題点

- １つの巨大なファイル
- 変数がglobalに適用されるので、コンフリクトの恐れがある

### CommonJSを使う時のimport

```ts
const jQuery = require('jQuery');
const lodash = require('Lodash');

jQuery(document).ready(()=>{
    lodash.get(obj, 'foo');
});
```

↑のように個別の変数として使えるようになりました。

```ts
exports.add = function(a, b){
    return a + b;
}

const { add } = require('/add.js');
console.log()
```

↑簡単にexportもできるようになりました。

`commonjs`のメリット

- ファイル単位の開発ができるようになった。
- 数百、数千件のJSファイルに分離ができるようになった。
- 簡単にライブラリーの関数を再利用できるようになった

#### 疑問：私たちはnodeで開発する時、import/exportを使っていますが

それは`typescript`と`babel`を使っているためです、以下のように`import`がrequireに変換されます。

```ts
import React from 'react'
// ↓　Typescript compiler or Babel
const React = require('react')
```

`commonjs`デメリット

- 言語の標準ではない。
- nodeをサポートする環境でのみ利用的る
  - browserやdenoで使えない

#### 他の`commonjs`の問題点

1. 定的分析が難しい

    ```ts
    // 分岐によるrequire
    if(condition) React = require('react');

    // 分岐によるrequire
    require(condition ? 'react' : 'angular');

    // 変数名を変えて再定義
    const originalRequire = global.require;
    originalRequire(...);
    ```

    このようなコードは複雑度を上げ、browserのtree shakingの効率が下がる

2. 非同期モジュールを定義が難しい

    ```ts
    let initialized = false

    exports.initialize = async function initialize() {
        if(initialized){
            throw error('already initialized')
        }

        await connectDB();
        initialized = true;
    }
    ```

    ↑のように毎回初期化を確認するしかない

    基本的に同期的に動くので、jsの強い非同期と相性が悪い

3. 静かなrequire関数の再定義

    `require`は基本的に関数なので、勝手に定義が可能(例 jestなどのテストframework)

    ```ts
    const defaultRequire = global.require;
    const myRequire = (param:string) => {
        ...
    };

    global.require = myRequire
    ```

### ECMAScript Modules (ESM)

sample

```ts
export function add(x, y){
    return x + y;
}

import { add } from "./add.js"
console.log(add(1, 2))
```

メリット

- 定的分析がしやすい

    ```ts
    // エラー
    if(condition) import React from "react";

    // エラー
    import something from condition ? "react" : "angular";

    // エラー
    const myImport = import;
    myImport React from "react";
    ```

- top-level awaitを利用した、非同期モジュール

    ```ts
    const db = await connectDB(); 
    ```

- 言語標準

    node（v12↑）だけではなくbrowserやdenoでも利用できる

## エラーの原因

- 非同期コードで同期コードを利用：簡単
- 同期コードで非同期コードを利用：難しい（同期関数が非同期関数になる必要）

node(同期)で既存commonJSのpackage（同期）はrequireしやすい

node(同期)でESMのpackage（非同期 ex ky）はrequireが難しい

## package.json type:module

```json
 {
    ...
    "type": "module" // default commonjs 
 }
```

ESMタイプのモジュールに対応するため、`type`を追加してます。

## 移行までの難しさ

`ESM`は`typescript`4.7からサポートをしている。

移行できるサービス

1. Typescriptを使ってない時
2. 利用するligがESMをサポートする時
3. jest yarn ts-node などを利用しないとき

## 参考記事

- [ESMでのimport](https://www.youtube.com/watch?v=mee1QbvaO10)
- [MDN web docs import](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Statements/import)
