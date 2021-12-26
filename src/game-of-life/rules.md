# コンウェイのライフゲームのルール

*注：コンウェイのライフゲームとそのルールに既に精通している場合は、次のセクションに進んでください。*

[Wikipedia gives a great description of the rules of Conway's Game of
Life:][wikipedia]

> ライフゲームの宇宙は、正方形のセルの無限の2次元直交グリッドであり、
> それぞれが活性か非活性化か、「人口が多い」または「人口が少ない」という2つの可能な状態のいずれかにあります。
> すべてのセルは、水平、垂直、または斜めに隣接するセルである8つの隣接セルと相互作用します。
> 時間の各ステップで、次の遷移が発生します。:
> 1. 活性状態の隣人が2人未満の活性セルは、人口不足が原因であるかのように非活性になります。
>
> 2. 2つまたは3つの活性状態の隣人がいる活性セルは、次の世代に活性を続けます。
>
> 3. 人口過多のように、3つ以上の活性状態の隣人がいる活性セルは非活性になります。
>
> 4. ちょうど3つの活性状態の隣人を持つ非活性セルは、まるで生殖によるかのように活性セルになります。
>
> 初期パターンは、システムのシードを構成します。
> 第1世代は、シード内のすべてのセルに上記のルールを同時に適用することによって作成されます。
> 誕生と死は同時に発生し、これが発生する離散モーメントはダニと呼ばれることもあります（言い換えると、各世代は純粋関数です）。
> ルールは、次の世代を作成するために繰り返し適用され続けます。

[wikipedia]: https://en.wikipedia.org/wiki/Conway%27s_Game_of_Life

次の初期宇宙を考えてみましよう:

<img src='../images/game-of-life/initial-universe.png' alt='Initial Universe' width=80 />

各セルを考慮して次世代を計算できます。
左上のセルは非活性になります。
ルール(4)は、デッドセルに適用される唯一の遷移ルールです。
ただし、左上のセルには正確に3つの隣接セルがないため、遷移ルールは適用されず、次世代でも無効のままになります。
同じことが最初の行の他のすべてのセルにも当てはまります。

2行目の一番上の生細胞を考えると、物事は面白くなります。
3列目。 生細胞の場合、最初の3つのルールのいずれかが潜在的に適用されます。
このセルの場合、ライブネイバーは1つしかないため、ルール(1)が適用されます。
このセルは次世代で消滅します。
同じ運命が一番下の生細胞を待っています。

中央のライブセルには、上部と下部のライブセルの2つのライブセルがあります。
これは、ルール(2)が適用され、次世代でも存続することを意味します。

最後の興味深いケースは、真ん中の活性セルのすぐ左右にある非活性セルです。
3つの活性セルはすべてこれらのセルの両方に隣接しています。
つまり、ルール(4)が適用され、これらのセルは次世代で活性になります。

すべてをまとめると、次の世代で以下の宇宙が得られます。

<img src='../images/game-of-life/next-universe.png' alt='Next Universe' width=80 />

これらの単純で決定論的なルールから、奇妙で刺激的な行動が浮かび上がります。

| Gosper's glider gun | Pulsar | Space ship |
|---|---|---|
| ![Gosper's glider gun](https://upload.wikimedia.org/wikipedia/commons/e/e5/Gospers_glider_gun.gif) | ![Pulsar](https://upload.wikimedia.org/wikipedia/commons/0/07/Game_of_life_pulsar.gif) | ![Lighweight space ship](https://upload.wikimedia.org/wikipedia/commons/3/37/Game_of_life_animated_LWSS.gif) |

<center>
<iframe width="560" height="315" src="https://www.youtube.com/embed/C2vgICfQawE?rel=0&amp;start=65" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>
</center>

## 演習

* 例の宇宙の次の世代を手動で計算します。 おなじみの何かに気づきましたか？

  <details>
    <summary>Answer</summary>

    これは、サンプル宇宙の初期状態である必要があります。

    <img src='../images/game-of-life/initial-universe.png' alt='Initial Universe' width=80 />

    このパターンは*周期的*です。2ティックごとに初期状態に戻ります。

  </details>

* 安定している最初の宇宙を見つけることができますか？
  つまり、すべての世代が常に同じである宇宙です。

  <details>
    <summary>Answer</summary>

    安定した宇宙は無数にあります！
    自明に安定した宇宙は空の宇宙です。
    2x2の正方形の活性セルも安定した宇宙です。

  </details>
