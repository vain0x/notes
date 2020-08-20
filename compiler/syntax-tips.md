# 構文に関する細かいこと

## シンボルの定義に依存する構文の例

識別子がどのようなシンボルとして定義されているかによって構文解析の結果が変わってしまうことがある。

```c
// C

// 型 t* の変数 x の宣言? (auto t* x と書けば明確)
// 変数 t, x の積?
t*x;
```

```fsharp
// F#

// B に等しい型エイリアス A の宣言?
// B をケースに持つ判別共用体型 A の宣言? (type A = | B と書けば明確)
type A = B
```

## Rust の match 式とレコード式の衝突

Rust の match 式とレコード式は微妙に衝突していて、match 式の条件部分にレコード式を直接書くことはできない。

```rust
    struct K<T> { value: T }

    match K { value: () } { _ => {} }
    //      ^ これを match の本体の開始と区別できない
```

同様に if や while の条件式や for のイテレータにも同様の制限がある。

```rust
    if K { value: true }.value {}
    //   ^ 衝突
    
    while K { value: true }.value {}
    //      ^
    
    for _ in K { value: vec![] }.value {}
    //         ^
```

## フィールドと引数の構文の類似

レコードのフィールドと名前付き引数は構文的に似ている。差異:

- フィールドの宣言は可視性を持つが、仮引数は可視性を持たない
    - TypeScript などの一部の言語ではコンストラクタのパラメータに可視性をつけることができて、そのようなパラメータは自動的にフィールドに昇格する機能がある ([TypeScript\: Handbook - Classes](https://www.typescriptlang.org/docs/handbook/classes.html#parameter-properties))
- 仮引数には名前だけでなく任意のパターンを書ける
- 実引数には名前だけでなく任意の式を書ける

```rust
// Rust

struct Point {
    pub x: f64,   // フィールドの宣言
    pub y: f64,
}

fn new_point(
    x: f64,       // パラメータの宣言
    y: f64,
) -> f64 {
    Point {
        x: x,     // フィールドの指定 (単に x, とも書ける (オブジェクトショートハンド))
        y: y,
    }
}
```

```csharp
    CreatePoint(x: x, y: y);   // 名前付き引数の指定
```

BNF:

```
param
    = ident ':' ty

fn_decl
    = 'fn' ident
        '(' (param (',' param)* ','?)? ')' block

record_decl
    = 'struct' ident
        '{' (field_decl (',' field_decl)* ','?)? '}'

field_decl
    = ident ':' ty

arg
    = (ident ':')? expr

call_expr
    = callee '(' (arg (',' arg)* ',')? ')'

record_expr
    = name
        '{' (record_expr_field (',' record_expr_field) ','?)* '}'

record_expr_field
    = ident (':' expr)?
```
