# デバッグする

さらに多くのコードを書く前に、問題が発生した場合に備えて、いくつかのデバッグツールを用意しておく必要があります。
[Rustで生成されたWebAssemblyのデバッグに使用できるツールとアプローチを一覧表示するリファレンスページ](reference-debugging)を確認してください。

[reference-debugging]: ../reference/debugging.html

## パニック時のログを有効にする

[コードがパニックになった場合、開発者コンソールに有益なエラーメッセージを表示する必要があります。](../reference/debugging.html#logging-panics)
`wasm-pack-template`には、[wasm-game-of-life/src/utils.rsで設定されている`console_error_panic_hook`クレート](panic-hook)へのオプションのデフォルトで有効な依存関係が付属しています。
必要なことは、初期化関数または共通のコードパスにフックをインストールすることだけです。 `wasm-game-of-life/src/lib.rs`の`Universe::new`コンストラクター内で呼び出すことができます。

```rust
pub fn new() -> Universe {
    utils::set_panic_hook();

    // ...
}
```

[panic-hook]: https://github.com/rustwasm/console_error_panic_hook

## ライフゲームにログを追加する

`Universe::tick`関数の各セルについて[`web-sys`クレートを介して `console.log`関数を使用してロギングを追加](logging)しましょう。

まず、依存関係として `web-sys`を追加し、`wasm-game-of-life/Cargo.toml`でその `"console"`機能を有効にします。

```toml
[dependencies]

# ...

[dependencies.web-sys]
version = "0.3"
features = [
  "console",
]
```

使いやすくするために、 `console.log`関数を` println！ `スタイルのマクロでラップします。

[logging]: ../reference/debugging.html#logging-with-the-console-apis

```rust
extern crate web_sys;

// A macro to provide `println!(..)`-style syntax for `console.log` logging.
macro_rules! log {
    ( $( $t:tt )* ) => {
        web_sys::console::log_1(&format!( $( $t )* ).into());
    }
}
```

これで、Rustコードに `log`への呼び出しを挿入することで、コンソールへのメッセージのログ記録を開始できます。
たとえば、各セルの状態、近傍の活性セルの数、次の状態をログに記録するには、次のように `wasm-game-of-life/src/lib.rs`を変更できます。

```diff
diff --git a/src/lib.rs b/src/lib.rs
index f757641..a30e107 100755
--- a/src/lib.rs
+++ b/src/lib.rs
@@ -123,6 +122,14 @@ impl Universe {
                 let cell = self.cells[idx];
                 let live_neighbors = self.live_neighbor_count(row, col);

+                log!(
+                    "cell[{}, {}] is initially {:?} and has {} live neighbors",
+                    row,
+                    col,
+                    cell,
+                    live_neighbors
+                );
+
                 let next_cell = match (cell, live_neighbors) {
                     // Rule 1: Any live cell with fewer than two live neighbours
                     // dies, as if caused by underpopulation.
@@ -140,6 +147,8 @@ impl Universe {
                     (otherwise, _) => otherwise,
                 };

+                log!("    it becomes {:?}", next_cell);
+
                 next[idx] = next_cell;
             }
         }
```

## デバッガーを使用して各ステップで一時停止する

[ブラウザのステッピングデバッガは、Rustで生成されたWebAssemblyが相互作用するJavaScriptを検査するのに役立ちます。](../reference/debugging.html#using-a-debugger)

たとえば、デバッガーを使用して、 `universe.tick()`の呼び出しの上に[JavaScript`debugger`ステートメント](dbg-stmt)を配置することで、`renderLoop`関数の各反復で一時停止できます。

```js
const renderLoop = () => {
  debugger;
  universe.tick();

  drawGrid();
  drawCells();

  requestAnimationFrame(renderLoop);
};
```

これにより、ログに記録されたメッセージを検査し、現在レンダリングされているフレームを前のフレームと比較するための便利なチェックポイントが提供されます。

[dbg-stmt]:https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/debugger

[![Screenshot of debugging the Game of Life](../images/game-of-life/debugging.png)](../images/game-of-life/debugging.png)

## 演習

* 状態を活性から非活性に、またはその逆に遷移した各セルの行と列を記録する `tick`関数にログを追加します。

* `Universe::new`メソッドに`panic!()`を導入します。
  WebブラウザのJavaScriptデバッガでパニックのバックトレースを調べます。
  デバッグシンボルを無効にし、 `console_error_panic_hook`オプションの依存関係なしで再構築し、スタックトレースを再度検査します。それほど役に立ちませんか？
