# `.wasm`のサイズを縮小する

ライフゲーム・アプリケーションなど、ネットワークを介してクライアントに提供する `.wasm`のサイズに注意する必要があります。
`.wasm`が小さいほど、ページの読み込みが速くなり、ユーザーは幸せになります。

## ビルド構成を介して、ライフゲームの`.wasm`をどれだけ小さくすることができますか？

[微調整して`.wasm`のサイズを小さくできるビルド構成オプションを確認してください。](../reference/code-size.html#optimizing-builds-for-code-size)

デフォルトのリリースビルド構成（デバッグシンボルなし）では、WebAssemblyバイナリは29,410バイトです。

```
$ wc -c pkg/wasm_game_of_life_bg.wasm
29410 pkg/wasm_game_of_life_bg.wasm
```

LTOを有効にし、 `opt-level ="z"`を設定し、`wasm-opt -Oz`を実行すると、結果の`.wasm`はわずか17,317バイトに縮小されます。

```
$ wc -c pkg/wasm_game_of_life_bg.wasm
17317 pkg/wasm_game_of_life_bg.wasm
```

さらに、`gzip`（ほぼすべてのHTTPサーバーが行う）で圧縮すると、わずか9,045バイトになります！

```
$ gzip -9 < pkg/wasm_game_of_life_bg.wasm | wc -c
9045
```

## 演習
* [`wasm-snip`ツール](../reference/code-size.html#use-the-wasm-snip-tool)を使用して、ライフゲームの`.wasm`からパニック状態のインフラストラクチャ関数を削除せよ。 何バイト節約できますか？

* [グローバルアロケータとしての `wee_alloc`](https://github.com/rustwasm/wee_alloc)を使用して、または使用せずに、ライフゲームクレートを作成せよ。
  このプロジェクトを開始するために複製した`rustwasm/wasm-pack-template`テンプレートには、`wasm-game-of-life/Cargo.toml`の `[features]`セクションの `default`キーに追加することで有効にできる`wee_alloc`機能があります。

  ```toml
  [features]
  default = ["wee_alloc"]
  ```

  `wee_alloc`を使用すると、` .wasm`からどのくらいのサイズが削られますか？

* 単一の `Universe`をインスタンス化するだけなので、コンストラクターを提供する代わりに、単一の`staticmut`グローバルインスタンスを操作する操作をエクスポートできます。
  このグローバルインスタンスが前の章で説明したダブルバッファリング手法も使用している場合は、それらのバッファも「静的ミュート」グローバルにすることができます。
  これにより、ライフゲームの実装からすべての動的割り当てが削除され、アロケータを含まない `#![no_std]`クレートにすることができます。
  アロケータの依存関係を完全に削除することで、 `.wasm`からどのくらいのサイズが削除されましたか？