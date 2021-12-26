# チュートリアル：コンウェイのライフゲーム

RustとWebAssemblyに[Conway's Game of Life][gol]を実装するチュートリアルです。

[gol]:https://en.wikipedia.org/wiki/Conway%27s_Game_of_Life

## このチュートリアルは誰のためのものですか？

このチュートリアルは、RustとJavaScriptの基本的な経験があり、Rust、WebAssembly、JavaScriptを一緒に使用する方法を学びたい人を対象としています。

基本的なRust、JavaScript、およびHTMLの読み書きに慣れている必要があります。あなたは間違いなく専門家である必要はありません。

## 何を学びますか？

* WebAssemblyにコンパイルするためのRustツールチェーンを設定する方法。

* Rust、WebAssembly、JavaScript、HTML、およびCSSから作成された複数言語によるプログラム群を開発するためのワークフロー。

* RustとWebAssemblyの両方の長所とJavaScriptの長所を最大限に活用するようにAPIを設計する方法。

* RustからコンパイルされたWebAssemblyモジュールをデバッグする方法。

* RustおよびWebAssemblyプログラムの時間をプロファイリングして高速化する方法。

* プロファイルRustおよびWebAssemblyプログラムのサイズを変更して、 `.wasm`バイナリをより小さく高速にしてネットワーク経由でダウンロードする方法。