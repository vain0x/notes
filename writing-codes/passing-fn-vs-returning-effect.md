# 関数を渡す vs. エフェクトを返す

どちらも計算の一部を抽象化できる。

TODO: 意味のある例

## 関数を渡す

```ts
const foo = (f: (i: number) => number): number => {
    let x = 0
    for (const i of [1, 2, 3]) {
        x += f(i)
    }
    return x
}

foo(i => i) //=> 6
foo(i => i * i) //=> 14
```

## エフェクトを返す

```ts
type FooResult<A> =
    {
        kind: "FOO_YIELD"
        value: number
        cont: (value: number) => FooResult<A>
    } | {
        kind: "FOO_RETURN"
        value: A
    }

const foo = (): FooResult<number> => {
    let x = 0
    let i = 0

    const aux = (a: number) => {
        x += a
        if (i <= 3) {
            i++
            return { "FOO_YIELD", value: i, cont: aux }
        }
        return { kind: "FOO_RETURN", value: x }
    }
    return aux(1)
}

const performFooResult = (result: FooResult<number>) => {
    while (true) {
        switch (result.kind) {
            case "FOO_YIELD":
                result = result.cont(result.value)
                continue

            case "FOO_RETURN":
                return result.value

            default:
                throw "never"
        }
    }
}

performFooResult(foo()) //=> 6
```
