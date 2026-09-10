# 谜题 2 解答

## 题面

你知道吗？Rust 支持命名参数，声明参数名后即可乱序传参

```rust
let connect = |timeout, retries| {};

let (timeout, retries);
connect(retries = 3, timeout = 30);

assert_eq!(timeout, 30);
assert_eq!(retries, 3);
```

## 解答

命名参数（named arguments / named parameters）是指在调用函数时通过**参数名**来指定参数，而不只是通过位置来确定参数。通常，命名参数会与默认参数一起使用。

在 Python，命名参数叫作关键字参数（Keyword Argument）：

```python
def create_user(name, age, active=True):
    ...

create_user(name="Alice", age=30, active=False)
create_user(age=30, name="Alice")   # 可以打乱顺序
create_user("Alice", age=30)        # 也可以与位置参数混用
```

目前，Rust 不支持命名参数，你可以看到人们为此进行了相当多的讨论（例如：[State of named function parameters in Rust](https://users.rust-lang.org/t/state-of-named-function-parameters-in-rust/139512/)、[Pre-RFC: Named arguments](https://internals.rust-lang.org/t/pre-rfc-named-arguments/16413/)）。

那么，上述代码为什么能通过编译呢？

注意到在 Rust 中，`=` 是赋值运算符，源码中的 `retries = 3` 和 `timeout = 30` 是[赋值表达式（Assignment expressions）](https://doc.rust-lang.org/stable/reference/expressions/operator-expr.html#r-expr.assign)，且一个赋值表达式始终产生 `()`（详见 [reference](https://doc.rust-lang.org/stable/reference/expressions/operator-expr.html#r-expr.assign.result)）。

所以调用 `connect` 时实际传入的参数是两个 `()`。

而 `connect` 是一个[闭包（Closure）](https://doc.rust-lang.org/stable/reference/types/closure.html#r-type.closure)，它的参数（或称作「模式」）允许忽略类型注解。省略类型注解时，参数的类型将会从上下文中推断出来（详见 [reference](https://doc.rust-lang.org/stable/reference/expressions/closure-expr.html#r-expr.closure.parameter-restriction)）。

因此，题目中的 `connect` 实际上接受两个 `()` 而不是两个 `i32`。这也是本谜题中误导人的一部分——如果`connect` 不为空，而是真正地使用了 `timeout` 和 `retries`，这将无法编译。

借助 `cargo clippy`，我们也可以发现上述题目生成了如下警告：

```text
warning: passing unit values to a function
 --> src/main.rs:5:5
  |
5 |     connect(retries = 3, timeout = 30);
  |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  |
  = help: for further information visit https://rust-lang.github.io/rust-clippy/rust-1.98.0/index.html#unit_arg
  = note: `-W clippy::unit-arg` implied by `-W clippy::all`
  = help: to override `-W clippy::all` add `#[allow(clippy::unit_arg)]`
help: move the expressions in front of the call and replace them with the unit literal `()`
  |
5 ~     let _: () = retries = 3;
6 +     let _: () = timeout = 30;
7 ~     connect((), ());
  |
```

它敏锐地识别到你给函数传递的参数是 `()`，但你传的不是字面量。这一下就将谜题给解决了。

---

编译环境：`rustc 1.98.1 (48a229cea 2026-09-01) (Arch Linux rust 1:1.98.1-1)`。

才疏学浅，如有错误还望指正。
