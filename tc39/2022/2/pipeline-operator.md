TC39 Pipeline operator

- https://github.com/tc39/proposal-pipeline-operator

- Stage 2

- https://tc39.es/proposal-pipeline-operator/



### Deep nesting is hard to read

```javascript
one(two(three(four))) //<- low readability !!
```

```rust
let res = four
let res = three(res);
let res = two(res);
let res = one(res);
```

ていうてもあるよね

```javascript=
let res = four
res = three(res);
...


const res = (() => {
    let res = four;
    res = three(res);
    ...
    return res
})()
```

```python=
# deep learning
model = model.vec(~~)
model = model.vec(~~2)
...
```

```rust
let res = four
let res = three(res);
let res = two(res);
let res = one(res);
```

Rustのこの書き方は手続き的で, pipeline operator は宣言的だよね. 宣言的というのは無条件で手続機な表現に勝るのかみたいな話もありそう

- Domain
- Coding Style

Note: react-domはある種JSXをUIに描画する広義の **純粋** な機能を扱い, React は　副作用を扱う. これを二つ合わせて **React** なのではないか仮説. React Componentのmethodや, hooksは React の扱う副作用をユーザーがコールバックという形でlifecycleの上で定義させるインターフェースなのではないか.


### Method chaining is limited

method chainingは左から右に関数を連鎖させることができるのでreadabilityは良いし各引数を関数でグループ化させて所属を明確化できる

けれど...
chainingするには事前にchainingされる構造体にメソッドが生えてないといけないしyieldとかの機能を格納することができなくて適用性(applicability)に制限がある

```javascript=
value.one().two().three() // how to insert arithmetic ?
```

都度変数に格納するのは, `let` の曖昧さを受け入れるか, 一瞬しか使われない変数のためにわざわざ変数名を(しかも具体的になるため長い命名になってしまいがち)考えるのはしんどい

Pipeline Operator、文じゃなくて式で表されてるってのも汎用性高くて良き

## Hack Pipe operator vs F# pipes

基本似ている

### Hack Pipe Operator: 
- pipes expression

### F# Pipes
- pipes unary function
    - 単一引数を取る関数
        - unary function, binary function, ternary function...
    - これfunctionだと算術演算子とか渡せなくない？

 元になった#のproposal
    https://github.com/tc39/proposal-smart-pipelines


# 2022/2/22 Pipeline operator

この会の続き
https://hackmd.io/Y9KpaTJeR0qwMDKVCZnc7w

https://www.kabuku.co.jp/developers/pipe-operator-debate

F# style か Hack styleか

F# pipe + partial application = Hack pipe

https://benlesh.com/posts/tc39-pipeline-proposal-hack-vs-f-sharp/


現在のJavaScriptで実装できるpipe
```javascript=
function pipe(initialArg, ...fns) {
  return fns.reduce((prevValue, fn) => fn(prevValue), initialArg);
}

const result = pipe(
  2,
  (x) => x ** 2,
  (x) => x - 1,
);
```

## RxJSでの例

RxJS だと, 昔 (v5.5以前) はモナド的にmethod chainを使っていた.
これは問題があって
- 先週やったようにyeildや算術演算式を格納できない
- tree shakingが効かない

というのがある. methodだと, (swcとかだと多分いけるけど)メソッドが使われているかの検査ができず全てのメソッドがバンドルに残ってしまう. それを避けるために piped functionを使ってパフォーマンスの最適化を図った.

## Hack pipe (current)

`^` に返り値が入るようにする.

```javascript=
// An example of using existing non-unary function APIs:
const randomNumberBetween20And50 = Math.random() * 100
  |> Math.pow(^, 2)
  |> Math.min(^, 50)
  |> Math.max(20, ^)
```

F#との違いは
- ^
- より愚直

Hack pipeの方がparserの対応が大変そう (^がある分)
- accorn の対応を待ってさらにlinter, bundler, transplierのその依存のアップデートを待たないといけない
- webpack4 の optional chaining や nullish coallesing がaccornが古くて使えない問題とかもあるし...

```javascript=
// F# await
const hoge = hoge()
  |> (hoge) => await bar(hoge)
              ~~~~~~~ oops!
```

optional hack pipe も考えられ中(?)

