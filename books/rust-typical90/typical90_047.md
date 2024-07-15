---
title: "047 - Monochromatic Diagonal（★7）"
---

[047 \- Monochromatic Diagonal（★7）](https://atcoder.jp/contests/typical90/tasks/typical90_au)


# アルゴリズム

## 例題1 をそのまま解く

色の変更ルールは次の表のようになっています。

|縦Sの色|横Tの色||合成した色|
|---|---|---|---|
|R🔴|R🔴||R🔴|
|G🟢|B🔵||R🔴|
|B🔵|G🟢||R🔴|
|G🟢|G🟢||G🟢|
|B🔵|R🔴||G🟢|
|R🔴|B🔵||G🟢|
|B🔵|B🔵||B🔵|
|R🔴|G🟢||B🔵|
|G🟢|R🔴||B🔵|

このルールの通り、文字列長×文字列長の 2次元配列を埋めます。

|S＼T||G🟢|R🔴|G🟢|R🔴|B🔵||
|---|---|---|---|---|---|---|---|
|R🔴||B🔵|R🔴|B🔵|R🔴|G🟢|
|G🟢||G🟢|B🔵|G🟢|B🔵|R🔴|↖️🟢|
|B🔵||R🔴|G🟢|R🔴|G🟢|B🔵|↖️🔴|
|G🟢||G🟢|B🔵|G🟢|B🔵|R🔴|↖️🔵|
|B🔵||R🔴|G🟢|R🔴|G🟢|B🔵|
||||↖️🔴|↖️🟢||↖️🟢

「↖️」のある斜めの列が、すべて同じ色になっているものとなります。この列の数を数える、という問題です。

2次元配列を埋めれば解けますが、計算量が $O(N^2)$ となります。時間切れしそうです。

## 部分文字列の一致問題に持ち込む

R🔴 になるパターンは以下の3通りでした。

|縦Sの色|横Tの色||合成した色|
|---|---|---|---|
|R🔴|R🔴||R🔴|
|G🟢|B🔵||R🔴|
|B🔵|G🟢||R🔴|

横Tの色を、事前に B🔵, G🟢 を入れ替えたものを T' とします。そうすると次のようになります。

|縦Sの色|横T'の色||合成した色|
|---|---|---|---|
|R🔴|R🔴||R🔴|
|G🟢|G🟢||R🔴|
|B🔵|B🔵||R🔴|


|縦＼横|│|R🔴|G🟢|B🔵|
|---|---|---|---|---|
|R🔴|│|R🔴|B🔵|G🟢|
|G🟢|│|B🔵|G🟢|R🔴|
|B🔵|│|G🟢|R🔴|B🔵|

こうすると、「S と T' が同じ文字列になるときには、対応する S と T を組み合わせるとすべて  R🔴 になる」と、問題を言い換えられます。

例題1 で合成した結果が赤くなるところを探すところを、T' で置き換えてみます。赤くならないところは 🚧 にします。

|S＼T||G🟢|R🔴|G🟢|R🔴|B🔵||
|---|---|---|---|---|---|---|---|
|R🔴||🚧|R🔴|🚧|R🔴|🚧|
|G🟢||🚧|🚧|🚧|🚧|R🔴|
|B🔵||R🔴|🚧|R🔴|🚧|🚧|↖️🔴|
|G🟢||🚧|🚧|🚧|🚧|R🔴|
|B🔵||R🔴|🚧|R🔴|🚧|🚧|
||||↖️🔴|

部分文字列の一致問題に置き換えられます。

|S＼T'||B🔵|R🔴|B🔵|R🔴|G🟢||
|---|---|---|---|---|---|---|---|
|R🔴||🚧|R🔴|🚧|R🔴|🚧|
|G🟢||🚧|🚧|🚧|🚧|G🟢|
|B🔵||B🔵|🚧|B🔵|🚧|🚧|↖️✅|
|G🟢||🚧|🚧|🚧|🚧|G🟢|
|B🔵||B🔵|🚧|B🔵|🚧|🚧|
||||↖️✅|

このようにして、斜め列を探す問題を、 T の 2色を置き換える形で扱うことができました。これを繰り返し、3つの問題を解くことで、すべての答えが求まります。

* R🔴 の斜め列を探す問題: T の G🟢, B🔵 を置き換えて、部分文字列の一致を調べる
* G🟢 の斜め列を探す問題: T の B🔵, R🔴 を置き換えて、部分文字列の一致を調べる
* B🔵 の斜め列を探す問題: T の R🔴, G🟢 を置き換えて、部分文字列の一致を調べる

R🔴 = 0, B🔵 = 1, G🟢 = 2 として、部分文字列の一致を調べる処理を `f()` とすると、`f()` に渡す引数を次のように加工する感じです。

```rust
let result = f(&s, &t.iter().map(|&i| [0, 2, 1][i]).collect_vec())
    + f(&s, &t.iter().map(|&i| [1, 0, 2][i]).collect_vec())
    + f(&s, &t.iter().map(|&i| [2, 1, 0][i]).collect_vec());
println!("{result}");
```

部分文字列の一致 `f()` が $O(N^2)$ よりも短く求められれば、TLE せずに AC できます。


## Z-algorithm で解く

* [ac\-library/document\_ja/string\.md at master · atcoder/ac\-library · GitHub](https://github.com/atcoder/ac-library/blob/master/document_ja/string.md)

> 入力の長さを $n$ として、長さ $n$ の配列を返す。 $i$
 番目の要素は `s[0..n)`と`s[i..n)`のLCP(Longest Common Prefix)の長さ。
>
> $O(n)$

こちらをそのまま使えます。

部分文字列一致のうち、まず右上の斜め 5列を調べます。

|S＼T'|│|B🔵|R🔴|B🔵|R🔴|G🟢||
|---|---|---|---|---|---|---|---|
|R🔴|│|🚧|R🔴|🚧|R🔴|🚧|
|G🟢|│||🚧|🚧|🚧|G🟢|↖️9
|B🔵|│|||R🔴|🚧|🚧|↖️8
|G🟢|│||||🚧|G🟢|↖️7
|B🔵|│|||||🚧|↖️6
||||||||↖️5

S と T' をつなげた文字列 「R🔴 G🟢 B🔵 G🟢 B🔵 B🔵 R🔴 B🔵 R🔴 G🟢」に、Z-algorithm を適用します。

|i|R🔴|G🟢|B🔵|G🟢|B🔵|B🔵|R🔴|B🔵|R🔴|G🟢|LCPの長さ|斜めの長さ|一致？|
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
|0|R🔴|G🟢|B🔵|G🟢|B🔵|B🔵|R🔴|B🔵|R🔴|G🟢|10|-||
|1||R🔴<br />🚧|G🟢|B🔵|G🟢|B🔵|B🔵|R🔴|B🔵|R🔴|0|-||
|2|||R🔴<br />🚧|G🟢|B🔵|G🟢|B🔵|B🔵|R🔴|B🔵|0|-||
|3||||R🔴<br />🚧|G🟢|B🔵|G🟢|B🔵|B🔵|R🔴|0|-||
|4|||||R🔴<br />🚧|G🟢|B🔵|G🟢|B🔵|B🔵|0|-||
|5||||||R🔴<br />🚧|G🟢|B🔵|G🟢|B🔵|0|5||
|6|||||||R🔴|G🟢<br />🚧|B🔵|G🟢|1|4||
|7||||||||R🔴<br />🚧|G🟢|B🔵|0|3||
|8|||||||||R🔴|G🟢|2|2|✅|
|9||||||||||R🔴<br />🚧|0|1||

Z-algorithm では S 同士の部分文字列一致も含まれます。この問題では 半分スライドしたところから、S と T' の部分文字列一致問題になります。文字列の最後まで一致している、🚧 が現れないものが、すべて R🔴 になる文字列に対応します。これで、表の対角線から右上側をすべて求められます。

|S＼T'|│|B🔵|R🔴|B🔵|R🔴|G🟢||
|---|---|---|---|---|---|---|---|
|R🔴|│|||||
|G🟢|│|🚧|||||
|B🔵|│|B🔵|🚧||||
|G🟢|│|🚧|🚧|🚧|||
|B🔵|│|B🔵|🚧|R🔴|🚧||
||||↖️9|↖️8|↖️7|↖️6|

左下側も同様です。 T' と S をつなげた文字列 「B🔵 R🔴 B🔵 R🔴 G🟢 R🔴 G🟢 B🔵 G🟢 B🔵」に、Z-algorithm を適用します。

|i|B🔵|R🔴|B🔵|R🔴|G🟢|R🔴|G🟢|B🔵|G🟢|B🔵|LCPの長さ|斜めの長さ|一致？|
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
|6|||||||B🔵<br />🚧|R🔴|B🔵|R🔴|0|4||
|7||||||||B🔵|R🔴<br />🚧|B🔵|1|3||
|8|||||||||B🔵<br />🚧|R🔴|0|2||
|9||||||||||B🔵|1|1|✅|

ソースコードは次のようになります。

```rust
fn f_by_z_algorithm(s: &[usize], t: &[usize]) -> usize {
    let n = s.len();

    let v: Vec<&usize> = s.iter().chain(t.iter()).collect();
    let z = z_algorithm_arbitrary(&v);
    let mut result = (0..n).filter(|&i| z[n + i] >= n - i).count();

    let v: Vec<&usize> = t.iter().chain(s.iter()).collect();
    let z = z_algorithm_arbitrary(&v);
    result += (1..n).filter(|&i| z[n + i] >= n - i).count();

    result
}
```

これで解けました。

## Rolling-hash で解く

文字列 S, T' を R=0, G=1, B=2 の 3進数で考えると、次のイメージになります。

|i|S|3進数|10進数|
|---|---|---|---|
|0|||0|
|1|R🔴|0|0|
|2|R🔴 G🟢|01|1|
|3|R🔴 G🟢 B🔵|012|5|
|4|R🔴 G🟢 B🔵 G🟢|0121|16|
|5|R🔴 G🟢 B🔵 G🟢 B🔵|01212|50|

|i|T'|3進数|10進数|
|---|---|---|---|
|0|||0|
|1|B🔵|2|2|
|2|B🔵 R🔴|20|6|
|3|B🔵 R🔴 B🔵|202|20|
|4|B🔵 R🔴 B🔵 R🔴|2020|60|
|5|B🔵 R🔴 B🔵 R🔴 G🟢|20201|181|

この数を使い、部分文字列一致を調べます。また右上を考えます。

|S＼T'|│|B🔵|R🔴|B🔵|R🔴|G🟢||
|---|---|---|---|---|---|---|---|
|R🔴|│|🚧|R🔴|🚧|R🔴|🚧|
|G🟢|│||🚧|🚧|🚧|G🟢|↖️4
|B🔵|│|||R🔴|🚧|🚧|↖️3
|G🟢|│||||🚧|G🟢|↖️2
|B🔵|│|||||🚧|↖️1
||||||||↖️0

||S 部分文字列|T' 部分文字列|S 値|T' 値|一致？|
|---|---|---|---|---|---|
|0|R🔴 G🟢 B🔵 G🟢 B🔵|B🔵 R🔴 B🔵 R🔴 G🟢|50|181||
|1|R🔴 G🟢 B🔵 G🟢|R🔴 B🔵 R🔴 G🟢|16|181 - 2 * 3^4 = 19||
|2|R🔴 G🟢 B🔵|B🔵 R🔴 G🟢|5|181 - 6 * 3^3 = 19||
|3|R🔴 G🟢|R🔴 G🟢|1|181 - 20 * 3^2 = 1|✅|
|4|R🔴|G🟢|0|181 - 60 * 3 = 1||

S の右側を 3文字減らした 「R🔴 G🟢」と、T' の右側を 2文字減らした「R🔴 G🟢」が一致しているかを考えます。

S 「R🔴 G🟢: 1」は `S[2]` として計算済みです。

T' 「R🔴 G🟢」は、「B🔵 R🔴 B🔵 R🔴 G🟢: 181」から左端の「B🔵 R🔴 B🔵: 20」 を除いたものと考えます。同じ5桁になるように「B🔵 R🔴 B🔵 ❔ ❔」と右に 2桁増やすと、この値は $20 * 3^2 = 180$ になります。引き算すると 181 - 180 = 1 です。

このようにして、「R🔴 G🟢」と「R🔴 G🟢」が同じ値と分かります。

左下も同様です。

|S＼T'|│|B🔵|R🔴|B🔵|R🔴|G🟢||
|---|---|---|---|---|---|---|---|
|R🔴|│|||||
|G🟢|│|🚧|||||
|B🔵|│|B🔵|🚧||||
|G🟢|│|🚧|🚧|🚧|||
|B🔵|│|B🔵|🚧|R🔴|🚧||
||||↖️3|↖️2|↖️1|↖️0|

||T' 部分文字列|S 部分文字列|T' 値|S 値|一致？|
|---|---|---|---|---|---|
|1|B🔵 R🔴 B🔵 R🔴|G🟢 B🔵 G🟢 B🔵|60|50 - 0 * 3^4 = 50||
|2|B🔵 R🔴 B🔵|B🔵 G🟢 B🔵|20|50 - 1 * 3^3 = 23||
|3|B🔵 R🔴|G🟢 B🔵|6|50 - 5 * 3^2 = 5||
|4|B🔵|B🔵|2|50 - 16 * 3 = 2|✅|

コード例です。

```rust
fn f_by_rolling_hash(s: &[usize], t: &[usize]) -> usize {
    let n = s.len();

    let mut k = vec![Mint::new(0); n + 1];
    k[0] = Mint::new(1);
    for i in 0..n {
        k[i + 1] = k[i] * 3;
    }

    let mut s0 = vec![Mint::new(0); n + 1];
    let mut t0 = vec![Mint::new(0); n + 1];
    for (i, &x) in s.iter().enumerate() {
        s0[i + 1] = s0[i] * 3 + x;
    }
    for (i, &x) in t.iter().enumerate() {
        t0[i + 1] = t0[i] * 3 + x;
    }

    let mut result = 0usize;
    for i in 0..n {
        if s0[n - i] == t0[n] - t0[i] * k[n - i] {
            result += 1;
        }
    }
    for i in 1..n {
        if t0[n - i] == s0[n] - s0[i] * k[n - i] {
            result += 1;
        }
    }

    result
}
```

Rolling-hash の難点は、このように n乗の計算をすると、簡単に 32bit などの整数型の上限を超えることです。多倍長整数は遅いです。そこでたとえば素数 998244353 との余りを取り、低確率な衝突が発生する hash として扱うようにしています。

低確率な衝突が起きると WA です。

この衝突確率を下げるために、bit 数を増やす、複数の 32bit ハッシュを計算してすべて一致する、という方法があります。本問題では単純な 998244353 との余りで AC しましたので、ここには立ち入らないことにします。 [^1]

[^1]: 衝突について詳しくは ABC339 [F \- Product Equality](https://atcoder.jp/contests/abc339/tasks/abc339_f) をどうぞ。私はしっかり刺されました。


# 実装例

## Z-algorithm
https://github.com/hossy3/atcoder-solutions/blob/main/atcoder/typical90/src/bin/047_z_algorithm.rs

## Rolling-hash
https://github.com/hossy3/atcoder-solutions/blob/main/atcoder/typical90/src/bin/047_rolling_hash.rs