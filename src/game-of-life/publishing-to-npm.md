# npmへ公開する

動作し、高速で、*そして*小さな `wasm-game-of-life`パッケージができたので、npmに公開して、他のJavaScript開発者が既製のライフゲーム実装を必要とする場合に再利用できるようにします。

## 前提条件

まず、[npmアカウントを持っていることを確認してください](https://www.npmjs.com/signup)。

次に、次のコマンドを実行して、ローカルでアカウントにログインしていることを確認します。

```
wasm-pack login
```

## 公開する

`wasm-game-of-life`ディレクトリ内で`wasm-pack`を実行して、 `wasm-game-of-life/pkg`ビルドが最新であることを確認します。

```
wasm-pack build
```

今すぐ `wasm-game-of-life/pkg`の内容を確認してください。これは、次のステップでnpmに公開するものです。

準備ができたら、 `wasm-packpublish`を実行してパッケージをnpmにアップロードします。

```
wasm-pack publish
```

npmに公開するのに必要なことはこれだけです！

...他の人もこのチュートリアルを行っていることを除いて。
したがって、 `wasm-game-of-life`の名前はnpmで使用され、最後のコマンドはおそらく機能しませんでした。

`wasm-game-of-life/Cargo.toml`を開き、`name`の末尾にユーザー名を追加して、独自の方法でパッケージを明確にします。

```toml
[package]
name = "wasm-game-of-life-my-username"
```

次に、再構築して再度公開します。

```
wasm-pack build
wasm-pack publish
```

今回はうまくいくはずです！
