# What is WebAssembly?

WebAssembly(wasm)は、[広範な仕様]を備えた単純なマシンモデルと実行可能形式です。
ポータブルでコンパクトに設計されており、ネイティブ速度またはそれに近い速度で実行されます。

プログラミング言語として、WebAssemblyは、方法は異なりますが、同じ構造を表す2つの形式で構成されています。

1. `.wat`テキスト形式("**W**eb**A**ssembly**T**ext"の`wat`と呼ばれる)は[S式]を使用し、LispファミリーのLispファミリー (SchemeやClojureのような言語) にいくらか似ています。

2. `.wasm`バイナリ形式は低レベルであり、wasm仮想マシンで直接使用することを目的としています。 概念的にはELFやMach-Oに似ています。

`wat`の階乗関数を次に示します。

```
(module
  (func $fac (param f64) (result f64)
    local.get 0
    f64.const 1
    f64.lt
    if (result f64)
      f64.const 1
    else
      local.get 0
      local.get 0
      f64.const 1
      f64.sub
      call $fac
      f64.mul
    end)
  (export "fac" (func $fac)))
```

`wasm`ファイルがどのように見えるか知りたい場合は、上記のコードで[wat2wasmデモ]を使用できます。

## Linear Memory

WebAssemblyには非常に単純な[メモリモデル]があります。
wasmモジュールは、基本的にバイトのフラット配列である単一の「線形メモリ」にアクセスできます。
この[メモリは拡張可能]で、ページサイズの倍数（64K）になります。 縮小することはできません。

## Is WebAssembly Just for the Web?

現在、JavaScriptおよびWebコミュニティ全般で注目を集めていますが、wasmはホスト環境について何も想定していません。
したがって、wasmが将来さまざまなコンテキストで使用される「ポータブル実行可能」形式になると推測するのは理にかなっています。
ただし、*今日*の時点で、wasmは主にJavaScript(JS)に関連しており、JavaScript(JS)にはさまざまな種類があります(Webと[Node.js]の両方を含む)。

[メモリモデル]: https://webassembly.github.io/spec/core/syntax/modules.html#syntax-mem
[メモリは拡張可能]: https://webassembly.github.io/spec/core/syntax/instructions.html#syntax-instr-memory
[広範な仕様]: https://webassembly.github.io/spec/
[value types]: https://webassembly.github.io/spec/core/syntax/types.html#value-types
[Node.js]: https://nodejs.org
[S式]: https://en.wikipedia.org/wiki/S-expression
[wat2wasmデモ]: https://webassembly.github.io/wabt/demo/wat2wasm/
