# 名前解決 (name resolution; nameres)

ソースコードに出現する名前 (識別子など) が、実際にどの変数や関数を指しているかを判明させる処理。意味解析の一部。

## 用語集

(言語・コミュニティによって用語にブレがある。)

- 識別子 (identifier; ident, id):
    名前を表すトークン。通常はアルファベット、数字、`_` からなる。キーワードは除く
- シンボル (symbol):
    名前によって指されるもの。変数や関数など
- シンボルテーブル (symbol table):
    名前とシンボルの対応表
- 修飾子 (qualifier):
    名前の参照元を明示する記号。`std::io` の `std::` の部分
- パス (path):
    `std::io::stdin` のように修飾子を含む名前 (Rust コンパイラでの用語)
- 完全修飾名 (fully-qualified name; FQN):
    名前空間のルートからシンボルへのパス。シンボルを完全に特定する。絶対パスのようなもの。
- シンボルの出現 (occurrence)
    - ソースコードに書かれた識別子や名前で、そのシンボルに解決されたもの
    - 例えば `fn f() { f() }` において `fn f` や `f()` などの `f` の部分が関数 f の出現
- 定義箇所 (definition site; def site):
    - シンボルの出現箇所で、そのシンボルの定義になっている部分 (何をもって定義になっているとするかは場合による。例えば Rust では `fn f() { ... }` は関数 f の定義箇所と考えられる)
- 使用箇所 (use site):
    - シンボルの出現箇所、そのシンボルを使っている部分
    - 定義箇所であると同時に使用箇所であることもありうる。例えば、いわゆるオブジェクトショートハンド (`let Point { x, y } = point;` の `x` など)

TODO:

- 名前空間 (namespace)
- 環境 (environment; env)
- スコープ (scope)

## hoisting (巻き上げ)

宣言をブロックの先頭で環境に導入すること。

```rust
{
    // twice はここで、このブロックの環境に導入される。
    // a は導入されない。

    let a = twice(3); // a はここで導入される。

    fn twice(x: i32) -> f64 {
        x * x
    }
}
```

- hoisting されるかは宣言の種類による。
- 単一のブロックに hoisting される同じ名前のシンボルが複数あるときどうするかは言語による。

## shadowing (隠蔽)

環境にシンボルを導入するとき、同名のシンボルをその環境から追い出すこと。

```rust
// 単一のブロック内での shadowing
{
    let a = 1;
    println!("{}", a);  //=> 1

    let a = "2";
    println!("{}", a);  //=> 2
}
```

```rust
// 入れ子のブロックでの shadowing
{
    let a = 1;
    
    {
        let a = "2";
        println!("{}", a);  //=> 2
    }
    
    println!("{}", a);  //=> 1
}
```

- hoisiting された宣言の shadowing をどうするかは言語による。Rust は hositing された宣言も shadowing できる。

```rust
{
    // fn f はここでスコープに導入される
    
    f(); // この f は関数

    let f = 1; // fn f を shadowing する。

    fn f() {}

    let g = f; // この f は整数
}
```

## 構文木を走査する名前解決処理

構文木上を DFS 順序で走査して、構文木に含まれるすべての名前を解決する手続き。

状態:

- 環境のスタック (`List[Map[String, Symbol]]`)
    - 値と型の環境を分けて持つ方がよいかもしれない。

処理:

- ルートやブロックに入ったとき:
    - 環境のスタックに空の `Map` を push する。
    - ブロックに直接含まれる、hositing の対象となる宣言を列挙して、それが定義するシンボルを環境に導入する。
- 名前の定義箇所を見たとき:
    - hoisting の対象でなければ、環境に導入する。
- 名前の使用箇所を見たとき:
    - 非修飾名なら、環境のスタックの末尾から順に、この名前がマップに含まれるか調べて、含まれていたらそれが指すシンボルに解決する。
    - いずれにも含まれていなかったら未解決 (unresolved) とする。
- ブロックから出るとき:
    - 環境のスタックを pop する。
  
## 出現から逆順に辿る名前解決処理

- 名前が含まれるブロックの前方にある宣言を逆順に見て、もっとも近くで導入される同じ名前のシンボルに解決する。
- 名前が含まれるブロックの後方にある、hoisiting の対象となる宣言によって導入される同じ名前のシンボルに解決する。
- 解決できなければ、上のブロックを見て同じ処理を繰り返す。
- 最後まで解決されなければ未解決 (unresolved) とする。

## 名前解決が型やトレイト、動的な状態に依存するケース

オブジェクト指向言語によくあるドット記法 `x.f()` は、`f` が何を指すかが `x` の型によるので、型検査・型推論の後でないと処理できない。

`eval` などにより動的にシンボルを環境に導入できる言語では、名前は実行時まで解決できない。

- [Dependent names - cppreference.com](https://en.cppreference.com/w/cpp/language/dependent_name) (C++ ではテンプレート引数が何に解決されるかによって解決結果が異なるパスを依存名と呼ぶ)

## See also

- [Name lookup - cppreference.com](https://en.cppreference.com/w/cpp/language/lookup) (C++ の名前ルックアップ)
- [Name resolution - Guide to Rustc Development](https://rustc-dev-guide.rust-lang.org/name-resolution.html)
- [Three Architectures for a Responsive IDE](https://rust-analyzer.github.io/blog/2020/07/20/three-architectures-for-responsive-ide.html)
