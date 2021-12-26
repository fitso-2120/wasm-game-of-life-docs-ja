# Hello World!

このセクションでは、最初のRustとWebAssemblyをビルドして実行する方法を説明します
プログラム：「Hello、World!」を警告するWebページ

開始する前に、[セットアップ手順](setup.html)に従っていることを確認してください。

## プロジェクトテンプレートのクローンを作成する

プロジェクトテンプレートは、適切なデフォルトで事前構成されているため、すばやく実行できます
Web用のコードをビルド、統合、およびパッケージ化します。

次のコマンドを使用して、プロジェクトテンプレートのクローンを作成します。

```text
cargo generate --git https://github.com/rustwasm/wasm-pack-template
```

これにより、新しいプロジェクトの名前の入力を求められます。我々は使用するだろう
**"wasm-game-of-life"**。

```text
wasm-game-of-life
```

## 中身

新しい `wasm-game-of-life`プロジェクトに参加してください

```
cd wasm-game-of-life
```

そしてその内容を見てみましょう：

```text
wasm-game-of-life/
├──Cargo.toml
├──LICENSE_APACHE
├──LICENSE_MIT
├──README.md
└──src
    ├──lib.rs
    └──utils.rs
```

これらのファイルのいくつかを詳しく見てみましょう。

### `wasm-game-of-life/Cargo.toml`

`Cargo.toml`ファイルは`cargo`の依存関係とメタデータを指定します。
パッケージマネージャーとビルドツール。これは事前設定されています
`wasm-bindgen`依存関係、後で掘り下げるいくつかのオプションの依存関係、
そして、 `.wasm`ライブラリを生成するために適切に初期化された`crate-type`。

### `wasm-game-of-life/src/lib.rs`

`src/lib.rs`ファイルは、コンパイル先のRustクレートのルートです。
WebAssembly。 JavaScriptとのインターフェースに `wasm-bindgen`を使用します。インポートします
`window.alert` JavaScript関数、および`greet` Rust関数をエクスポートします。
グリーティングメッセージを警告します。

```rust
mod utils;

wasm_bindgen::prelude::*を使用します。

// `wee_alloc`機能が有効になっている場合、グローバルとして`wee_alloc`を使用します
//allocator.
#[cfg（feature = "wee_alloc"）]
#[global_allocator]
static ALLOC: wee_alloc::WeeAlloc = wee_alloc::WeeAlloc::INIT;

#[wasm_bindgen]
extern {
    fn alert(s: &str);
}

#[wasm_bindgen]
pub fn greet（）{
    alert("Hello, wasm-game-of-life!");
}
```

### `wasm-game-of-life/src/utils.rs`

`src/utils.rs`モジュールは、Rustを操作するための一般的なユーティリティを提供します
WebAssemblyに簡単にコンパイルできます。これらのユーティリティのいくつかを見ていきます
[wasmコードのデバッグ](debugging.html)など、チュートリアルの後半で詳しく説明しますが、今のところこのファイルは無視してかまいません。

## プロジェクトをビルドする

`wasm-pack`を使用して、次のビルド手順を調整します。

* Rust 1.30以降があり、 `wasm32-unknown-unknown`ターゲットが`rustup`を介してインストールされていることを確認してください。
* Rustソースを `cargo`を介してWebAssembly`.wasm`バイナリにコンパイルします。
* `wasm-bindgen`を使用して、Rustで生成されたWebAssemblyを使用するためのJavaScriptAPIを生成します。

これらすべてを行うには、プロジェクトディレクトリ内で次のコマンドを実行します。

```
wasm-pack build
```

ビルドが完了すると、そのアーティファクトは `pkg`ディレクトリにあります。
そしてそれはこれらの内容を持っているべきです：

```
pkg/
├──package.json
├──README.md
├──wasm_game_of_life_bg.wasm
├──wasm_game_of_life.d.ts
└──wasm_game_of_life.js
```

`README.md`ファイルはメインプロジェクトからコピーされていますが、他のファイルは完全に新しいものです。

### `wasm-game-of-life/pkg/wasm_game_of_life_bg.wasm`

`.wasm`ファイルは、RustソースからRustコンパイラによって生成されるWebAssemblyバイナリです。
これには、すべてのRust関数とデータのwasmにコンパイルされたバージョンが含まれています。
たとえば、エクスポートされた "greet" 機能があります。

### `wasm-game-of-life/pkg/wasm_game_of_life.js`

`.js`ファイルは`wasm-bindgen`によって生成され、JavaScriptの接着剤が含まれています
DOMおよびJavaScript関数をRustにインポートし、WebAssembly関数への優れたAPIをJavaScriptに公開します。
たとえば、WebAssemblyモジュールからエクスポートされた `greet`関数をラップするJavaScriptの`greet`関数があります。
今のところ、この接着剤はあまり効果がありませんが、もっと通過し始めるとwasmとJavaScriptの間を行き来する興味深い値は、境界を越えてそれらの値をシェパードするのに役立ちます。

```js
import * as wasm from './wasm_game_of_life_bg';

//...

export function greet() {
    return wasm.greet();
}
```

### `wasm-game-of-life/pkg/wasm_game_of_life.d.ts`

`.d.ts`ファイルには、JavaScriptの[TypeScript] []型宣言が含まれています。
TypeScriptを使用している場合は、WebAssemblyを呼び出すことができます。
関数のタイプがチェックされ、IDEは自動完了と提案を提供できます！
TypeScriptを使用していない場合は、このファイルを無視しても問題ありません。

```typescript
export function greet(): void;
```

[TypeScript]:http://www.typescriptlang.org/

### `wasm-game-of-life/pkg/package.json`

[`package.json`ファイルには、生成されたJavaScriptおよびWebAssemblyパッケージに関するメタデータが含まれています。][package.json]
これは、npmおよびJavaScriptバンドラーが、パッケージ、パッケージ名、バージョン、およびその他の多数の依存関係を判別するために使用します。 JavaScriptツールとの統合に役立ち、パッケージをnpmに公開できます。

```json
{
  "name": "wasm-game-of-life",
 "collaborators": [
    "Your Name<your.email@example.com>"
  ],
  "description": null,
  "version": "0.1.0",
  "license": null,
  "repository": null,
  "files": [
    "wasm_game_of_life_bg.wasm",
    "wasm_game_of_life.d.ts"
  ],
  "main": "wasm_game_of_life.js",
  "types": "wasm_game_of_life.d.ts"
}
```

[package.json]:https://docs.npmjs.com/files/package.json

## Webページに入れる

`wasm-game-of-life`パッケージを取得してWebページで使用するには、[`create-wasm-app`JavaScriptプロジェクトテンプレート][create-wasm-app]を使用します。

[create-wasm-app]:https://github.com/rustwasm/create-wasm-app

`wasm-game-of-life`ディレクトリ内で次のコマンドを実行します。

```
npm init wasm-app www
```

新しい `wasm-game-of-life/www`サブディレクトリに含まれるものは次のとおりです。

```
wasm-game-of-life/www/
├── bootstrap.js
├── index.html
├── index.js
├── LICENSE-APACHE
├── LICENSE-MIT
├── package.json
├── README.md
└── webpack.config.js
```

もう一度、これらのファイルのいくつかを詳しく見てみましょう。

### `wasm-game-of-life/www/package.json`

このpackage.jsonには、webpackとwebpack-dev-serverの依存関係、およびnpmに公開された最初のwasm-pack-templateパッケージのバージョンであるhello-wasm-packへの依存関係が事前構成されています。

### `wasm-game-of-life/www/webpack.config.js`

このファイルは、webpackとそのローカル開発サーバーを構成します。事前設定されており、webpackとそのローカル開発サーバーを機能させるためにこれを微調整する必要はありません。

### `wasm-game-of-life/www/index.html`

これは、WebページのルートHTMLファイルです。 `index.js`の非常に薄いラッパーである`bootstrap.js`をロードする以外はほとんど何もしません。

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>Hello wasm-pack!</title>
  </head>
  <body>
    <script src="./bootstrap.js"></script>
  </body>
</html>
```

### `wasm-game-of-life/www/index.js`

`index.js`は、WebページのJavaScriptのメインエントリポイントです。デフォルトの `wasm-pack-template`のコンパイル済みWebAssemblyとJavaScriptglueを含む`hello-wasm-pack` npmパッケージをインポートし、 `hello-wasm-pack`の`greet`関数を呼び出します。

```js
import * as wasm from "hello-wasm-pack";

wasm.greet();
```

### 依存関係をインストールします

まず、 `wasm-game-of-life/ www`サブディレクトリ内で`npm install`を実行して、ローカル開発サーバーとその依存関係がインストールされていることを確認します。

```text
npm install
```

このコマンドは1回だけ実行する必要があり、 `webpack`JavaScriptバンドラーとその開発サーバーをインストールします。

> `webpack`は、RustとWebAssemblyを操作するために必要ではないことに注意してください。
> これは、ここで便宜上選択したバンドラーと開発サーバーにすぎません。
> ParcelとRollupは、ECMAScriptモジュールとしてのWebAssemblyのインポートもサポートする必要があります。
> 必要に応じて、RustとWebAssembly [バンドラーなし][]を使用することもできます。

[バンドラーなし]:https://rustwasm.github.io/docs/wasm-bindgen/examples/without-a-bundler.html

### `www`でローカルの`wasm-game-of-life`パッケージを使用する

npmの `hello-wasm-pack`パッケージを使用するのではなく、代わりにローカルの`wasm-game-of-life`パッケージを使用したいと思います。これにより、ゲームオブライフプログラムを段階的に開発できるようになります。

`wasm-game-of-life/www/package.json`を開き、`"devDependencies" `の上に`"wasm-game-of-life":"file:../pkg"`エントリーを含む`"dependencies"`フィールドを追加します。


```js
{
  //...
  "dependencies"：{   //この3行のブロックを追加します！
    "wasm-game-of-life":"file:../pkg"
  }、
  "devDependencies"：{
    //...
  }
}
```

次に、 `wasm-game-of-life/www/index.js`を変更して、`hello-wasm-pack`パッケージの代わりに `wasm-game-of-life`をインポートします。

```js
import * as wasm from "wasm-game-of-life";

wasm.greet();
```

新しい依存関係を宣言したので、それをインストールする必要があります。
```text
npm install
```

これで、Webページをローカルで提供する準備が整いました。

## ローカルでのサービス

次に、開発サーバー用の新しいターミナルを開きます。サーバーを新しい端末で実行すると、サーバーをバックグラウンドで実行したままにすることができ、その間、他のコマンドの実行をブロックすることはありません。新しいターミナルで、 `wasm-game-of-life/www`ディレクトリ内から次のコマンドを実行します。


```
npm run start
```

Webブラウザを[http://localhost:8080/](http://localhost:8080/)に移動すると、次のアラートメッセージが表示されます。

[!["Hello、wasm-game-of-life!" のスクリーンショットWebページアラート](../images/game-of-life/hello-world.png)](../images/game-of-life/hello-world.png)

変更を加えて反映させたいときはいつでも
[http://localhost:8080/](http://localhost:8080/), `wasm-game-of-life`ディレクトリ内で`wasm-pack build`コマンドを再実行するだけです。

## 演習

* `wasm-game-of-life/src/lib.rs`の`greet`関数を変更して、アラートメッセージをカスタマイズする `name:&str`パラメーターを取得し、内部から`greet`関数に名前を渡します。 `wasm-game-of-life/www/index.js`。 `.wasm`バイナリを`wasm-pack build`で再構築し、Webブラウザで[http://localhost:8080/](http://localhost:8080/)を更新すると、カスタマイズされた挨拶が表示されます。
   <details>
     <summary>Answer</summary>

     `wasm-game-of-life/src/lib.rs`の`greet`関数の新しいバージョン：

     ```rust
      #[wasm_bindgen]
      pub fn greet(name: &str) {
        alert(&format!("Hello, {}!", name));
      }
     ```

     `wasm-game-of-life/www/index.js`での`greet`の新しい呼び出し：

     ```js
     wasm.greet("Your Name");
     ```

   </details>