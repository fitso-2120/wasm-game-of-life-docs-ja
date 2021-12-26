# Implementing Conway's Game of Life

## 設計

Before we dive in, we have some design choices to consider.
始める前に、考慮すべきいくつかの設計上の選択肢があります。

### 無限の宇宙

ライフゲームは無限の宇宙でプレイされますが、無限のメモリと計算能力はありません。
このやや厄介な制限を回避するには、通常、次の3つのフレーバーのいずれかがあります。

1. 宇宙のどのサブセットで興味深いことが起こっているかを追跡し、必要に応じてこの領域を拡張します。
  最悪の場合、この拡張には制限がなく、実装はどんどん遅くなり、最終的にはメモリが不足します。

2. 固定サイズのユニバースを作成します。
  この宇宙では、端のセルの隣接セルが中央のセルよりも少なくなっています。
  このアプローチの欠点は、宇宙の末端に到達するグライダーのような無限のパターンが消されてしまうことです。

3. 固定サイズの周期的なユニバースを作成します。
  ここで、端のセルには、宇宙の反対側にラップアラウンドする隣接セルがあります。
  隣がが宇宙の端を包み込むので、グライダーは永遠に走り続けることができます。
  (宇宙の形がトーラス - ドーナツで、高さが小円の円周の分割数、幅が大円の円周の分割数に当たります)

3番目のオプションを実装します。


### RustとJavaScriptのインターフェース

> ⚡ これは、このチュートリアルを理解して取り除くための最も重要な概念の1つです。

JavaScriptのガベージコレクションヒープ（ `Object`s、`Array`s、およびDOMノードが割り当てられる）は、Rust値が存在するWebAssemblyの線形メモリ空間とは異なります。

WebAssemblyは現在、ガベージ収集されたヒープに直接アクセスできません（2018年4月の時点で、これは["インターフェイスタイプ"提案][interface-types]で変更される予定です）。

一方、JavaScriptは、WebAssembly線形メモリ空間の読み取りと書き込みを行うことができますが、スカラー値（ `u8`、`i32`、 `f64`などの[`ArrayBuffer`][array-buf]としてのみ可能です...）。

WebAssembly関数もスカラー値を受け取り、返します。

これらは、すべてのWebAssemblyおよびJavaScript通信を構成する構成要素です。

[interface-types]: https://github.com/WebAssembly/interface-types/blob/master/proposals/interface-types/Explainer.md
[array-buf]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer

`wasm_bindgen`は、この境界を越えて複合構造を操作する方法の一般的な理解を定義します。

これには、Rust構造をボックス化し、使いやすさのためにポインターをJavaScriptクラスでラップするか、RustからJavaScriptオブジェクトのテーブルにインデックスを付けることが含まれます。 `wasm_bindgen`は非常に便利ですが、データ表現、およびこの境界を越えて渡される値と構造を考慮する必要がなくなるわけではありません。

代わりに、選択したインターフェイスデザインを実装するためのツールと考えてください。

WebAssemblyとJavaScriptの間のインターフェースを設計するとき、次のプロパティを最適化する必要があります。

1. **WebAssemblyリニアメモリへのコピーとWebAssemblyリニアメモリからのコピーを最小限に抑えます。**
   不要なコピーは、不要なオーバーヘッドを課します。

2. **シリアル化と逆シリアル化を最小限に抑えます。**
   コピーと同様に、シリアル化と逆シリアル化もオーバーヘッドを課し、多くの場合、コピーも課します。
   不透明なハンドルをデータ構造に渡すことができれば（一方の側でシリアル化し、WebAssemblyリニアメモリの既知の場所にコピーし、もう一方の側で逆シリアル化する代わりに）、多くの場合、オーバーヘッドを大幅に削減できます。
   `wasm_bindgen`は、JavaScriptの` Object`またはボックス化されたRust構造体への不透明なハンドルを定義して操作するのに役立ちます。

一般的なルールとして、優れたJavaScript↔WebAssemblyインターフェイスデザインは、多くの場合、WebAssembly線形メモリに存在するRustタイプとして大規模で長寿命のデータ構造が実装され、不透明なハンドルとしてJavaScriptに公開されるデザインです。

JavaScriptは、これらの不透明なハンドルを取り、データを変換し、重い計算を実行し、データをクエリし、最終的に小さなコピー可能な結果を返すエクスポートされたWebAssembly関数を呼び出します。

計算の小さな結果を返すだけで、JavaScriptのガベージコレクションヒープとWebAssembly線形メモリの間ですべてをコピーしたりシリアル化したりすることを回避できます。


### ライフゲームでのRustとJavaScriptのインターフェース

避けるべきいくつかの危険を列挙することから始めましょう。
ステップごとに宇宙全体をWebAssembly線形メモリにコピーしたり、WebAssembly線形メモリからコピーしたりする必要はありません。
宇宙内のすべてのセルにオブジェクトを割り当てたり、各セルの読み取りと書き込みに境界を越えた呼び出しを課したりする必要はありません。

これはどこに私たちを残しますか？ 宇宙、WebAssembly線形メモリに存在し、セルごとに1バイトのフラット配列として表すことができます。 `0`は非活性セルで、`1`は活性セルです。

これが4x4の宇宙が記憶の中でどのように見えるかです

![Screenshot of a 4 by 4 universe](../images/game-of-life/universe.png)

ユニバースの特定の行と列にあるセルの配列インデックスを見つけるには、次の数式を使用できます。

```text
index(row, column, universe) = row * width(universe) + column
```

宇宙のセルをJavaScriptに公開する方法はいくつかあります。
まず、 `Universe`に[`std::fmt::Display`][`Display`]を実装します。
これを使用して、テキスト文字としてレンダリングされたセルのRust`String`を生成できます。
このRust文字列は、WebAssembly線形メモリからJavaScriptのガベージコレクションヒープ内のJavaScript文字列にコピーされ、HTMLの `textContent`を設定して表示されます。
この章の後半では、ヒープ間で宇宙のセルをコピーすることを回避し、 `<canvas>`にレンダリングするために、この実装を進化させます。

*別の実行可能な設計の代替案は、宇宙全体をJavaScriptに公開する代わりに、Rustが各ステップの後に状態を変更したすべてのセルのリストを返すことです。 このように、JavaScriptはレンダリング時に宇宙全体を反復する必要はなく、関連するサブセットのみを反復する必要があります。 トレードオフは、このデルタベースの設計を実装するのが少し難しいことです。*

## Rustの実装

前の章では、最初のプロジェクトテンプレートのクローンを作成しました。 そのプロジェクトテンプレートを今すぐ変更します。

まず、 `wasm-game-of-life/src/lib.rs`から`alert`import関数と `greet`関数を削除し、それらをセルの型定義に置き換えます。

```rust
#[wasm_bindgen]
#[repr(u8)]
#[derive(Clone, Copy, Debug, PartialEq, Eq)]
pub enum Cell {
    Dead = 0,
    Alive = 1,
}
```

各セルが1バイトとして表されるように、 `#[repr(u8)]`があることが重要です。
また、`Dead`バリアントが`0`であり、`Alive`バリアントが`1`であることが重要です。
これにより、セルの隣接セルを追加して簡単にカウントできます。

次に、宇宙を定義しましょう。 宇宙には幅と高さがあり、長さ `width * height`のセルのベクトルがあります。

```rust
#[wasm_bindgen]
pub struct Universe {
    width: u32,
    height: u32,
    cells: Vec<Cell>,
}
```

特定の行と列のセルにアクセスするには、前述のように、行と列をセルベクトルのインデックスに変換します。

```rust
impl Universe {
    fn get_index(&self, row: u32, column: u32) -> usize {
        (row * self.width + column) as usize
    }

    // ...
}
```

セルの次の状態を計算するには、隣接するセルがいくつ活性かをカウントする必要があります。 それを行うために `live_neighbor_count`メソッドを書いてみましょう！

```rust
impl Universe {
    // ...

    fn live_neighbor_count(&self, row: u32, column: u32) -> u8 {
        let mut count = 0;
        for delta_row in [self.height - 1, 0, 1].iter().cloned() {
            for delta_col in [self.width - 1, 0, 1].iter().cloned() {
                if delta_row == 0 && delta_col == 0 {
                    continue;
                }

                let neighbor_row = (row + delta_row) % self.height;
                let neighbor_col = (column + delta_col) % self.width;
                let idx = self.get_index(neighbor_row, neighbor_col);
                count += self.cells[idx] as u8;
            }
        }
        count
    }
}
```

`live_neighbor_count`メソッドは、デルタとモジュロを使用して、宇宙の端を`if`で特別にケーシングすることを回避します。
`-1`のデルタを適用する場合、`1`を減算するのではなく、 `self.height - 1`を*加算*し、剰余にその処理を実行させます。 `row`と`column`は `0`にすることができ、それらから`1`を減算しようとすると、符号なし整数のアンダーフローが発生します。

これで、現在の世代から次世代を計算するために必要なすべてが揃いました。
ゲームの各ルールは、 `match`式の条件への直接的な変換に従います。
さらに、ステップが発生するタイミングをJavaScriptで制御する必要があるため、このメソッドを`#[wasm_bindgen]`ブロック内に配置して、JavaScriptに公開されるようにします。


```rust
/// JavaScriptに公開したパブリックメソッド。
#[wasm_bindgen]
impl Universe {
    pub fn tick(&mut self) {
        let mut next = self.cells.clone();

        for row in 0..self.height {
            for col in 0..self.width {
                let idx = self.get_index(row, col);
                let cell = self.cells[idx];
                let live_neighbors = self.live_neighbor_count(row, col);

                let next_cell = match (cell, live_neighbors) {
                    // Rule 1: Any live cell with fewer than two live neighbours
                    // dies, as if caused by underpopulation.
                    (Cell::Alive, x) if x < 2 => Cell::Dead,
                    // Rule 2: Any live cell with two or three live neighbours
                    // lives on to the next generation.
                    (Cell::Alive, 2) | (Cell::Alive, 3) => Cell::Alive,
                    // Rule 3: Any live cell with more than three live
                    // neighbours dies, as if by overpopulation.
                    (Cell::Alive, x) if x > 3 => Cell::Dead,
                    // Rule 4: Any dead cell with exactly three live neighbours
                    // becomes a live cell, as if by reproduction.
                    (Cell::Dead, 3) => Cell::Alive,
                    // All other cells remain in the same state.
                    (otherwise, _) => otherwise,
                };

                next[idx] = next_cell;
            }
        }

        self.cells = next;
    }

    // ...
}
```

これまでのところ、宇宙の状態はセルのベクトルとして表されています。
可読可能にするために、基本的なテキストレンダラーを実装しましょう。
アイデアは、宇宙を1行ずつテキストとして記述し、活性セルごとに、Unicode文字 `◼`（「黒い中程度の正方形」）を印刷することです。
非活性セルについては、 `◻`（「白い中程度の正方形」）を印刷します。

Rustの標準ライブラリから[`Display`]traitを実装することにより、ユーザー向けの方法で構造をフォーマットする方法を追加できます。 これにより、[`to_string`]メソッドも自動的に提供されます。

[`Display`]: https://doc.rust-lang.org/1.25.0/std/fmt/trait.Display.html
[`to_string`]: https://doc.rust-lang.org/1.25.0/std/string/trait.ToString.html

```rust
use std::fmt;

impl fmt::Display for Universe {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        for line in self.cells.as_slice().chunks(self.width as usize) {
            for &cell in line {
                let symbol = if cell == Cell::Dead { '◻' } else { '◼' };
                write!(f, "{}", symbol)?;
            }
            write!(f, "\n")?;
        }

        Ok(())
    }
}
```

最後に、活性セルと非活性セルの興味深いパターンで宇宙を初期化するコンストラクターと、`render`メソッドを定義します。

```rust
/// Public methods, exported to JavaScript.
#[wasm_bindgen]
impl Universe {
    // ...

    pub fn new() -> Universe {
        let width = 64;
        let height = 64;

        let cells = (0..width * height)
            .map(|i| {
                if i % 2 == 0 || i % 7 == 0 {
                    Cell::Alive
                } else {
                    Cell::Dead
                }
            })
            .collect();

        Universe {
            width,
            height,
            cells,
        }
    }

    pub fn render(&self) -> String {
        self.to_string()
    }
}
```

これで、ライフゲームの実装のRustの半分が完了しました。

`wasm-game-of-life`ディレクトリ内で`wasm-pack build`を実行して、WebAssemblyに再コンパイルします。

## JavaScriptを使用したレンダリング

まず、 `<pre>`要素を `wasm-game-of-life/www/index.html`に追加して、宇宙を`<script>`タグのすぐ上にレンダリングしましょう。

```html
<body>
  <pre id="game-of-life-canvas"></pre>
  <script src="./bootstrap.js"></script>
</body>
```

さらに、 `<pre>`をWebページの中央に配置する必要があります。
CSSフレックスボックスを使用してこのタスクを実行できます。
`wasm-game-of-life/www/index.html`の`<head>`内に次の`<style>`タグを追加します。

```html
<style>
  body {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
  }
</style>
```

`wasm-game-of-life/www/index.js`の上部で、古い`greet`関数ではなく `Universe`を取り込むようにインポートを修正しましょう。

```js
import { Universe } from "wasm-game-of-life";
```

また、追加したばかりの `<pre>`要素を取得して、新しいUniverseをインスタンス化します。

```js
const pre = document.getElementById("game-of-life-canvas");
const universe = Universe.new();
```

JavaScriptは[`requestAnimationFrame`ループ][requestAnimationFrame]で実行されます。
各反復で、現在の宇宙を`<pre>`に描画し、`Universe::tick`を呼び出します。

[requestAnimationFrame]: https://developer.mozilla.org/en-US/docs/Web/API/window/requestAnimationFrame

```js
const renderLoop = () => {
  pre.textContent = universe.render();
  universe.tick();

  requestAnimationFrame(renderLoop);
};
```

レンダリングプロセスを開始するには、レンダリングループの最初の反復を最初に呼び出すだけです。

```js
requestAnimationFrame(renderLoop);
```

開発サーバーがまだ実行されていることを確認し（ `wasm-game-of-life/www`内で`npm run start`を実行）、
これが[http://localhost:8080/](http://localhost:8080/)
次のようになります:

[![Screenshot of the Game of Life implementation with text rendering](../images/game-of-life/initial-game-of-life-pre.png)](../images/game-of-life/initial-game-of-life-pre.png)

## メモリから直接Canvasにレンダリングする

Rustで `String`を生成（および割り当て）してから、`wasm-bindgen`にそれを有効なJavaScript文字列に変換させると、宇宙のセルの不要なコピーが作成されます。
JavaScriptコードはすでに宇宙の幅と高さを認識しており、セルを直接構成するWebAssemblyの線形メモリを読み取ることができるため、 `render`メソッドを変更してセル配列の先頭へのポインターを返します。

また、Unicodeテキストをレンダリングする代わりに、[Canvas API]の使用に切り替えます。
チュートリアルの残りの部分では、このデザインを使用します。

[Canvas API]: https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API

`wasm-game-of-life/www/index.html`内で、前に追加した`<pre>`をレンダリングする`<canvas>`に置き換えましょう（これも`<body>`内のJavaScriptをロードする`<script>`の前にある必要があります）：

```html
<body>
  <canvas id="game-of-life-canvas"></canvas>
  <script src='./bootstrap.js'></script>
</body>
```

Rustの実装から必要な情報を取得するには、宇宙の幅、高さ、およびセル配列へのポインターのゲッター関数をさらに追加する必要があります。
これらはすべてJavaScriptにも公開されています。 `wasm-game-of-life/src/lib.rs`に次の追加を行います。

```rust
/// Public methods, exported to JavaScript.
#[wasm_bindgen]
impl Universe {
    // ...

    pub fn width(&self) -> u32 {
        self.width
    }

    pub fn height(&self) -> u32 {
        self.height
    }

    pub fn cells(&self) -> *const Cell {
        self.cells.as_ptr()
    }
}
```

次に、 `wasm-game-of-life/www/index.js`で、`wasm-game-of-life`から `Cell`もインポートし、キャンバスにレンダリングするときに使用する定数を定義しましょう。

```js
import { Universe, Cell } from "wasm-game-of-life";

const CELL_SIZE = 5; // px
const GRID_COLOR = "#CCCCCC";
const DEAD_COLOR = "#FFFFFF";
const ALIVE_COLOR = "#000000";
```

それでは、このJavaScriptコードの残りの部分を書き直して、 `<pre>`の `textContent`に書き込むのではなく、代わりに`<canvas>`に描画してみましょう。

```js
// Construct the universe, and get its width and height.
const universe = Universe.new();
const width = universe.width();
const height = universe.height();

// Give the canvas room for all of our cells and a 1px border
// around each of them.
const canvas = document.getElementById("game-of-life-canvas");
canvas.height = (CELL_SIZE + 1) * height + 1;
canvas.width = (CELL_SIZE + 1) * width + 1;

const ctx = canvas.getContext('2d');

const renderLoop = () => {
  universe.tick();

  drawGrid();
  drawCells();

  requestAnimationFrame(renderLoop);
};
```

セル間にグリッドを描画するには、等間隔の水平線のセットと等間隔の垂直線のセットを描画します。
これらの線は十字に交差してグリッドを形成します。

```js
const drawGrid = () => {
  ctx.beginPath();
  ctx.strokeStyle = GRID_COLOR;

  // Vertical lines.
  for (let i = 0; i <= width; i++) {
    ctx.moveTo(i * (CELL_SIZE + 1) + 1, 0);
    ctx.lineTo(i * (CELL_SIZE + 1) + 1, (CELL_SIZE + 1) * height + 1);
  }

  // Horizontal lines.
  for (let j = 0; j <= height; j++) {
    ctx.moveTo(0,                           j * (CELL_SIZE + 1) + 1);
    ctx.lineTo((CELL_SIZE + 1) * width + 1, j * (CELL_SIZE + 1) + 1);
  }

  ctx.stroke();
};
```

自動生成されたwasmモジュール `wasm_game_of_life_bg`で定義されている`memory`を介してWebAssemblyの線形メモリに直接アクセスできます。
セルを描画するには、宇宙のセルへのポインタを取得し、セルバッファをオーバーレイする`Uint8Array`を作成し、各セルを反復処理して、セルが死活に応じて、それぞれ白または黒の長方形を描画します。
ポインターとオーバーレイを操作することにより、ステップごとに境界を越えてセルをコピーすることを回避します。

```js
// Import the WebAssembly memory at the top of the file.
import { memory } from "wasm-game-of-life/wasm_game_of_life_bg";

// ...

const getIndex = (row, column) => {
  return row * width + column;
};

const drawCells = () => {
  const cellsPtr = universe.cells();
  const cells = new Uint8Array(memory.buffer, cellsPtr, width * height);

  ctx.beginPath();

  for (let row = 0; row < height; row++) {
    for (let col = 0; col < width; col++) {
      const idx = getIndex(row, col);

      ctx.fillStyle = cells[idx] === Cell.Dead
        ? DEAD_COLOR
        : ALIVE_COLOR;

      ctx.fillRect(
        col * (CELL_SIZE + 1) + 1,
        row * (CELL_SIZE + 1) + 1,
        CELL_SIZE,
        CELL_SIZE
      );
    }
  }

  ctx.stroke();
};
```

レンダリングプロセスを開始するには、上記と同じコードを使用して、レンダリングループの最初の反復を開始します。

```js
drawGrid();
drawCells();

requestAnimationFrame(renderLoop);
```

`requestAnimationFrame()`を呼び出す _前に_ `drawGrid()`と`drawCells()`を呼び出すことに注意してください。
これを行う理由は、変更を加える前に宇宙の_初期_状態が描画されるようにするためです。
代わりに単に `requestAnimationFrame(renderLoop)`と呼ぶと、描画された最初のフレームが実際には2番目の「ステップ」である `universe.tick()`の最初の呼び出しの _後_ になるという状況になります。

## It Works!

ルート `wasm-game-of-life`ディレクトリ内からこのコマンドを実行して、WebAssemblyとバインディンググルーを再構築します。

```
wasm-pack build
```

開発サーバーがまだ実行されていることを確認してください。 そうでない場合は、 `wasm-game-of-life/www`ディレクトリ内から再起動します。

```
npm run start
```

[http://localhost:8080/](http://localhost:8080/)を更新すると、エキサイティングな表示で迎えられるはずです！

[![Screenshot of the Game of Life implementation](../images/game-of-life/initial-game-of-life.png)](../images/game-of-life/initial-game-of-life.png)

余談ですが、[hashlife](https://en.wikipedia.org/wiki/Hashlife)と呼ばれるライフゲームを実装するための非常に優れたアルゴリズムもあります。
アグレッシブなメモ化を使用し、実行時間が長くなるほど、実際に*指数関数的に高速*になって将来の世代を計算できます。
それを考えると、なぜこのチュートリアルでハッシュライフを実装しなかったのか疑問に思われるかもしれません。
RustとWebAssemblyの統合に焦点を当てているこのテキストの範囲外ですが、hashlifeについて自分で学ぶことを強くお勧めします。

## 演習

* 単一の宇宙船で宇宙を初期化しなさい。

* 最初の宇宙をハードコーディングする代わりに、各セルが、死活の可能性が半々のランダムな宇宙を生成しなさい。

  *ヒントt: [the `js-sys` crate](https://crates.io/crates/js-sys)を使用して
  [the `Math.random` JavaScript function](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math/random)をインポートせよ。*

  <details>
    <summary>解答</summary>
    *まず、`wasm-game-of-life/Cargo.toml`のdependencyに`js-sys`を追加します。:*

    ```toml
    # ...
    [dependencies]
    js-sys = "0.3"
    # ...
    ```

    *次に`js_sys::Math::random`関数を使って乱数を利用します*

    ```rust
    extern crate js_sys;

    // ...

    if js_sys::Math::random() < 0.5 {
        // Alive...
    } else {
        // Dead...
    }
    ```
  </details>

* 各セルをバイトで表すと、セルの反復処理が簡単になりますが、メモリを浪費するという犠牲が伴います。
  各バイトは8ビットですが、各セルが死活を表すために必要なのは1ビットだけです。<br/>
  各セルが1ビットの領域のみを使用するようにデータ表現をリファクタリングせよ。

  <details>
    <summary>解答</summary>

    Rustでは`Vec<Cell>`の代わりに[the `fixedbitset` クリートと`FixedBitSet`型](https://crates.io/crates/fixedbitset)を使ってセルを表現できます。

    ```rust
    // Make sure you also added the dependency to Cargo.toml!
    extern crate fixedbitset;
    use fixedbitset::FixedBitSet;

    // ...

    #[wasm_bindgen]
    pub struct Universe {
        width: u32,
        height: u32,
        cells: FixedBitSet,
    }
    ```

    Universe コンストラクタは、以下のように修正します。:

    ```rust
    pub fn new() -> Universe {
        let width = 64;
        let height = 64;

        let size = (width * height) as usize;
        let mut cells = FixedBitSet::with_capacity(size);

        for i in 0..size {
            cells.set(i, i % 2 == 0 || i % 7 == 0);
        }

        Universe {
            width,
            height,
            cells,
        }
    }
    ```

    宇宙の次のステップでセルを更新するには、 `FixedBitSet`の`set`メソッドを使用します。

    ```rust
    next.set(idx, match (cell, live_neighbors) {
        (true, x) if x < 2 => false,
        (true, 2) | (true, 3) => true,
        (true, x) if x > 3 => false,
        (false, 3) => true,
        (otherwise, _) => otherwise
    });
    ```

    ビットの先頭へのポインタをJavaScriptに渡すには、 `FixedBitSet`をスライスに変換してから、スライスをポインタに変換します。

    ```rust
    #[wasm_bindgen]
    impl Universe {
        // ...

        pub fn cells(&self) -> *const u32 {
            self.cells.as_slice().as_ptr()
        }
    }
    ```
    JavaScriptでは、Wasmメモリから `Uint8Array`を作成することは以前と同じですが、セル数が、バイトではなくビット当たりであるため、配列の長さが`width * height`ではなく、 `width * height / 8`である点が異なります。
    
    ```js
    const cells = new Uint8Array(memory.buffer, cellsPtr, width * height / 8);
    ```

    インデックスと`Uint8Array`が与えられると
    *n<sup>対象の</sup>* ビットは次の関数で設定されます。

    ```js
    const bitIsSet = (n, arr) => {
      const byte = Math.floor(n / 8);
      const mask = 1 << (n % 8);
      return (arr[byte] & mask) === mask;
    };
    ```
    これらすべてを考慮すると、 `drawCells`の新しいバージョンは次のようになります。

    ```js
    const drawCells = () => {
      const cellsPtr = universe.cells();

      // This is updated!
      const cells = new Uint8Array(memory.buffer, cellsPtr, width * height / 8);

      ctx.beginPath();

      for (let row = 0; row < height; row++) {
        for (let col = 0; col < width; col++) {
          const idx = getIndex(row, col);

          // This is updated!
          ctx.fillStyle = bitIsSet(idx, cells)
            ? ALIVE_COLOR
            : DEAD_COLOR;

          ctx.fillRect(
            col * (CELL_SIZE + 1) + 1,
            row * (CELL_SIZE + 1) + 1,
            CELL_SIZE,
            CELL_SIZE
          );
        }
      }

      ctx.stroke();
    };
    ```

  </details>
