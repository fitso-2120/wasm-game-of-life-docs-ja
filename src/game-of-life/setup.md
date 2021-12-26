# 設定

このセクションでは、RustプログラムをWebAssemblyにコンパイルしてJavaScriptに統合するためのツールチェーンを設定する方法について説明します。

## Rustツールチェーン

`rustup`、`rustc`、 `cargo` を含む標準のRustツールチェーンが必要になります。

[これらの手順に従って、Rustツールチェーンをインストールしてください。][rust-install]

RustとWebAssemblyのエクスペリエンスは、Rustリリーストレインに乗って安定します！
つまり、実験的な機能フラグは必要ありません。ただし、Rust1.30以降が必要です。

## `wasm-pack`

`wasm-pack`は、Rustで生成されたWebAssemblyを構築、テスト、公開するためのワンストップショップです。

[ここで `wasm-pack`を入手してください！][wasm-pack-install]

## `cargo-generate`

[`cargo-generate`は、新しいRustプロジェクトをすばやく立ち上げて実行するのに役立ちます
既存のgitリポジトリをテンプレートとして活用します。][cargo-generate]

次のコマンドで `cargo-generate`をインストールします。

```
cargo install cargo-generate
```

## `npm`

`npm`はJavaScriptのパッケージマネージャーです。これを使用して、
JavaScriptバンドラーと開発サーバー。チュートリアルの最後に、
コンパイルした `.wasm`を` npm`レジストリに公開します。

[これらの手順に従って `npm`をインストールしてください。][npm-install]

すでに `npm`がインストールされている場合は、次のコマンドで最新であることを確認してください。

```
npm install npm@latest -g
```

[rust-install]:https://www.rust-lang.org/tools/install
[npm-install]:https://www.npmjs.com/get-npm
[wasm-pack]:https://github.com/rustwasm/wasm-pack
[cargo-generate]:https://github.com/ashleygwilliams/cargo-generate
[wasm-pack-install]:https://rustwasm.github.io/wasm-pack/installer/