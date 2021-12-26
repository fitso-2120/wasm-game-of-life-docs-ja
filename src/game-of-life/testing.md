# コンウェイのライフゲームをテストする

JavaScriptを使用したブラウザーでのライフゲームレンダリングのRust実装ができたので、
Rustで生成されたWebAssembly関数のテストについて説明しましょう。

`tick`関数をテストして、期待どおりの出力が得られることを確認します。

次に、 `wasm_game_of_life/src/lib.rs`ファイルの既存の`impl Universe`ブロック内にいくつかのsetter関数とgetter関数を作成します。
さまざまなサイズの `Universe`を作成できるように、`set_width`関数と `set_height`関数を作成します。

```rust
#[wasm_bindgen]
impl Universe { 
    // ...

    /// Set the width of the universe.
    ///
    /// Resets all cells to the dead state.
    pub fn set_width(&mut self, width: u32) {
        self.width = width;
        self.cells = (0..width * self.height).map(|_i| Cell::Dead).collect();
    }

    /// Set the height of the universe.
    ///
    /// Resets all cells to the dead state.
    pub fn set_height(&mut self, height: u32) {
        self.height = height;
        self.cells = (0..self.width * height).map(|_i| Cell::Dead).collect();
    }

}
```

`#[wasm_bindgen]`属性なしで `wasm_game_of_life/src/lib.rs`ファイル内に別の`impl Universe`ブロックを作成します。
JavaScriptに公開したくない、テストに必要な関数がいくつかあります。
Rustで生成されたWebAssembly関数は、借用した参照を返すことはできません。
Rustで生成されたWebAssemblyを属性を使用してコンパイルし、発生するエラーを確認してください。

`Universe`の`cells`の内容を取得するために `get_cells`の実装を書きます。
また、 `set_cells`関数を記述して、`Universe`の特定の行と列の `cells`を`活性セル`に設定できるようにします。

```rust
impl Universe {
    /// Get the dead and alive values of the entire universe.
    pub fn get_cells(&self) -> &[Cell] {
        &self.cells
    }

    /// Set cells to be alive in a universe by passing the row and column
    /// of each cell as an array.
    pub fn set_cells(&mut self, cells: &[(u32, u32)]) {
        for (row, col) in cells.iter().cloned() {
            let idx = self.get_index(row, col);
            self.cells[idx] = Cell::Alive;
        }
    }

}
```

次に、 `wasm_game_of_life/tests/web.rs`ファイルにテストを作成します。

その前に、ファイルにはすでに1つの動作テストがあります。
`wasm-game-of-life`ディレクトリで`wasm-pack test --chrome --headless`を実行すると、Rustで生成されたWebAssemblyテストが機能していることを確認できます。
`--firefox`、`--safari`、および `--node`オプションを使用して、
それらのブラウザでコードをテストします。
`wasm_game_of_life/tests/web.rs`ファイルで、`wasm_game_of_life`クレートと `Universe`タイプをエクスポートする必要があります。

```rust
extern crate wasm_game_of_life;
use wasm_game_of_life::Universe;
```

`wasm_game_of_life/tests/web.rs`ファイルで、いくつかの宇宙船ビルダー関数を作成します。

`tick`関数を呼び出す入力宇宙船用に1つ必要であり、1ステップ後に取得する予定の宇宙船が必要です。
`input_spaceship`関数で宇宙船を作成するために、`Alive`として初期化するセルを選択しました。
`expected_spaceship`関数内の宇宙船の位置は、`input_spaceship`のステップで手動で計算されました。
1ステップ後の入力宇宙船のセルが、予想される宇宙船と同じであることを自分で確認できます。

```rust
#[cfg(test)]
pub fn input_spaceship() -> Universe {
    let mut universe = Universe::new();
    universe.set_width(6);
    universe.set_height(6);
    universe.set_cells(&[(1,2), (2,3), (3,1), (3,2), (3,3)]);
    universe
}

#[cfg(test)]
pub fn expected_spaceship() -> Universe {
    let mut universe = Universe::new();
    universe.set_width(6);
    universe.set_height(6);
    universe.set_cells(&[(2,1), (2,3), (3,2), (3,3), (4,2)]);
    universe
}
```

次に、`test_tick`関数の実装を記述します。
まず、`input_spaceship()`と`expected_spaceship()`のインスタンスを作成します。
次に、`input_universe`で`tick`を呼び出します。
最後に、 `assert_eq!`マクロを使用して `get_cells()`を呼び出し、`input_universe`と`expected_universe`が同じ`Cell`配列値を持つようにします。
コードブロックに `#[wasm_bindgen_test]`属性を追加して、Rustで生成されたWebAssemblyコードをテストし、 `wasm-pack test`を使用してWebAssemblyコードをテストできるようにします。

```rust
#[wasm_bindgen_test]
pub fn test_tick() {
    // Let's create a smaller Universe with a small spaceship to test!
    let mut input_universe = input_spaceship();

    // This is what our spaceship should look like
    // after one tick in our universe.
    let expected_universe = expected_spaceship();

    // Call `tick` and then see if the cells in the `Universe`s are the same.
    input_universe.tick();
    assert_eq!(&input_universe.get_cells(), &expected_universe.get_cells());
}
```

`wasm-pack test --firefox --headless`を実行して、`wasm-game-of-life`ディレクトリ内でテストを実行します。
