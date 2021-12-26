# Adding Interactivity

ライフゲームの実装にいくつかのインタラクティブな機能を追加することにより、JavaScriptとWebAssemblyのインターフェースを引き続き調査します。
ユーザーがセルをクリックすることでセルが死活を切り替えることができるようにし、ゲームを一時停止できるようにします。
これにより、セルパターンの描画がはるかに簡単になります。


## ゲームの一時停止と再開

ゲームをプレイしているか一時停止しているかを切り替えるボタンを追加しましょう。
`wasm-game-of-life/www/index.html`に、`<canvas>`のすぐ上にボタンを追加します。

```html
<button id="play-pause"></button>
```

`wasm-game-of-life/www/index.js` JavaScriptで、次の変更を行います。

* `requestAnimationFrame`への最新の呼び出しによって返された識別子を追跡し、その識別子を使用して`cancelAnimationFrame`を呼び出すことでアニメーションをキャンセルできるようにします。

* 再生/一時停止ボタンをクリックしたときに、キューに入れられたアニメーションフレームの識別子があるかどうかを確認します。
  アニメーションフレーム識別子がある場合、ゲームは現在再生中です。
  アニメーションフレームをキャンセルして、`renderLoop`が再度呼び出されないようにし、ゲームを効果的に一時停止します。
  キューに入れられたアニメーションフレームの識別子がない場合は、現在一時停止されているため、`requestAnimationFrame`を呼び出してゲームを再開します。

JavaScriptがRustとWebAssemblyを駆動しているので、これが私たちがする必要があるすべてであり、Rustソースを変更する必要はありません。

`requestAnimationFrame`によって返される識別子を追跡するために`animationId`変数を導入します。
キューに入れられたアニメーションフレームがない場合、この変数を `null`に設定します。

```js
let animationId = null;

// This function is the same as before, except the
// result of `requestAnimationFrame` is assigned to
// `animationId`.
const renderLoop = () => {
  drawGrid();
  drawCells();

  universe.tick();

  animationId = requestAnimationFrame(renderLoop);
};
```

At any instant in time, we can tell whether the game is paused or not by
inspecting the value of `animationId`:

```js
const isPaused = () => {
  return animationId === null;
};
```

これで、再生/一時停止ボタンをクリックすると、ゲームが現在一時停止しているか再生中であるかを確認し、`renderLoop`アニメーションを再開するか、次のアニメーションフレームをキャンセルします。
さらに、ボタンのテキストアイコンを更新して、次にクリックしたときにボタンが実行するアクションを反映します。

```js
const playPauseButton = document.getElementById("play-pause");

const play = () => {
  playPauseButton.textContent = "⏸";
  renderLoop();
};

const pause = () => {
  playPauseButton.textContent = "▶";
  cancelAnimationFrame(animationId);
  animationId = null;
};

playPauseButton.addEventListener("click", event => {
  if (isPaused()) {
    play();
  } else {
    pause();
  }
});
```
最後に、以前は `requestAnimationFrame(renderLoop)`を直接呼び出してゲームとそのアニメーションをキックスタートしていましたが、ボタンが正しい初期テキストアイコンを取得するように、これを `play`の呼び出しに置き換えたいと思います。

```diff
// This used to be `requestAnimationFrame(renderLoop)`.
play();
```

[http://localhost:8080/](http://localhost:8080/)を更新すると、ボタンをクリックしてゲームを一時停止および再開できるようになります。


## `"クリック"`イベントでセルの状態を切り替える

ゲームを一時停止できるようになったので、セルをクリックしてセルを変更する機能を追加します。

セルを切り替えるとは、その状態を活性状態から非活性状態に、または非活性状態から活性状態に切り替えることです。
`wasm-game-of-life/src/lib.rs`の`Cell`に `toggle`メソッドを追加します。

```rust
impl Cell {
    fn toggle(&mut self) {
        *self = match *self {
            Cell::Dead => Cell::Alive,
            Cell::Alive => Cell::Dead,
        };
    }
}
```

特定の行と列のセルの状態を切り替えるには、行と列のペアをセルベクトルのインデックスに変換し、そのインデックスのセルで toggle メソッドを呼び出します。

```rust
/// Public methods, exported to JavaScript.
#[wasm_bindgen]
impl Universe {
    // ...

    pub fn toggle_cell(&mut self, row: u32, column: u32) {
        let idx = self.get_index(row, column);
        self.cells[idx].toggle();
    }
}
```

このメソッドは、JavaScriptで呼び出せるように、 `#[wasm_bindgen]`アノテーションが付けられた `impl`ブロック内で定義されています。

`wasm-game-of-life/www/index.js`では、`<canvas>`要素のクリックイベントをリッスンし、クリックイベントのページ相対座標をキャンバス相対座標に変換してから行に変換します そしてcolumn、 `toggle_cell`メソッドを呼び出し、最後にシーンを再描画します。

```js
canvas.addEventListener("click", event => {
  const boundingRect = canvas.getBoundingClientRect();

  const scaleX = canvas.width / boundingRect.width;
  const scaleY = canvas.height / boundingRect.height;

  const canvasLeft = (event.clientX - boundingRect.left) * scaleX;
  const canvasTop = (event.clientY - boundingRect.top) * scaleY;

  const row = Math.min(Math.floor(canvasTop / (CELL_SIZE + 1)), height - 1);
  const col = Math.min(Math.floor(canvasLeft / (CELL_SIZE + 1)), width - 1);

  universe.toggle_cell(row, col);

  drawGrid();
  drawCells();
});
```

`wasm-game-of-life`で`wasm-pack build`を使用して再構築し、[http://localhost:8080/](http://localhost:8080/)を再度更新すると、独自のパターンを描画できるようになります。
セルをクリックして、それらの状態を切り替えます。

## 演習

* [`<input type="range">`][input-range]ウィジェットを導入して、アニメーションフレームごとに発生するティック数を制御せよ。

* クリックすると宇宙をランダムな初期状態にリセットするボタンと、宇宙をすべての非活性セルにリセットするボタンを追加せよ。

* `Ctrl+Click`で、ターゲットセルの中央に[グライダー](https://en.wikipedia.org/wiki/Glider_(Conway％27s_Life))を、`Shift+Click`で、パルサーを挿入せよ。

[input-range]: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/input/range
