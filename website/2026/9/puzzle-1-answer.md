# 谜题 1 解答

## 题面

你知道吗？Rust 也有空值合并运算符

```rust
fn foo(xs: &mut Vec<Option<i32>>) -> Option<i32> {
    Some(xs.pop() ?? -1)
}
```

## 解答

空值合并运算符（Nullish Coalescing Operator）是一个逻辑运算符，写作两个问号 `??`。

它的作用是，当 `??` 左侧表达式的结果是 `null` 或者是 `undefined` 时，返回右侧的值（一般用来设定默认值），否则返回左侧的值。

它有点类似于逻辑或 `||`，只不过逻辑或将 `0`、`''`、`false` 视为假值，而 `??` 将其视为有效值。

一些例子：

```javascript
const value = null ?? '默认';   // '默认'
const num = 0 ?? 100;          // 0（因为 0 不是 null/undefined）
const str = '' ?? '空字符串';   // ''（同上）
```

显然，Rust 中没有 `null` 也没有 `undefined`，更没有所谓的**空值合并运算符**。那上述代码为什么能通过编译呢？

注意到 [`Vec::pop`](https://doc.rust-lang.org/stable/std/vec/struct.Vec.html#method.pop) 返回一个 `Option<T>`，在题目中，`T` 的类型为 `Option<i32>`，因此 `xs.pop()` 得到了一个 `Option<Option<i32>>`。

而 Rust 的问号操作符 `?` 可以进行错误传播，当你在 `Result` 或者 `Option` 的值后面加上 `?` 时，其相当于以下代码（以 `Option` 为例，`Result` 同理）：

```rust
match v {
    Some(inner) => inner,
    None => return None,
}
```

> 你也可以通过实现 `std::ops::Try` trait 来为你的自定义类型启用 `?` 运算符，不过这个 trait 目前（截止 2026 年 9 月）还未稳定，仍处于 Nightly 阶段。

因此，题目中的 `??`，本质是两个问号操作符，将 `Option<Option<i32>>` 变成一个 `i32`（或者是直接返回 None，这取决于 `xs.pop()` 实际的值）。

而 `-1` 也不是所谓的「默认值」。其中的 `-` 不是作用于 `1` 的一元的负号，而是二元的减法，即通过 `??` 得到的 `i32` 的值减去 `1`。

最后，为了满足返回 `Option<i32>` 的要求，我们在外侧加了一层 `Some`。

所以，本谜题的排版也是十分重要的一环，如若使用 `cargo fmt` 进行格式化，`Some(xs.pop() ?? -1)` 将会被格式化成 `Some(xs.pop()?? - 1)`，这将减少许多的误导性。

---

编译环境：`rustc 1.98.1 (48a229cea 2026-09-01) (Arch Linux rust 1:1.98.1-1)`。

才疏学浅，如有错误还望指正。
