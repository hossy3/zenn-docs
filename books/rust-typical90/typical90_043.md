---
title: "043 - Maze Challenge with Lack of Sleep（★4）"
---

[043 \- Maze Challenge with Lack of Sleep（★4）](https://atcoder.jp/contests/typical90/tasks/typical90_aq)


# アルゴリズム

図示が難しいです。後日描き直すつもりです。


## 位置と向きを使ってグラフを組み立てる

### 頂点

人🚶がゴール🚩にたどり着くために曲がる回数を最小化したい、という問題です。

|||||||
|---|---|---|---|---|---|
|.|.|.|#|🚩|.|
|🚶|#|.|#|#|.|
|.|#|.|.|.|.|
|.|.|.|#|#|.|

|||||||
|---|---|---|---|---|---|
|➡️1|➡️|⬇️2|🧱|🚩|⬅️5|
|⬆️🚶|🧱|⬇️|🧱|🧱|⬆️|
|.|🧱|➡️3|➡️|➡️|⬆️4|
|.|.|.|🧱|🧱|.|

よくある迷路の探索では、座標 (R, C) に対する歩数を 2次元の配列で管理して解きます。この問題では「同じ向きに動き続けても追加コストがかからない」ことを表せるように、「向き」についても扱います。 3次元配列です。

たとえば、スタートとその 1つ上のマスの曲がる回数を考えます。スタートは曲がらなくても着けます。0回です。

1つ上のマスは、上向きでスタートした場合には、1回も曲がらずに辿りつけます。このときは上向きです。ここから 1回曲がると左右を向くことができます。次のようになります。

* スタート:
  * `map[1][0][Up] = 0`
  * `map[1][0][Left] = 0`
  * `map[1][0][Right] = 0`
  * `map[1][0][Down] = 0`
* スタートの 1つ上 = 左上隅
  * `map[0][0][Up] = 0`
  * `map[0][0][Left] = 1`
  * `map[0][0][Right] = 1`
  * `map[0][0][Down] = 1` or `2` ※

曲がる回数を最小にするときに U ターンは必要ないはずです。毎回直角に曲がるだけとして、2回曲がることにしても良いです。 U ターンを1回曲がることにしても答えには影響しません。どちらで解いても良いです。

### 辺

人の向いている方向ごとにグラフを組み立てます。矢印の方向にはコスト 0 で移動できます。有向辺のコストを 0 にします。

曲がることは、同じ位置の別方向に移るにはコスト 1、と考えます。

|||||||
|---|---|---|---|---|---|
|⬆️|⬆️|⬆️|🧱|⬆️|⬆️|
|⬆️|🧱|⬆️|🧱|🧱|⬆️|
|⬆️|🧱|⬆️|⬆️|⬆️|⬆️|
|⬆️|⬆️|⬆️|🧱|🧱|⬆️|

|||||||
|---|---|---|---|---|---|
|⬅️|⬅️|⬅️|🧱|⬅️|⬅️|
|⬅️|🧱|⬅️|🧱|🧱|⬅️|
|⬅️|🧱|⬅️|⬅️|⬅️|⬅️|
|⬅️|⬅️|⬅️|🧱|🧱|⬅️|

|||||||
|---|---|---|---|---|---|
|➡️|➡️|➡️|🧱|➡️|➡️|
|➡️|🧱|➡️|🧱|🧱|➡️|
|➡️|🧱|➡️|➡️|➡️|➡️|
|➡️|➡️|➡️|🧱|🧱|➡️|

|||||||
|---|---|---|---|---|---|
|⬇️|⬇️|⬇️|🧱|⬇️|⬇️|
|⬇️|🧱|⬇️|🧱|🧱|⬇️|
|⬇️|🧱|⬇️|⬇️|⬇️|⬇️|
|⬇️|⬇️|⬇️|🧱|🧱|⬇️|

## グラフで最小ステップ数を調べる

### 初期状態

初期状態をこのように設定します。数字がないところは +∞ などを入れておきます。

|||||||
|---|---|---|---|---|---|
|⬆️|⬆️|⬆️|🧱|⬆️|⬆️|
|⬆️🚶0|🧱|⬆️|🧱|🧱|⬆️|
|⬆️|🧱|⬆️|⬆️|⬆️|⬆️|
|⬆️|⬆️|⬆️|🧱|🧱|⬆️|

|||||||
|---|---|---|---|---|---|
|⬅️|⬅️|⬅️|🧱|⬅️|⬅️|
|⬅️🚶0|🧱|⬅️|🧱|🧱|⬅️|
|⬅️|🧱|⬅️|⬅️|⬅️|⬅️|
|⬅️|⬅️|⬅️|🧱|🧱|⬅️|

|||||||
|---|---|---|---|---|---|
|➡️|➡️|➡️|🧱|➡️|➡️|
|➡️🚶0|🧱|➡️|🧱|🧱|➡️|
|➡️|🧱|➡️|➡️|➡️|➡️|
|➡️|➡️|➡️|🧱|🧱|➡️|

|||||||
|---|---|---|---|---|---|
|⬇️|⬇️|⬇️|🧱|⬇️|⬇️|
|⬇️🚶0|🧱|⬇️|🧱|🧱|⬇️|
|⬇️|🧱|⬇️|⬇️|⬇️|⬇️|
|⬇️|⬇️|⬇️|🧱|🧱|⬇️|


### 終了状態

U ターンありの場合は、次のように各マスでの人の向きと、それまでに曲がった回数が求まります。

|||||||
|---|---|---|---|---|---|
|⬆️0|⬆️2|⬆️2|🧱|⬆️🚩6|⬆️4|
|⬆️0|🧱|⬆️2|🧱|🧱|⬆️4|
|⬆️1|🧱|⬆️2|⬆️4|⬆️4|⬆️4|
|⬆️1|⬆️2|⬆️2|🧱|🧱|⬆️5|

|||||||
|---|---|---|---|---|---|
|⬇️1|⬇️2|⬇️2|🧱|⬇️🚩6|⬇️5|
|⬇️0|🧱|⬇️2|🧱|🧱|⬇️5|
|⬇️0|🧱|⬇️2|⬇️4|⬇️4|⬇️4|
|⬇️0|⬇️2|⬇️2|🧱|🧱|⬇️4|

|||||||
|---|---|---|---|---|---|
|⬅️1|⬅️2|⬅️2|🧱|⬅️🚩5|⬅️5|
|⬅️0|🧱|⬅️3|🧱|🧱|⬅️5|
|⬅️1|🧱|⬅️3|⬅️4|⬅️4|⬅️4|
|⬅️1|⬅️2|⬅️2|🧱|🧱|⬅️5|

|||||||
|---|---|---|---|---|---|
|➡️1|➡️1|➡️1|🧱|➡️🚩6|➡️5|
|➡️0|🧱|➡️3|🧱|🧱|➡️5|
|➡️1|🧱|➡️3|➡️3|➡️3|➡️3|
|➡️1|➡️1|➡️1|🧱|🧱|➡️5|

🚩の最小値 5 が答えになります。

## pathfinding の dijkstra で解く (TLE)

グラフの組み立てができれば、解くのはライブラリにおまかせできそうです。いつもの pathfinding を使います。

```rust
let (_, result) = reachables.get(&t.unwrap()).unwrap();
println!("{result}");
```

しかし、この方針は残念ながらうまくいきません。

マスの数が 1000*1000 あます。それぞれ 4方向あると、状態数は 400万個になります。 pathfinding の状態管理に使っている HashMap は Rust では定数倍が重いです。 400万要素あるとタイムアウト TLE してしまいます。


## 3次元配列の dijkstra 01-BFS で解く

3次元配列をで経路探索します。コスト(曲がる回数)は 0回か 1回ですから、 両端キュー `VecDeque` を使った 01-BFS が有効です。次のように書けます。

```rust
let mut a = vec![vec![[usize::MAX, usize::MAX, usize::MAX, usize::MAX]; w]; h]; // UDLR
a[rs][cs] = [0, 0, 0, 0];
let dirs = [(-1, 0), (1, 0), (0, -1), (0, 1)];

let mut queue = VecDeque::<((usize, usize, usize), usize)>::new(); // ((r, c, dir), cost)
for dir in 0..4 {
    queue.push_back(((rs, cs, dir), 0));
}

while let Some(((r, c, dir), cost)) = queue.pop_front() {
    let (dr, dc) = dirs[dir];
    let r0 = r.wrapping_add_signed(dr);
    let c0 = c.wrapping_add_signed(dc);
    if r0 < h && c0 < w && s[r0][c0] != '#' {
        if a[r0][c0][dir] > cost {
            a[r0][c0][dir] = cost;
            queue.push_front(((r0, c0, dir), cost));
        }
    }

    let cost0 = cost + 1;
    for (dir0, _) in dirs.iter().enumerate() {
        if a[r][c][dir0] > cost0 {
            a[r][c][dir0] = cost0;
            queue.push_back(((r, c, dir0), cost0))
        }
    }
}
```

`queue` の変化です。このように各マスの `((r, c, dir), cost)` を更新していき、最後にはすべてのマスの cost = 曲がる回数が埋まります。

|queue|先頭取り出し|先頭に追加|末尾に追加|
|---|---|---|---|
|[((1, 0, 0↑), 0),<br />((1, 0, 1↓), 0),<br />((1, 0, 2←), 0),<br />((1, 0, 3→), 0)]|(1, 0, 0↑), 0)|[(0, 0, 0↑), 0)]|[]|
|[((0, 0, 0↑), 0),<br />((1, 0, 1↓), 0),<br />((1, 0, 2←), 0),<br />((1, 0, 3→), 0)]|((0, 0, 0↑), 0)|[]|[((0, 0, 1↓), 1),<br />((0, 0, 2←), 1),<br />((0, 0, 3→), 1)]|
|[((1, 0, 1↓), 0),<br />((1, 0, 2←), 0),<br />((1, 0, 3→), 0),<br />((0, 0, 1↓), 1),<br />((0, 0, 2←), 1),<br />((0, 0, 3→), 1)]|((1, 0, 1↓), 0)|[((2, 0, 1↓), 0)]|[]|
|[((2, 0, 1↓), 0),<br />((1, 0, 2←), 0),<br />((1, 0, 3→), 0),<br />((0, 0, 1↓), 1),<br />((0, 0, 2←), 1),<br />((0, 0, 3→), 1)]|((2, 0, 1↓), 0)|[((3, 0, 1↓), 0)]|[((2, 0, 0↑), 1),<br />((2, 0, 2←), 1),<br />((2, 0, 3→), 1)]|


この方針なら制限時間の 1/10 以内で余裕をもって解けます。

## 3次元配列の dijkstra BFS で解く

コストが 0, 1 に限らない、一般の幅優先探索 dijkstra を使っても解けます。 `VecDeque` の代わりに、優先度付きキュー `BinaryHeap` を使います。曲がる回数が少ないものを優先度が高いものとして `pop()` で取り出せるように、コストを反転させた値をタプルの第一要素に入れます。

```diff rust
 let mut a = vec![vec![[usize::MAX, usize::MAX, usize::MAX, usize::MAX]; w]; h]; // UDLR
 a[rs][cs] = [0, 0, 0, 0];
 let dirs = [(-1, 0), (1, 0), (0, -1), (0, 1)];
-let mut queue = VecDeque::<((usize, usize, usize), usize)>::new(); // ((r, c, dir), cost)
+let mut queue = BinaryHeap::<(Reverse<usize>, (usize, usize, usize))>::new(); // (cost, (r, c, dir))
 for dir in 0..4 {
-    queue.push_back(((rs, cs, dir), 0));
+    queue.push((Reverse(0), (rs, cs, dir)));
 }
-while let Some(((r, c, dir), cost)) = queue.pop_front() {
+while let Some((Reverse(cost), (r, c, dir))) = queue.pop() {
     let (dr, dc) = dirs[dir];
     let r0 = r.wrapping_add_signed(dr);
     let c0 = c.wrapping_add_signed(dc);
     if r0 < h && c0 < w && s[r0][c0] != '#' {
         if a[r0][c0][dir] > cost {
             a[r0][c0][dir] = cost;
-            queue.push_front(((r0, c0, dir), cost));
+            queue.push((Reverse(cost), (r0, c0, dir)));
         }
     }
     let cost0 = cost + 1;
     for (dir0, _) in dirs.iter().enumerate() {
         if a[r][c][dir0] > cost0 {
             a[r][c][dir0] = cost0;
-            queue.push_back(((r, c, dir0), cost0))
+            queue.push((Reverse(cost0), (r, c, dir0)))
         }
     }
 }
```

他の言語を使っている人から見ると、コストの反転に  `Reverse(cost)` するのが面倒そう、単に `-cost` で十分では、とも思います。けれどこのように `usize` と `Reverse<usize>` の型を区別して使うことで、正のコストと負のコストを混同しにくくなります。ポカミスが入りにくくなります。

一般のコストを扱える構造だと、01-BFS より重いです。それでも TLE 制限時間の半分程度で解けます。


# Tips

## 上下左右を辿る

[012 - Red Painting（★4）](./typical90_012) で紹介しました。


## 2次元配列の位置の示し方: rc か ij か xy か

この問題は 3次元配列ですが、 2次元配列について考えます。このマスの位置をプログラミングコードでどう示すか、おそらくいろいろな派があると思います。

* `(r, c)`, `a[r][c]` 派 (行 row, 列 column)
  * pros: 変数名で行列どちらを指すかが分かりやすい
  * cons: `dr`, `dc` はあまり言わない
* `(i, j)`, `a[i][j]` 派 (i, j, k, ...)
  * pros: ループを回す順番が分かりやすい
  * cons: `i`, `j` ともに方向を表さない名前
* `(x, y)`, `a[y][x]` 派 (x, y, z, ...)
  * pros: 幾何的、`dx`, `dy` を数学で良く見る
  * cons: `a[y][x]` インデックスが逆順になる

混ぜると危険です。この方法・この変数名でいつも解く、と決めておくのをおすすめします。


# 実装例

## pathfinding (TLE)
https://github.com/hossy3/atcoder-solutions/blob/main/atcoder/typical90/src/bin/043_pathfinding_tle.rs

他にも高速化を頑張ってみましたが、私には TLE の壁を越えられませんでした。

## 3次元配列の dijkstra 01-BFS
https://github.com/hossy3/atcoder-solutions/blob/main/atcoder/typical90/src/bin/043_2d_dijkstra_01bfs.rs

## 3次元配列の dijkstra BFS
https://github.com/hossy3/atcoder-solutions/blob/main/atcoder/typical90/src/bin/043_2d_dijkstra_2.rs
