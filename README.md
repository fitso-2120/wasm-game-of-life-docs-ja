# wasm-game-of-life-docs-ja
Rustのwasm用チュートリアルの和訳です。<br/>
<br/>
google翻訳にお世話になりました。最後に章を追加して、機械翻訳を利用した際の注意点をまとめてみました。
<br/>
<div align="center">

  <h1>The Rust and WebAssembly Book</h1>

  <strong>この小さな本では、RustとWebAssemblyを一緒に使用する方法について説明しています。 また、クールな演習を含むチュートリアルで構成されています。</strong>

  <h3>
    <a href="https://rustwasm.github.io/docs/book/">Read the Book</a>
    <span> | </span>
    <a href="https://github.com/rustwasm/book/blob/master/CONTRIBUTING.md">Contributing</a>
    <span> | </span>
    <a href="https://discordapp.com/channels/442252698964721669/443151097398296587">Chat</a>
  </h3>

  <sub>Built with 🦀🕸 by <a href="https://rustwasm.github.io/">The Rust and WebAssembly Working Group</a></sub>
</div>

## About

このリポジトリには、Rust for wasmの使用、一般的なワークフロー、開始方法など、深く掘り下げていくためのドキュメントが含まれています。 それは Rust でいくつかの本当にきちんとしたことをするためのガイドとして機能します。 

RustとWebAssemblyを一緒に使用する方法を学び始めたい場合は、[オンライン][book]で読むことができます。

[Open issues for improving the Rust and WebAssembly book.][book-issues]

[book-issues]: https://github.com/rustwasm/book/issues

## Building the Book

この本は[`mdbook`][mdbook]を使って作られています。 インストールするには、 `cargo`をインストールする必要があります。 Rustツールをインストールしていない場合は、最初に[`rustup`][rustup]をインストールする必要があります。 セットアップを取得するには、サイトの指示に従ってください。

Rust環境のインストールが終わったら、次のようにします。

```bash
$ cargo install mdbook
```

バイナリを実行できるように、 `cargo install`ディレクトリが`$PATH`にあることを確認してください。

次に、このディレクトリから次のコマンドを実行します。

```bash
$ mdbook build
```

これにより、ブックと出力ファイルが「book」というディレクトリに作成されます。 そこから、`index.html`ファイルに移動してブラウザで表示できます。 加えている可能性のある変更を確認する場合は、次のコマンドを実行して変更を自動的に生成することもできます。

```bash
$ mdbook serve
```

これにより、変更を加えるとファイルが自動的に生成され、ローカルで提供されるため、毎回 `build`を呼び出さなくてもファイルを簡単に表示できます。

ファイルはすべて Markdown で書かれているので、それらを読むために本を生成したくない場合は、 `src`ディレクトリからそれらを読むことができます。

[mdbook]: https://github.com/rust-lang-nursery/mdBook
[rustup]: https://github.com/rust-lang-nursery/rustup.rs/
[book]: https://rustwasm.github.io/book/game-of-life/introduction.html


